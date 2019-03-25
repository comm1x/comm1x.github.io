---
layout: post
title: "Как запустить PHP-скрипт в отладочном режиме"
description: "Как запустить PHP-скрипт в отладочном режиме"
tags: [php, bash, debug]
---
### Коротко

Все относительно просто, достаточно запустить скрипт с включенным флагом `xdebug.remote_autostart`. Делается это так:

{% highlight bash %}
php -dxdebug.remote_autostart=On script.php
{% endhighlight %}
<!-- more -->
Также можно включить глобальный режим отладки, определив переменную окружения `XDEBUG_CONFIG="idekey=netbeans-xdebug"`

Теперь в текущей bash-сессии все вызовы `php script.php` будут вызываны в отладочном режиме.

### Решение

Теперь осталось все это дело красиво оформить, добавляем в ~/.bashrc

{% highlight bash %}
alias php-debug="php -dxdebug.remote_autostart=On"
alias php-debug-enable="export XDEBUG_CONFIG=\"idekey=netbeans-xdebug\""
alias php-debug-disable="export -n XDEBUG_CONFIG"
{% endhighlight %}

Перезагружаем сессию или вызываем `source ~/.bashrc` и теперь можно использовать
`php-debug` - запуск скрипта в отладочном режиме
`php-debug-enable` и `php-debug-disable` - включение и выключение глобального отладочного режима

Пример использования:

{% highlight bash %}
php script.php 
php-debug script.php 
php-debug-enable
php script1.php
php script2.php
php script3.php
php-debug-disable
{% endhighlight %}


### Дополнительно
1. В алиасе `php-debug-enable` не забудьте убедиться, что в настроена на аналогичный ключ.
2. Если в PHPStorm указать пустой IDEKEY, то среда будет перехватывать все xdebug вызовы, что в целом неплохо, можно не
заморачиваться в поводу значения ключа (собственно, по умолчанию оно и является пустым).