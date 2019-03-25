---
layout: post
title: "Как добавить уникальный индекс в Mysql, удалив дублирующиеся данные"
description: "Как добавить уникальный индекс в Mysql, удалив дублирующиеся данные"
tags: [mysql]
---
### Коротко

Все относительно просто. Сперва разберемся с дубликатами.

Допустим мы хотим удалить все записи из таблицы `users` с одинаковым login.
 
Находим все записи с одинаковым id и удаляем те, у которых id меньше:

{% highlight sql %}
DELETE t1 FROM users t1, users t2 WHERE 
    t1.login = t2.login 
    AND t1.id < t2.id;
{% endhighlight %}
<!-- more -->
Если мы хотим добавить составной уникальный ключ, например first_name + last_name, то для удаления таких дубликатов
просто добавим дополнительное условие:

{% highlight sql %}
DELETE t1 FROM users t1, users t2 WHERE 
    t1.first_name = t2.first_name
    AND t1.last_name = t2.last_name
    AND t1.id < t2.id;
{% endhighlight %}
    
При большем количестве полей составного ключа просто добавляем условия. И далее добавляем ключ:

{% highlight sql %}
-- Для одного столбца
ALTER TABLE users ADD UNIQUE login_unique (login);

-- Или составной
ALTER TABLE users ADD UNIQUE fn_name_unique (first_name, last_name);
{% endhighlight %}

### Дополнительно

Рекомендую всегда добавлять имя ключу, чтобы избежать дублирования индексов, в случае если запрос выполнится 2 раза.