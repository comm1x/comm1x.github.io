---
layout: post
title: "Ответы-агрегаты в REST-API"
description: "Ответы-агрегаты в REST-API"
tags: [rest, aggregate, roots, api, ответы, агрегаты]
---

## TLDR

Чтобы передать вложенные сущности в REST-API - не вкладывайте сущности друг в друга. Вместо этого
положите все сущности в агрегат и передайте его.

| Агрегат (он же агрегирующий контейнер, объект-агрегат) - объект, в который вложены все другие плоские сущности. 

Ответы с агрегатами легче расширять и поддерживать.

## Проблема вложенных сущностей

> В REST один ресурс должен отвечать за одну сущность

Это правило не работает. Если мне надо получить 10к сущностей, то
я должен выполнить 10к запросов. Прекрасно, живите сами с таким рестом.

На практике, мы вкладываем сущности в сущности, и вместе их передаем. И это порождает 
следующие проблемы. 

### Проблема 1: разный набор вложенных полей

**У одной и той же сущности может быть разных набор вложенных сущностей**, 
в зависимости от вызываемого ресурса.

| Например у `user` в одном запросе могут быть поля `transactions` и `purchases`, а в другом `uploaded_videos`, `pets` и `friends`.

И в итоге, когда вы работаете с `user` на клиенте, **вам нужно знать откуда пришел объект** `user`, 
чтобы знать какой набор вложенных сущностей у него сейчас доступен.

Это повышает недетерминированость кода на клиенте, код становится неоднозначным. Мы уже не можем
утверждать упадет ли код или нет, пока не увидим откуда приходят данные.

```js
function getFirstFavoriteCarOf(user) {
    return user.favorite_cars.first()  // Корректный ли это код?
}
```


### Проблема 2: редукция связей many-many до one-many

Вкладывая сущности друг друга, у вас всегда будет родитель и ребенок, т.е. вам доступно только
отношение one-many. 

**Невозможно выразить many-many через вложенные сущности.** 

Решения два:
1. редуцировать many-many до one-many
2. выносить родителей, детей и отношения на один уровень

> Еще раз. Редукция это когда у вас есть `users` и `books`, и между ними many-many (люди которые прочитали
книги; один юзер может прочитать несколько книг; одну книгу могут прочитать много людей).
Если вы передадите список `users` в которых будет поле `read_books`, то у каждого `user` будет свой
набор книг. Т.е. мы many-many разделили на one-many и разложили по юзерам.

Что не так с редукцией?

Редукция искажает предметную область, порождает соотношения, которых нет в вашей предметной области. 
Может путать разработчиков. В примере выше - книги вложены в юзеров, это можно прочитать как
"книги принадлежат юзерам". А в другом ответе могут юзера принадлежать книгам (быть вложенными в них).
И получается что транзитивно через книги **юзера принадлежат... юзерам**? Что?

На деле никто никому не принадлежит, потому что принадлежность фиктивная, она получилась редукцией.

**Редукция - источник больших проблем и непонимания.** Избегайте ее.

## Решение - Агрегаты

Вместо вкладывания сущностей в сущности, их нужно все положить в один контейнер.
Соотношения many-many также кладутся отдельно от сущностей

```js
{
    "shops": [...],
    
    // каждый продукт хранит id родительского магазина, по которому можно сгруппировать
    "products": [{ shop_id: 1, ... }, ...], 
    
    "users": [...],
    "user_to_product": [...], // many-many
    "reviews": [{ product_id: 1, user_id: 2, ...}, ...] 
    "purchases": [...],
    "cities": {115: {...}, 216: {...}
}
```


Что нам это дает:

- **Клиент сам формирует ответ для себя**   
При использовании агрегатов, бэкенд снимает с себя всю ответственность за построение предварительной 
структуры, которая была бы удобна клиенту.  
Подготовка удобной для клиента структуры - это на самом деле **ответственность клиента**, а не бэкенда.

- Использование агрегатов снимает с бэкенда вопрос **"в какой структуре передать данные?"**  
Теперь бэкенд может думать только над тем **что передать**.

- Добавление новых сущностей - это всего лишь **добавление одного поля** в агрегат

Единственный пункт, который все же можно оставить в ответственности бэка - это передача [словарей](https://en.wikipedia.org/wiki/Associative_array) вместо списков.
Иногда есть смысл передавать не список, а уже построенный словарь с `id` в качестве ключей (пример `cities` выше).      
В поле `reviews` можно было бы построить словарь `product_id => List<Review>` 

#### Эволюция структуры ответа

Рост уровня вложенности сущностей, влечет за собой рост сложности ответа.
При уровне вложенности = 5, уже сложно говорить о переиспользуемости и универсальности ответа. В случае с агрегатами,
мы можем добавлять сколько угодно разных сущностей в агрегат, и структура ответа будет оставаться примитивной.

Эволюция структуры в агрегате - это просто добавление или удаление поля.
В "классических ответах" в случае когда сущности связаны по схеме: `A -> B -> C` проблематично будет убрать B из схемы,
не убрав С.

#### Отсутствие дублирования сущностей

Это не очень важно, но все же плюс. В агрегатах исключено дублирование сущностей, поэтому ответ
будет существенно меньше чем при вложенных сущностях.

### Недостатки подхода

Появляется необходимость формирования структуры на клиенте.
