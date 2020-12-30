# Underlay. ISIS

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/HW3_topo.png)


Распределение Loopback-адресов:
| Адрес | Устройство | Интерфейс |
| ------ | ------ | ------ |
| 10.0.250.127/32 | Core | Loopback 0 |
| 10.0.250.1/32 | Spine1 | Loopback 0.0 |
| 10.0.250.2/32 | Spine2 | Loopback 0 |
| 10.0.250.3/32 | Spine3 | Loopback 0 |
| 10.0.250.128/32 | Leaf1 | Loopback 0.0 |
| 10.0.250.129/32 | Leaf2 | Loopback 0 |
| 10.0.250.130/32 | Leaf3 | Loopback 0 |
| 10.0.250.131/32 | Leaf4 | Loopback 0 |


Распределение p2p-адресов Core-Spine:
| Host | Iface | Address | Description |
| ------ | ------ | ------ | ------ |
| Core | e0/0 | 10.0.10.0/31 | to_Spine1 |
| Core | e0/1 | 10.0.10.2/31 | to_Spine2 |
| Core | e0/2 | 10.0.10.4/31 | to_Spine3 |
| Spine1 | xe-0/0/4 | 10.0.10.1/31 | to_Core |
| Spine2 | xe-0/0/4 | 10.0.10.3/31 | to_Core |
| Spine3 | xe-0/0/4 | 10.0.10.5/31 | to_Core |


Распределение p2p-адресов Spine-Leaf
| Host | Iface | Address | Description |
| ------ | ------ | ------ | ------ |
| Spine1 | xe-0/0/1 | 10.0.12.0/31 | to_Leaf1 |
| Spine1 | xe-0/0/2 | 10.0.12.2/31 | to_Leaf2 |
| Spine1 | xe-0/0/3 | 10.0.12.4/31 | to_Leaf3 |
| Spine2 | e1/1 | 10.0.12.6/31 | to_Leaf1 |
| Spine2 | e1/2 | 10.0.12.8/31 | to_Leaf2 |
| Spine2 | e1/3 | 10.0.12.10/31 | to_Leaf3 |
| Spine3 | e1/1 | 10.0.12.12/31 | to_Leaf4 |
| Leaf1 | xe-0/0/2 | 10.0.13.0/31 | to_Leaf2 |
| Leaf1 | xe-0/0/3 | 10.0.12.1/31 | to_Spine1 |
| Leaf1 | xe-0/0/4 | 10.0.12.7/31 | to_Spine2 |
| Leaf2 | e1/2 | 10.0.13.1/31 | to_Leaf1 |
| Leaf2 | e1/3 | 10.0.12.3/31 | to_Spine1 |
| Leaf2 | e1/4 | 10.0.12.9/31 | to_Spine2 |
| Leaf3 | e1/3 | 10.0.12.5/31 | to_Spine1 |
| Leaf3 | e1/4 | 10.0.12.11/31 | to_Spine2 |
| Leaf4 | e1/4 | 10.0.12.13/31 | to_Spine3 |

Для установления соседств также настроим аутентификацию при помощи md5. Пароли для area 49.0010, area 49.0020 и area 49.0030 будут otus_area10, otus_area20 и otus_area30 соответственно.

Конфигурация оборудования
<details>
  <summary>Core</summary>
<pre><code>
service password-encryption
interface Loopback0
 ip address 10.0.250.127 255.255.255.255
 ip router isis 
!
interface Ethernet0/0
 description to_Spine1
 ip address 10.0.10.0 255.255.255.254
 ip router isis 
 isis circuit-type level-2-only
 isis network point-to-point 
!
interface Ethernet0/1
 description to_Spine2
 ip address 10.0.10.2 255.255.255.254
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/2
 description to_Spine3
 ip address 10.0.10.4 255.255.255.254
 ip router isis 
 isis circuit-type level-2-only
!
router isis
 net 49.0010.0100.0025.0127.00
 is-type level-2-only
 metric-style wide
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Loopback0
</code></pre>
</details>

<details>
  <summary>Spine1</summary>
<pre><code>
routing-options {
    router-id 10.0.250.1;
}
protocols {
    isis {
        level 2 wide-metrics-only;
        interface xe-0/0/1.0 {
            point-to-point;
            level 2 disable;
        }
        interface xe-0/0/2.0 {          
            level 2 disable;
        }
        interface xe-0/0/3.0 {
            level 2 disable;
        }
        interface xe-0/0/4.0 {
            point-to-point;
            level 1 disable;
        }
        interface lo0.0;
    }
}
</code></pre>
</details>

<details>
  <summary>Spine2</summary>
<pre><code>
feature isis
router isis 1
  net 49.0020.0100.0025.0002.00
  is-type level-1-2
  distribute level-1 into level-2 all
  metric-style transition
!
interface loopback0
  ip address 10.0.250.2/32
  ip router isis 1
!
interface Ethernet1/1
  description to_Leaf1
  no switchport
  ip address 10.0.12.6/31
  isis circuit-type level-1
  ip router isis 1
  no shutdown

interface Ethernet1/2
  description to_Leaf2
  no switchport
  ip address 10.0.12.8/31
  isis network point-to-point
  isis circuit-type level-1
  ip router isis 1
  no shutdown
!
interface Ethernet1/3
  description to_Leaf3
  no switchport
  ip address 10.0.12.10/31
  isis network point-to-point
  isis circuit-type level-1
  ip router isis 1
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.3/31
  isis circuit-type level-2
  ip router isis 1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Spine3</summary>
