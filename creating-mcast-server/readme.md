# Создание источника мультикаста в EVE-NG

В этом небольшом мануале опишу, как к себе в лабораторный стенд eve-ng добавить источник мультикаста для выполнения лабораторных работ. 
Итак, первым делом, необходимо образ себе скачать. Идем по [ссылке](https://yadi.sk/d/GNxWpaG-35pbqA) и качаем
После скачивания копируем архив с образом на нашу виртуалку с eve-ng. Я использую утилиту scp, поскольку у меня на Windows 10 установлена подсистема Linux

<pre><code>
bonik@blabla:/mnt/d/Downloads$ scp linux-ubuntu-mcast-source.zip bonik@192.168.1.206:/home/bonik/
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
</code></pre>

Было бы неплохо проверить контрольную сумму скачанного образа:
<pre><code>
root@eve-ng:~$ md5sum /opt/unetlab/addons/qemu/linux-ubuntu-mcast-source/virtioa.qcow2 
df6a2a4f73ccbdfbdd853687be60f37d  /opt/unetlab/addons/qemu/linux-ubuntu-mcast-source/virtioa.qcow2
</code></pre>

Если контрольная сумма файла не совпадает с тем, что выше - файл поврежден. Повторите процедуру скачивания.
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
#
# The loopback network interface
auto lo
iface lo inet loopback
#
# The network interface for multicasting
auto ens3
iface ens3 inet manual
#
auto vlan111
iface vlan111 inet static
        address 172.17.111.1
        netmask 255.255.255.252
        vlan_raw_device ens3
#
auto vlan112 
iface vlan112 inet static
        address 172.17.112.1
        netmask 255.255.255.252
        vlan_raw_device ens3
#
auto vlan113
iface vlan113 inet static
        address 172.17.113.1
        netmask 255.255.255.252
        vlan_raw_device ens3
#
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
![Топология сети](https://github.com/bonishvarik/otus-net-arch/blob/main/creating-mcast-server/2.png)

Конфигурация Cisco выглядит следующий образом: 
<pre><code>
ip multicast-routing 
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface Ethernet0/0.111
 encapsulation dot1Q 111
 ip address 172.17.111.2 255.255.255.252
 ip pim passive
!
interface Ethernet0/0.112
 encapsulation dot1Q 112
 ip address 172.17.112.2 255.255.255.252
 ip pim passive
!
interface Ethernet0/0.113
 encapsulation dot1Q 113
 ip address 172.17.113.2 255.255.255.252
 ip pim passive
!
interface Ethernet0/0.114
 encapsulation dot1Q 114
 ip address 172.17.114.2 255.255.255.252
 ip pim passive
!
ip pim rp-address 1.1.1.1
</code></pre>
<details>
<summary>Убедимся, что с Cisco можем пинговать сервер</summary>
<pre><code>
Router#ping 172.17.111.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.17.111.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
Router#ping 172.17.111.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.17.111.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/5 ms
Router#ping 172.17.112.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.17.112.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
Router#ping 172.17.113.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.17.113.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
Router#ping 172.17.114.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.17.114.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
</code></pre>
</details>

Теперь на сервере запускаем наш скрипт
<pre><code>
root@ubuntu:~# ./start_mcast.sh 
</code></pre>

И идем на циску. Для начала посмотрим наличие трафика на интерфейсе e0/0:
<pre><code>
Router#show interfaces e0/0 human-readable  | i rate
  Queueing strategy: fifo
  5 minute input rate 3.99 mega-bits/sec , 346 pps 
  5 minute output rate 0 bits/sec, 0 packets/sec
</code></pre>
Как видим, у нас есть почти 4 мегабита входящего потока

А теперь глянем мультикастовую таблицу маршрутизации: 
<code><pre>
Router#show ip mroute 
IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report, 
       Z - Multicast Tunnel, z - MDT-data group sender, 
       Y - Joined MDT-data group, y - Sending to MDT-data group, 
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute, 
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed, 
       Q - Received BGP S-A Route, q - Sent BGP S-A Route, 
       V - RD & Vector, v - Vector, p - PIM Joins on route, 
       x - VxLAN group
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 239.0.0.114), 00:14:14/stopped, RP 1.1.1.1, flags: SP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(172.17.114.1, 239.0.0.114), 00:14:14/00:02:40, flags: PT
  Incoming interface: Ethernet0/0.114, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(*, 239.0.0.113), 00:14:14/stopped, RP 1.1.1.1, flags: SP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(172.17.113.1, 239.0.0.113), 00:14:14/00:02:42, flags: PT
  Incoming interface: Ethernet0/0.113, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(*, 239.0.0.112), 00:14:14/stopped, RP 1.1.1.1, flags: SP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(172.17.112.1, 239.0.0.112), 00:14:14/00:02:43, flags: PT
  Incoming interface: Ethernet0/0.112, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(*, 239.0.0.111), 00:14:14/stopped, RP 1.1.1.1, flags: SP
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list: Null
(172.17.111.1, 239.0.0.111), 00:14:14/00:02:33, flags: PT
  Incoming interface: Ethernet0/0.111, RPF nbr 0.0.0.0
  Outgoing interface list: Null
</pre></code>

Отлично. У нас видны все запущенные мультикастовые группы и IP-адреса источников трафика, в соответствии со скриптом start_mcast.sh

Чтобы остановить вещание мультикаста на сервере, нужно выполнить команду 
<pre><code>
root@ubuntu:~# killall tsplay 
</code></pre>
