# Underlay. BGP

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/HW4_topo.png)


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

Для настройки underlay BGP необходимо выполнить следующие условия: 
- все BGP-соседства устанавливаются через адреса, указанные на физических интерфейсах
- соседу анонсируются только connected-сети (loopback и p2p)
- для установления соседства требуется аутентификация: otus_as65000, otus_as65001, otus_as65002 и otus_as65003 - для каждой AS
- на всех роутерах активирован maximum-path 4
- Core и три Spine находятся в одной AS, чтобы не устанавливать full-mesh соседств, Core настраивается в качестве route reflector
- для leaf1 и leaf2 активируется allowas-in

На всех маршрутизаторах Cisco создан route-map bgp-out, который матчит два prefix-list: p2p - содержит в себе point-to-point адреса; loopback - содержит в себе loopback-адрес устройства. 
<pre><code>
route-map bgp-out permit 10
 match ip address prefix-list loopback p2p
!
ip prefix-list loopback seq 5 permit 10.0.250.127/32
!
ip prefix-list p2p seq 5 permit 10.0.10.0/24 le 31
</code></pre>

Для устройств Juniper политики выглядят следующим образом:
<pre><code>
policy-options {
    policy-statement Lo0-2-bgp {
        term 1 {
            from {
                protocol direct;
                interface lo0.0;
            }
            then accept;
        }
    }                                   
    policy-statement load-balance {
        term 1 {
            then {
                load-balance per-packet;
            }
        }
    }
    policy-statement p2p-2-bgp {
        term 1 {
            from {
                protocol direct;
                route-filter 10.0.10.0/23 upto /31;
                route-filter 10.0.12.0/23 upto /31;
            }
            then accept;
        }
    }
}
</code></pre>

policy-statement load-balance нужна для балансировки трафика по нескольким линкам. 


Конфигурация оборудования
<details>
  <summary>Core</summary>
<pre><code>
service password-encryption
router bgp 65000
 bgp router-id 10.0.250.127
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor Spine peer-group
 neighbor Spine remote-as 65000
 neighbor Spine password 7 011C12114834071C77191E5949
 neighbor 10.0.10.1 peer-group Spine
 neighbor 10.0.10.3 peer-group Spine
 neighbor 10.0.10.5 peer-group Spine
 !
 address-family ipv4
  redistribute connected route-map bgp-out
  neighbor Spine route-reflector-client
  neighbor 10.0.10.1 activate
  neighbor 10.0.10.3 activate
  neighbor 10.0.10.5 activate
  maximum-paths eibgp 4
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
ip prefix-list loopback seq 5 permit 10.0.250.127/32
!
ip prefix-list p2p seq 5 permit 10.0.10.0/24 le 31
!
route-map bgp-out permit 10
 match ip address prefix-list loopback p2p
</code></pre>
</details>

<details>
  <summary>Spine1</summary>
<pre><code>
routing-options {
    router-id 10.0.250.1;
    autonomous-system 65000 loops 3;
    forwarding-table {
        export load-balance;
    }
}
protocols {
    bgp {
        export [ Lo0-2-bgp p2p-2-bgp ]; 
        group Core {
            type internal;
            authentication-key "$9$JEDkPz3901h9Aw24ojituOIyl8X7NVwleYg"; ## SECRET-DATA
            peer-as 65000;
            neighbor 10.0.10.0;
        }
        group Leaf1-2 {
            type external;
            family inet {
                unicast;
            }
            authentication-key "$9$uAUk1hrM87w2a7-n9AtIRdbsgGD.P5TFnDj9A"; ## SECRET-DATA
            export [ Lo0-2-bgp p2p-2-bgp ];
            peer-as 65001;
            multipath;
            as-override;
            neighbor 10.0.12.1;
            neighbor 10.0.12.3;
        }
        group Leaf3 {
            type external;              
            family inet {
                unicast;
            }
            authentication-key "$9$3JRh9pBhSeL7Vevm5QzCAM8XNs2ZUjiqm2gTz"; ## SECRET-DATA
            peer-as 65002;
            neighbor 10.0.12.5;
        }
    }
}
</code></pre>
</details>

<details>
  <summary>Spine2</summary>
<pre><code>
feature bgp
ip prefix-list loopback seq 5 permit 10.0.250.2/32 
ip prefix-list p2p seq 5 permit 10.0.10.0/23 le 31 
ip prefix-list p2p seq 10 permit 10.0.12.0/23 le 31 
!
route-map bgp-out permit 10
  match ip address prefix-list loopback p2p 
!
router bgp 65000
  router-id 10.0.250.2
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Leaf1-2
    remote-as 65001
    password 3 18a738b2cd29822ea46d57bcca54cb43
    address-family ipv4 unicast
      allowas-in 3
      disable-peer-as-check
  template peer Leaf3
    remote-as 65002
    password 3 18a738b2cd29822e19fb22a8e7065c21
    address-family ipv4 unicast
  neighbor 10.0.10.2
    remote-as 65000
    password 3 18a738b2cd29822e2cf894444529c6c2
    address-family ipv4 unicast
  neighbor 10.0.12.7
    inherit peer Leaf1-2
  neighbor 10.0.12.9
    inherit peer Leaf1-2
  neighbor 10.0.12.11
    inherit peer Leaf3