<pre><code>
feature isis 
router isis 1
  net 49.0030.0100.0025.0003.00
  is-type level-1-2
  distribute level-1 into level-2 all
  metric-style transition
!
interface loopback0
  ip address 10.0.250.3/32
  ip router isis 1
!
interface Ethernet1/1
  description to_Leaf4
  no switchport
  ip address 10.0.12.12/31
  isis network point-to-point
  isis circuit-type level-1
  ip router isis 1
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.5/31
  isis circuit-type level-2
  ip router isis 1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf1</summary>
<pre><code>
routing-options {
    router-id 10.0.250.128;
}
protocols {
    isis {
        level 2 disable;
        level 1 wide-metrics-only;
        interface xe-0/0/2.0;
        interface xe-0/0/3.0 {
            point-to-point;
        }
        interface xe-0/0/4.0;           
        interface lo0.0;
    }
}
</code></pre>
</details>

<details>
  <summary>Leaf2</summary>
<pre><code>
feature isis 
router isis 1
  net 49.0020.0100.0025.0129.00
  is-type level-1
  metric-style transition
!
interface loopback0
  ip address 10.0.250.129/32
  ip router isis 1
!
interface Ethernet1/2
  description to_Leaf1
  no switchport
  mac-address 000c.2984.fce1
  ip address 10.0.13.1/31
  ip router isis 1
  no shutdown
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  mac-address 000c.2984.fce2
  ip address 10.0.12.3/31
  no isis hello-padding
  ip router isis 1
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  mac-address 000c.2984.fce3
  ip address 10.0.12.9/31
  isis network point-to-point
  ip router isis 1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf3</summary>
<pre><code>
feature isis 
router isis 1
  net 49.0020.0100.0025.0130.00
  is-type level-1
  metric-style transition
!
interface loopback0
  ip address 10.0.250.130/32
  ip router isis 1
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  ip address 10.0.12.5/31
  ip router isis 1
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  mac-address 000c.2985.fce0
  ip address 10.0.12.11/31
  isis network point-to-point
  ip router isis 1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf4</summary>
<pre><code>
feature isis 
router isis 1
  net 49.0030.0100.0025.0131.00
  is-type level-1
  metric-style transition
!
interface loopback0
  ip address 10.0.250.131/32
  ip router isis 1
!
interface Ethernet1/4
  description to_Spine3
  no switchport
  mac-address 000c.2984.ffe0
  ip address 10.0.12.13/31
  isis network point-to-point
  ip router isis 1
  no shutdown
!
</code></pre>
</details>

Убедимся, что на Core видны Loopback-адреса всех устройств в сети:

<pre><code>
Core#show ip route isis | include /32
i L2     10.0.250.1/32 [115/10] via 10.0.10.1, 00:00:58, Ethernet0/0
i L2     10.0.250.2/32 [115/11] via 10.0.10.3, 00:00:58, Ethernet0/1
i L2     10.0.250.3/32 [115/11] via 10.0.10.5, 00:00:58, Ethernet0/2
i L2     10.0.250.128/32 [115/20] via 10.0.10.1, 00:00:58, Ethernet0/0
i L2     10.0.250.129/32 [115/21] via 10.0.10.1, 00:00:58, Ethernet0/0
i L2     10.0.250.130/32 [115/21] via 10.0.10.1, 00:00:58, Ethernet0/0
i L2     10.0.250.131/32 [115/51] via 10.0.10.5, 00:00:58, Ethernet0/2
</code></pre>
</details>
Далее проверим, что все Spine установили isis-соседства с Leaf:
<pre><code>
root@Spine1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Leaf1          1  Up                   19
xe-0/0/2.0            Leaf2          1  Up                   23  0:c:29:84:fc:e2
xe-0/0/3.0            Leaf3          1  Up                    8  50:0:0:7:0:7
xe-0/0/4.0            Core           2  Up                   24
</code></pre>
</details>


<pre><code>
Spine2# show isis adjacency 
IS-IS process: 1 VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
Leaf1           0205.8671.6713  1      UP     00:00:22   Ethernet1/1
Leaf2           N/A             1      UP     00:00:24   Ethernet1/2
Leaf3           N/A             1      UP     00:00:25   Ethernet1/3
Core            aabb.cc00.0110  2      UP     00:00:09   Ethernet1/4
</code></pre>



<pre><code>
Spine3# show isis adjacency 
IS-IS process: 1 VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
Leaf4           N/A             1      UP     00:00:31   Ethernet1/1
Core            aabb.cc00.0120  2      UP     00:00:09   Ethernet1/4
</code></pre>

Последним этапом проверим установление isis-сессии между Leaf1 и Leaf2:

<pre><code>
root@Leaf1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/2.0            Leaf2          1  Up                   27  0:c:29:84:fc:e1
xe-0/0/3.0            Spine1         1  Up                   25
xe-0/0/4.0            Spine2         1  Up                    6  50:0:0:5:0:7
</code></pre>

Проверим связность Loobback-адресов Leaf1 и Leaf4:
<pre><code>
root@Leaf1> ping 10.0.250.131 source 10.0.250.128 rapid 
PING 10.0.250.131 (10.0.250.131): 56 data bytes
!!!!!
--- 10.0.250.131 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 19.502/52.918/171.545/59.372 ms
</code></pre>
На этом задание выполнено: все ISIS-соседства успешно установлены и маршрутизаторы успешно "видят" друг друга.
