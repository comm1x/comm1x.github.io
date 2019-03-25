---
layout: post
title: Как подключить Yii к phpDaemon
description: "Как подключить Yii к phpDaemon"
tags: [yii, php, phpdaemon]
---
### Для чего

Для того, чтобы пользоваться Yii в режиме службы (демона). Хорошо подходит для развертки WebSocket-сервера.
<!-- more -->

### Решение

Решение на самом деле очень простое. Нужно всего лишь подключить файл фреймворка `yii.php` и построить приложение.

{% highlight php startinline %}
class WebSocketServer extends AppInstance
{

    public function onReady()
    {

        define('YII_ENABLE_ERROR_HANDLER', false);

        require_once('../vendor/yiisoft/yii/framework/yii.php');
        $config = '../protected/config/main.php';
        Yii::createWebApplication($config);
        // ...
    }

    // ...
}
{% endhighlight %}

Теперь можно спокойно вызывать `Yii::app()`

### Запуск контроллера

{% highlight php startinline=true %}
Yii::app()->runController('site/index');
{% endhighlight %}

### Правим PHPDoc

`Yii::app()` возвращает `CApplication`, а метод `runController()` принадлежит `CWebApplication`, поэтому среда может
ругаться. В этом случае переопределите PHPDoc класса Yii, либо через промежуточную
переменную (мы все равно это вызываем 1 раз, интерпретатор это дело соптимизирует):

{% highlight php startinline=true %}
/** @var \CWebApplication $app */
$app = Yii::app();
$app->runController('site/index');
{% endhighlight %}

### Подводные камни

Без `YII_ENABLE_ERROR_HANDLER = false`, Yii зарегистрирует свой обработчик ошибок и при первом же notice|warning|error отправит сервер спать (спасибо Василию Зорину, [он рассказал это здесь](https://groups.google.com/forum/#!topic/phpdaemon/56EUd76IvRo))
`protected/runtime` папка должна быть обязательно и с правами 777 (у меня без папки зависало все приложение, причем убивать его после зависа только через `kill -9`)
папка protected у меня лежит в корне проекта
при `protected/config/main.php` сервер работает, а вот при `protected/config.php` сервер уже падает
папки в protected: config, controllers, models, runtime

Перезапуск зависшего сервиса
Ищем PID-ы запущенный процессов

{% highlight bash %}
# ps ax | grep phpd | grep process
31862 ?        SNs    0:00 phpd: master process
31864 ?        SNl    0:00 phpd: worker process
{% endhighlight %}

Убиваем с наивысшим приоритетом

{% highlight bash %}
# kill -9 31862 31864
{% endhighlight %}

И перезапускаем

{% highlight bash %}
# phpd restart
{% endhighlight %}

### Дополнительно

[http://www.yiiframework.com/doc/guide/1.1/en/extension.integration#using-yii-in-3rd-party-systems](http://www.yiiframework.com/doc/guide/1.1/en/extension.integration#using-yii-in-3rd-party-systems)