</code></pre>
</details>

<details>
  <summary>Spine3</summary>
<pre><code>
feature bgp 
ip prefix-list loopback seq 5 permit 10.0.250.3/32 
ip prefix-list p2p seq 5 permit 10.0.10.0/23 le 31 
ip prefix-list p2p seq 10 permit 10.0.12.0/23 le 31 
route-map bgp-out permit 10
  match ip address prefix-list loopback p2p 
!
router bgp 65000
  router-id 10.0.250.3
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Core
    remote-as 65000
    password 3 18a738b2cd29822e2cf894444529c6c2
    address-family ipv4 unicast
  template peer Leaf4
    remote-as 65003
    password 3 18a738b2cd29822e98424f23475ac6ba
    address-family ipv4 unicast
  neighbor 10.0.10.4
    inherit peer Core
  neighbor 10.0.12.13
    inherit peer Leaf4
</code></pre>
</details>

<details>
  <summary>Leaf1</summary>
<pre><code>
routing-options {
    router-id 10.0.250.128;
    autonomous-system 65001 loops 3;
    forwarding-table {
        export load-balance;
    }
}
policy-options {
    policy-statement Lo0-2-bgp {
        term 1 {
            from {
                protocol direct;        
                interface lo0.0;
            }
            then accept;
        }
    }
    policy-statement load-balance {
        term 1 {
            then {
                load-balance per-packet;
            }
        }
    }
    policy-statement p2p-2-bgp {
        term 1 {
            from {
                protocol direct;
                route-filter 10.0.12.0/23 upto /31;
            }
            then accept;
        }
    }
}
protocols {
    bgp {
        export [ Lo0-2-bgp p2p-2-bgp ];
        group Spine {
            type external;
            family inet {
                unicast;
            }
            authentication-key "$9$T39p1RSv87SrH.Pfn6lKMLdb4aZGjHbw.P"; ## SECRET-DATA
            peer-as 65000;              
            multipath;
            neighbor 10.0.12.0;
            neighbor 10.0.12.6;
        }
        group Leaf2 {
            type internal;
            family inet {
                unicast;
            }
            authentication-key "$9$jzqfz/CuRcluO4JGU.mBIESvWNdbw24W8JG"; ## SECRET-DATA
            neighbor 10.0.13.1;
        }
    }
}
</code></pre>
</details>

<details>
  <summary>Leaf2</summary>
<pre><code>
feature bgp 
ip prefix-list loopback seq 5 permit 10.0.250.129/32 
ip prefix-list p2p seq 5 permit 10.0.12.0/23 le 31 
route-map bgp-out permit 10
  match ip address prefix-list loopback p2p 
!
router bgp 65001
  router-id 10.0.250.129
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Leaf1
    remote-as 65001
    password 3 18a738b2cd29822ea46d57bcca54cb43
    address-family ipv4 unicast
  template peer Spine
    remote-as 65000
    password 3 18a738b2cd29822ea46d57bcca54cb43
    address-family ipv4 unicast
      allowas-in 3
  neighbor 10.0.12.2
    inherit peer Spine
  neighbor 10.0.12.8
    inherit peer Spine
  neighbor 10.0.13.0
    inherit peer Leaf1
</code></pre>
</details>

<details>
  <summary>Leaf3</summary>
<pre><code>
feature bgp 
ip prefix-list loobpack seq 5 permit 10.0.250.130/32 
ip prefix-list p2p seq 5 permit 10.0.12.0/23 le 31 
route-map bgp-out permit 10
  match ip address prefix-list loobpack p2p 
!
router bgp 65002
  router-id 10.0.250.130
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 2
  template peer Spine
    remote-as 65000
    password 3 18a738b2cd29822e19fb22a8e7065c21
    address-family ipv4 unicast
  neighbor 10.0.12.4
    inherit peer Spine
  neighbor 10.0.12.10
    inherit peer Spine
</code></pre>
</details>

<details>
  <summary>Leaf4</summary>
<pre><code>
feature bgp 
ip prefix-list loopback seq 5 permit 10.0.250.131/32 
ip prefix-list p2p seq 5 permit 10.0.12.0/23 le 31 
route-map bgp-out permit 10
  match ip address prefix-list loopback p2p 
!
router bgp 65003
  router-id 10.0.250.131
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
  template peer Spine
    remote-as 65000
    password 3 18a738b2cd29822e98424f23475ac6ba
    address-family ipv4 unicast
  neighbor 10.0.12.12
    inherit peer Spine
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
На этом задание выполнено: все OSPF-соседства успешно установлены и маршрутизаторы успешно "видят" друг друга.
