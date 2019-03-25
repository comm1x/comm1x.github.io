---
layout: post
title: "Как записать загрузочный ISO на USB-Flash в Linux"
description: "Как записать загрузочный ISO на USB-Flash в Linux"
categories: 
tags: [iso, dd, linux, windows, win7, winusb, flash, загрузочный, образ, записать]
---

### Теория

Записать iso на флешку не сложно - `dd if=.. of=..` и готово.
Но вот будет ли она загрузочной или нет, зависит от самого ISO-образа, а именно
является ли он гибридным или нет. 

Гибридный образ - это такой образ, который можно записать и на CD/DVD и на USB.
Если не гибридный образ попытаться записать с помощью dd на USB-флешку, то он просто не запустится (не будет загрузочным).
Если образ гибридный, он легко записывается с помощью `dd`.

Большинство ISO-образов linux-дистрибутивов являются гибридными, чего не скажешь про Windows.
Запись не гибридных образов выглядит следующим образом:

- форматируем флешку и создаем на ней один пустой загрузочный раздел
- копируем файлы с ISO-образа на флешку

После этого флешка становится загрузочной. Кстати, после этого с помощью `dd`
можно снять образ флешки (`dd if=/dev/sdc1 of=~/windows.img`), и у нас уже будет загрузочный образ для флешки, который
после можно будет записывать с помощью dd. Будет ли он являться гибридным
мне не известно, т.к. не ясно будет ли этот образ загрузочным для CD/DVD.

### Запись ISO Linux

Как уже было сказано, большинство ISO-образов linux являются гибридными, поэтому:

{% highlight bash %}
dd if=path/to/image.iso of=/dev/sdc
{% endhighlight %}

Где `/dev/sdc` - ваша флешка

### Запись ISO Windows, способ №1 ручной

С помощью GParted на флешке создаем новую таблицу разделов MSDOS.
Далее создаем один раздел, форматируем в fat32 и устанавливаем флаг boot.
 
После этого нам нужно скопировать файлы с iso на usb:

{% highlight bash %}
mkdir /mnt/iso
mkdir /mnt/usb
mount -o loop ~/path/to/image.iso /mnt/iso
mount /dev/sdc1 /mnt/usb
cp -r /mnt/iso/* /mnt/usb/

sync
umount /mnt/usb
umount /mnt/iso
{% endhighlight %}

### Запись ISO Windows, способ №2 автоматический 

Весь процесс описанный в способе №1 за нас может сделать WinUSB.

Собираем утилиту (после make install утилита добавляется в меню приложений):

{% highlight bash %}
git clone https://github.com/slacka/WinUSB
./configure
make 
make install
{% endhighlight %}

Теперь можно использовать.
{% highlight bash %}
winusb --install win7_amd64.iso /dev/sdd1
{% endhighlight %}

Также у утилиты есть простой и понятный GUI. Так что загляните в меню.

### Дополнительно

Узнать имя флешки `lsblk`, `fdisk -l`

Еще один способ для капризных систем (arm, нетбуки т.д.): [http://serverfault.com/questions/6714/how-to-make-windows-7-usb-flash-install-media-from-linux](http://serverfault.com/questions/6714/how-to-make-windows-7-usb-flash-install-media-from-linux)

[http://wiki.rosalab.com/ru/index.php/Тестирование ISO образа на гибридность](Тестирование ISO образа на гибридность)