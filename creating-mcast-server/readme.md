В этом небольшом мануале опишу, как к себе в лабораторный стенд eve-ng добавить источник мультикаста для выполнения лабораторных работ. 
Итак, первым делом, необходимо образ себе скачать. Идем по ссылке и качаем: https://yadi.sk/d/Wn1enP3M_6iA3g
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
