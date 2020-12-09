# Проектирование адресного пространства

Подсети для оборудования:
| Подсеть | Назначение | Маска сети |
| ------ | ------ | ------ |
| 10.0.0.0/23 | p2p-соединение Spine - Core| /31 |
| 10.0.12.0/23 | p2p-соединение Spile - Leaf | /31 |
| 10.0.250.0/25 | Loopback-интерфейсы Spine/Core | /32 |
| 10.0.250.128/25 | Loopback-интерфейсы Leaf | /32 |

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/OtusTopo.png)


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
| Core | e0/0 | 10.0.10.2/31 | to_Spine2 |
| Core | e0/0 | 10.0.10.4/31 | to_Spine3 |
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

Конфигурация оборудования
<details>
  <summary>Core</summary>
<pre><code>
interface Loopback0
 ip address 10.0.250.127 255.255.255.255
!
interface Ethernet0/0
 description to_Spine1
 ip address 10.0.10.0 255.255.255.254
!
interface Ethernet0/1
 description to_Spine2
 ip address 10.0.10.2 255.255.255.254
!
interface Ethernet0/2
 description to_Spine3
 ip address 10.0.10.4 255.255.255.254
!
</code></pre>
</details>

<details>
  <summary>Spine1</summary>
<pre><code>
interfaces {
    xe-0/0/1 {
        unit 0 {
            description to_Spine1;
            family inet {
                address 10.0.12.0/31;
            }
        }
    }
    xe-0/0/2 {
        unit 0 {
            description to_Spine2;
            family inet {
                address 10.0.12.2/31;
            }
        }
    }                                   
    xe-0/0/3 {
        unit 0 {
            description to_Spine3;
            family inet {
                address 10.0.12.4/31;
            }
        }
    }
    xe-0/0/4 {
        unit 0 {
            description to_Core;
            family inet {
                address 10.0.10.1/31;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.250.1/32;
            }
       }
    }
}
</code></pre>
</details>

<details>
  <summary>Spine2</summary>
<pre><code>
interface loopback0
  ip address 10.0.250.2/32
!
interface Ethernet1/1
  description to_Leaf1
  no switchport
  ip address 10.0.12.6/31
  no shutdown
!
interface Ethernet1/2
  description to_Leaf2
  no switchport
  ip address 10.0.12.8/31
  no shutdown
!
interface Ethernet1/3
  description to_Leaf3
  no switchport
  ip address 10.0.12.10/31
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.3/31
  no shutdown
!
</code></pre>
</details>

<details>
  <summary>Spine3</summary>
<pre><code>
interface loopback0
  ip address 10.0.250.3/32
!
interface Ethernet1/1
  description to_Leaf4
  no switchport
  ip address 10.0.12.12/31
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.5/31
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf1</summary>
<pre><code>
interfaces {
    xe-0/0/2 {
        unit 0 {
            description to_Leaf2;
            family inet {
                address 10.0.13.0/31;
            }
        }
    }                                   
    xe-0/0/3 {
        unit 0 {
            description to_Spine1;
            family inet {
                address 10.0.12.1/31;
            }
        }
    }
    xe-0/0/4 {
        unit 0 {
            description to_Spine2;
            family inet {
                address 10.0.12.7/31;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.250.128/32;
            }
       }
    }
}
</code></pre>
</details>

<details>
  <summary>Leaf2</summary>
<pre><code>
interface loopback0
  ip address 10.0.250.129/32
!
interface Ethernet1/2
  description to_Leaf1
  no switchport
  ip address 10.0.13.1/31
  no shutdown
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  ip address 10.0.12.3/31
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  ip address 10.0.12.9/31
  no shutdown
!
</code></pre>
</details>

<details>
  <summary>Leaf3</summary>
<pre><code>
interface loopback0
  ip address 10.0.250.130/32
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  ip address 10.0.12.5/31
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  ip address 10.0.12.11/31
  no shutdown
!
</code></pre>
</details>

<details>
  <summary>Leaf4</summary>
<pre><code>
interface loopback0
  ip address 10.0.250.131/32
!
interface Ethernet1/4
  description to_Spine3
  no switchport
  ip address 10.0.12.13/31
  no shutdown
!
</code></pre>
</details>
