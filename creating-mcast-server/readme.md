В этом небольшом мануале опишу, как к себе в лабораторный стенд eve-ng добавить источник мультикаста для выполнения лабораторных работ. 
Итак, первым делом, необходимо образ себе скачать. Идем по ссылке и качаем: https://yadi.sk/d/GNxWpaG-35pbqA
После скачивания копируем архив с образом на нашу виртуалку с eve-ng. Я использую утилиту scp, поскольку у меня на Windows 10 установлена подсистема Linux

<pre><code>
bonik@Bonikacer:/mnt/d/Downloads$ scp linux-ubuntu-mcast-source.zip bonik@192.168.1.206:/home/bonik/
bonik@192.168.1.206's password:
linux-ubuntu-mcast-source.zip                                                                 100%  606MB  57.8MB/s   00:10
</code></pre>

После этого топаем в домашнюю директорию eve-ng и распаковываем архив
<pre><code>
bonik@eve-ng:~$ unzip linux-ubuntu-mcast-source.zip 
Archive:  linux-ubuntu-mcast-source.zip
   creating: linux-ubuntu-mcast-source/
  inflating: linux-ubuntu-mcast-source/virtioa.qcow2
</code></pre>

"Переодеваемся" в root, и переносим распакованный архив в каталог с qemu образами: 
<pre><code>
bonik@eve-ng:~$ su
Password: 
root@eve-ng:/home/bonik# mv linux-ubuntu-mcast-source /opt/unetlab/addons/qemu/
root@eve-ng:/home/bonik# ls /opt/unetlab/addons/qemu/ -la | grep mcast
drwxrwxr-x  2 bonik bonik 4096 Jan 14  2021 linux-ubuntu-mcast-source
</code></pre>

И завершаем процедуру выполнением команды (от рута!!!)
<pre><code>
root@eve-ng:/home/bonik# /opt/unetlab/wrappers/unl_wrapper -a fixpermissions
</code></pre>

Открываем EVE-NG и пробуем добавить Linux-хост: 

![Добавление хоста](https://github.com/bonishvarik/otus-net-arch/blob/main/creating-mcast-server/1.png)
Чуток расшифровки: 
1 - Собственно, сам шаблон
2 - Выбираем конкретный образ, в случае, если их несколько
3 - Имя сервера, которое будет отображаться на схеме
4 - Количество сетевых интерфейсов (если ничего, кроме мультикаста не нужно, оставляем 1)
5 - Тип подключения к серверу. Тут можно выбрать как VNC, так и telnet. Работают оба варианта.

После этого запускаем виртуалку и заходим на нее. Для авторизации используем логин/пароль root/root
Для начало посмотрим конфигурацию сетевых интерфейсов: 
<details>
  <summary>root@ubuntu:~# cat /etc/network/interfaces</summary>
<pre><code>
root@ubuntu:~# cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The network interface for multicasting
auto ens3
iface ens3 inet manual

auto vlan111
iface vlan111 inet static
        address 172.17.111.1
        netmask 255.255.255.252
        vlan_raw_device ens3

auto vlan112 
iface vlan112 inet static
        address 172.17.112.1
        netmask 255.255.255.252
        vlan_raw_device ens3

auto vlan113
iface vlan113 inet static
        address 172.17.113.1
        netmask 255.255.255.252
        vlan_raw_device ens3

auto vlan114
iface vlan114 inet static
        address 172.17.114.1
        netmask 255.255.255.252
        vlan_raw_device ens3
</code></pre>
</details>
Как видно выше, на сетевом интерфейсе ens3 созданы VLAN-интерфейсы с адресами 172.17.X.1/30, где X - номер VLAN'а. 
Убедимся, что все сетевые интерфейсы успешно поднялись:
<details>
  <summary>root@ubuntu:~# ip addr show</summary>
<pre><code>
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:00:00:01:00:00 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::5200:ff:fe01:0/64 scope link 
       valid_lft forever preferred_lft forever
3: vlan111@ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 50:00:00:01:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.111.1/30 brd 172.17.111.3 scope global vlan111
       valid_lft forever preferred_lft forever
    inet6 fe80::5200:ff:fe01:0/64 scope link 
       valid_lft forever preferred_lft forever
4: vlan112@ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 50:00:00:01:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.112.1/30 brd 172.17.112.3 scope global vlan112
       valid_lft forever preferred_lft forever
    inet6 fe80::5200:ff:fe01:0/64 scope link 
       valid_lft forever preferred_lft forever
5: vlan113@ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 50:00:00:01:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.113.1/30 brd 172.17.113.3 scope global vlan113
       valid_lft forever preferred_lft forever
    inet6 fe80::5200:ff:fe01:0/64 scope link 
       valid_lft forever preferred_lft forever
6: vlan114@ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 50:00:00:01:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.114.1/30 brd 172.17.114.3 scope global vlan114
       valid_lft forever preferred_lft forever
    inet6 fe80::5200:ff:fe01:0/64 scope link 
       valid_lft forever preferred_lft forever
</code></pre>
</details>
Отлично. Далее обратим внимание на содержимое директории /root, в которой лежит 2 интересующих нас файла: video.ts и start_mcast.sh.
video.ts - собственно, сам видеофайл, который мы будем вещать по нашей сети
start_mcast.sh - скрипт, который запускает вещание. 
Глянем на сам скрипт:
<details>
  <summary>root@ubuntu:~# cat start_mcast.sh</summary>
<pre><code>
#!/bin/bash
tsplay video.ts 239.0.0.111:1234 -loop -i 172.17.111.1 &
tsplay video.ts 239.0.0.112:1234 -loop -i 172.17.112.1 &
tsplay video.ts 239.0.0.113:1234 -loop -i 172.17.113.1 &
tsplay video.ts 239.0.0.114:1234 -loop -i 172.17.114.1 &
</code></pre>
</details>

Разберем вторую строку скрипта: 
<pre><code>
tsplay video.ts 239.0.0.111:1234 -loop -i 172.17.111.1 &
</code></pre>
tsplay - собственно, имя утилиты, которая будет вещать мультикастовый поток
video.ts - имя видеофайла, который будет вещаться
239.0.0.111:1234 - IP-адрес и UDP-порт мультикастовой группы
-loop - опция, которая при завершении файла, начинает воспроизводить его заново (для циклического вещания файла)
-i 172.17.111.1 - IP-адрес интерфейса, который будет выступать источником потока

В случае необходимости, всегда можно изменить сам скрипт и настройки сети, чтобы добавить или удалить сетевые интерфейсы или мультикастовые группы. 
Перед стартом скрипта, давайте подключим наш сервер к оборудованию Cisco и проверим, что мультикаст точно льется: 

