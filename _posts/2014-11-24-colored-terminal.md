---
layout: post
title: "Информативные приглашения в bash"
description: "Информативные приглашения в bash"
tags: [bash]
modified: 2016-04-06
---

### Как это выглядит

В данной статье я расскажу как настроить такое приглашение в терминале:

<figure class="center">
	<img src="/images/bash-git-ps1-example.png" alt="">
	<figcaption>Скриншот терминала с PS1 из последнего раздела</figcaption>
</figure>

### Цвета

Шаблон приглашения хранится в переменной PS1 текущего окружения.
Узнать ее текущее значение:
{% highlight bash %}
echo $PS1
{% endhighlight %}

Настраиваем красное приглашение для рута. Добавляем в `/root/.bashrc`

{% highlight bash %}
export PS1="\[\e[31;1m\]\u@\h \[\e[31;0m\e[33m\]\w# \[\e[0m\]"
{% endhighlight %}

Зеленое приглашение для остальных пользователей. Добавляем в `~/.bashrc`

{% highlight bash %}
export PS1="\[\e[32;1m\]\u@\h \[\e[31;0m\e[32m\]\w \[\e[0m\]"
{% endhighlight %}

Перезапустить консоль либо сделать:

{% highlight bash %}
source ~/.bashrc
{% endhighlight %}

Для того, чтобы изменить цвета на свои, нужно менять цифры: 0, 1, 31, 33

Более подробно о настройке цветов здесь: [http://habrahabr.ru/post/94647](http://habrahabr.ru/post/94647)

### Git в bash

Git умеет встраиваться в приглашение почти из коробки. Нужно вызвать функцию `__git_ps1`, чтобы получить
информацию о текущем состоянии в скобках. Выглядит это примерно так (только в цветах):
 
{% highlight bash %}
cmx ~/projects/sample (master)
{% endhighlight %}

Оборачиваем в цвета и получаем строку:
{% highlight bash %}
__git_ps1 '\e[36;1m(%s)\e[31;0m\e[32m '
{% endhighlight %}

Теперь экранируем ее и вставляем в PS1 и получаем:
{% highlight bash %}
export PS1="\[\e[32;1m\]\u@\h \[\e[31;0m\e[32m\]\w \$(__git_ps1 '\[\e[36;1m\](%s)\[\e[31;0m\e[32m\] ')\[\e[0m\]"
{% endhighlight %}

Также у `__git_ps1` есть дополнительные опции, включение которых позволяет выводить дополнительную информацию.
Я оставил наиболее полезные из них: 

* "*" dirty state
* "%" untracked files
* "+" добавленные, но не закомиченные файлы
 
Для включения этих опций нужно присвоить переменным значения до вычисления PS1.  

{% highlight bash %}
export GIT_PS1_SHOWDIRTYSTATE=1
export GIT_PS1_SHOWUNTRACKEDFILES=1
export GIT_PS1_DESCRIBE_STYLE=default
{% endhighlight %}

Полный список опций [https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh](https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh)

### Результат

Мой финальный код в `~/.bashrc` для `PS1` выглядит так:
{% highlight bash %}
export GIT_PS1_SHOWDIRTYSTATE=1
export GIT_PS1_SHOWUNTRACKEDFILES=1
export GIT_PS1_DESCRIBE_STYLE=default
export PS1="\[\e[32;1m\]\u \[\e[31;0m\e[32m\]\w \$(__git_ps1 '\[\e[36;1m\](%s)\[\e[31;0m\e[32m\] ')\[\e[0m\]"
{% endhighlight %}

Я также убрал `@\h` при работе на локальной машине