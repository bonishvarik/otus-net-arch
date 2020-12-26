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
| Leaf1 | xe-0/0/3 | 10.0.12.1/31 | to_Spine1 |
| Leaf1 | xe-0/0/4 | 10.0.12.7/31 | to_Spine2 |
| Leaf2 | e1/2 | 10.0.13.0/31 | to_Leaf3 |
| Leaf2 | e1/3 | 10.0.12.3/31 | to_Spine1 |
| Leaf2 | e1/4 | 10.0.12.9/31 | to_Spine2 |
| Leaf3 | e1/2 | 10.0.13.1/31 | to_Leaf2 |
| Leaf3 | e1/3 | 10.0.12.5/31 | to_Spine1 |
| Leaf3 | e1/4 | 10.0.12.11/31 | to_Spine2 |
| Leaf4 | e1/4 | 10.0.12.13/31 | to_Spine3 |

Для настройки underlay BGP необходимо выполнить следующие условия: 
- все BGP-соседства устанавливаются через адреса, указанные на физических интерфейсах
- соседу анонсируются только connected-сети (loopback и p2p)
- для установления соседства требуется аутентификация: otus_as65000, otus_as65001, otus_as65002. Номер AS в пароле используем тот, в которой находится вышестоящий по иерархии узел. То есть: Core -> Spine1, Spine2, Spine3 - otus_as65000; Spine1, Spine2 -> Leaf1, Leaf2, Leaf3 - otus_as65001; Spine3 -> Leaf4 - otus_as65002
- на всех роутерах активирован maximum-path 4

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

На Cisco Spine создан route-map leaf-as, в котором указываются номера AS нижестоящий узлов. Это сделано для динамического установления соседств:
<pre><code>
route-map leaf-as permit 10
  match as-number 65101-65199 
</code></pre>

Конфигурация оборудования
<details>
  <summary>Core</summary>
<pre><code>
service password-encryption
router bgp 65000
 bgp router-id 10.0.250.127
 bgp log-neighbor-changes
 bgp listen range 10.0.10.0/23 peer-group Spine
 bgp listen limit 10
 no bgp default ipv4-unicast
 neighbor Spine peer-group
 neighbor Spine remote-as 65001 alternate-as 65002 
 neighbor Spine password 7 011C12114834071C77191E5949
 !
 address-family ipv4
  redistribute connected route-map bgp-out
  neighbor Spine activate
  maximum-paths eibgp 4
 exit-address-family
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
    autonomous-system 65001 loops 3;
    forwarding-table {
        export load-balance;
    }
}
protocols {
protocols {
    bgp {
        export [ Lo0-2-bgp p2p-2-bgp ]; 
        group Core {
            type external;
            family inet {
                unicast;
            }
            authentication-key "$9$JEDkPz3901h9Aw24ojituOIyl8X7NVwleYg"; ## SECRET-DATA
            peer-as 65000;
            neighbor 10.0.10.0;
        }
        group Leaf {
            type external;
            family inet {
                unicast;
            }
            authentication-key "$9$hWXrvL-VYJUHYguBIRle4aZD.PFn/9tuPfBI"; ## SECRET-DATA
            multipath multiple-as;
            neighbor 10.0.12.1 {
                peer-as 65101;
            }
            neighbor 10.0.12.3 {
                peer-as 65102;          
            }
            neighbor 10.0.12.5 {
                peer-as 65103;
            }
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
route-map leaf-as permit 10
  match as-number 65101-65199 
!
router bgp 65001
  router-id 10.0.250.2
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Core
    remote-as 65000
    password 3 18a738b2cd29822e2cf894444529c6c2
    address-family ipv4 unicast
  template peer Leaf
    password 3 18a738b2cd29822ea46d57bcca54cb43
    address-family ipv4 unicast
  neighbor 10.0.10.2
    inherit peer Core
  neighbor 10.0.12.0/23 remote-as route-map leaf-as
    inherit peer Leaf
    address-family ipv4 unicast
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
route-map leaf-as permit 10
  match as-number 65201-65299 
!
router bgp 65002
  router-id 10.0.250.3
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Core
    remote-as 65000
    password 3 18a738b2cd29822e2cf894444529c6c2
    address-family ipv4 unicast
  template peer Leaf
    password 3 18a738b2cd29822e19fb22a8e7065c21
    address-family ipv4 unicast
  neighbor 10.0.10.4
    inherit peer Core
  neighbor 10.0.12.0/23 remote-as route-map leaf-as
    inherit peer Leaf
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
            authentication-key "$9$INlclMxNb4JDbsAuOBSyYgoZHqTz3n9Aq.uO"; ## SECRET-DATA
            peer-as 65001;
            multipath multiple-as;
            neighbor 10.0.12.0;
            neighbor 10.0.12.6;
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
router bgp 65102
  router-id 10.0.250.129
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Spine
    remote-as 65001
    password 3 18a738b2cd29822ea46d57bcca54cb43
    address-family ipv4 unicast
  neighbor 10.0.12.2
    inherit peer Spine
  neighbor 10.0.12.8
    inherit peer Spine
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
router bgp 65103
  router-id 10.0.250.130
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Spine
    remote-as 65001
    password 3 18a738b2cd29822ea46d57bcca54cb43
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
router bgp 65201
  router-id 10.0.250.131
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map bgp-out
    maximum-paths 4
  template peer Spine
    remote-as 65002
    password 3 18a738b2cd29822e19fb22a8e7065c21
    address-family ipv4 unicast
  neighbor 10.0.12.12
    inherit peer Spine
</code></pre>
</details>

Убедимся, что на Core видны Loopback-адреса всех устройств в сети:

<pre><code>
Core#sh ip route 10.0.250.0 255.255.255.0 longer-prefixes 

B        10.0.250.1/32 [20/0] via 10.0.10.1, 01:18:47
B        10.0.250.2/32 [20/0] via 10.0.10.3, 01:01:28
B        10.0.250.3/32 [20/0] via 10.0.10.5, 01:00:33
C        10.0.250.127/32 is directly connected, Loopback0
B        10.0.250.128/32 [20/0] via 10.0.10.3, 00:17:07
                         [20/0] via 10.0.10.1, 00:17:07
B        10.0.250.129/32 [20/0] via 10.0.10.3, 00:17:17
                         [20/0] via 10.0.10.1, 00:17:17
B        10.0.250.130/32 [20/0] via 10.0.10.3, 00:17:15
                         [20/0] via 10.0.10.1, 00:17:15
B        10.0.250.131/32 [20/0] via 10.0.10.5, 00:49:59
</code></pre>
</details>

Как видно выше, до Loopback-адресов Leaf1, Leaf2 и Leaf3 активны 2 маршрута - по одному через каждый Spine. 

Далее проверим, что все Spine установили BGP-соседства с Leaf:
<pre><code>
root@Spine1> show bgp summary 
Groups: 2 Peers: 4 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      23         13          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.0.10.0             65000        181        183       0       0     1:19:32 7/14/14/0            0/0/0/0
10.0.12.1             65101         43         52       0       0       17:52 2/3/3/0              0/0/0/0
10.0.12.3             65102         40         55       0       0       18:03 2/3/3/0              0/0/0/0
10.0.12.5             65103         39         53       0       0       18:02 2/3/3/0              0/0/0/0
</code></pre>
</details>


<pre><code>
Spine2# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.250.2, local AS number 65001
BGP table version is 48, IPv4 Unicast config peers 5, capable peers 4
17 network entries and 21 paths using 4184 bytes of memory
BGP attribute entries [7/1092], BGP AS path entries [6/48]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.10.2       4 65000      78      71       48    0    0 01:02:50 7         
10.0.12.7       4 65101     100      95       48    0    0 00:42:51 3         
10.0.12.9       4 65102      54      60       48    0    0 00:47:15 3         
10.0.12.11      4 65103      52      58       48    0    0 00:45:10 3       
</code></pre>


<pre><code>
Spine3# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.250.3, local AS number 65002
BGP table version is 33, IPv4 Unicast config peers 3, capable peers 2
18 network entries and 20 paths using 4144 bytes of memory
BGP attribute entries [8/1248], BGP AS path entries [6/64]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.10.4       4 65000      75      64       33    0    0 00:58:22 15        
10.0.12.13      4 65201      54      61       33    0    0 00:47:49 2    
</code></pre>

Проверим связность Loobback-адресов Leaf1 и Leaf4:
<pre><code>
root@Leaf1> ping 10.0.250.131 source 10.0.250.128 rapid 
PING 10.0.250.131 (10.0.250.131): 56 data bytes
!!!!!
--- 10.0.250.131 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 20.056/79.004/291.592/106.410 ms
</code></pre>

Проверим таблицы маршрутизации на Leaf, чтобы убедиться в наличии нескольких маршрутов "наверх":

<pre><code>
root@Leaf1> show route 10.0.250.0/24 protocol bgp active-path 

inet.0: 22 destinations, 38 routes (22 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.250.1/32      *[BGP/170] 00:19:38, localpref 100
                      AS path: 65001 I, validation-state: unverified
                    > to 10.0.12.0 via xe-0/0/3.0
10.0.250.2/32      *[BGP/170] 00:43:59, MED 0, localpref 100
                      AS path: 65001 ?, validation-state: unverified
                    > to 10.0.12.6 via xe-0/0/4.0
10.0.250.3/32      *[BGP/170] 00:43:59, localpref 100, from 10.0.12.6
                      AS path: 65001 65000 65002 ?, validation-state: unverified
                    > to 10.0.12.0 via xe-0/0/3.0
                      to 10.0.12.6 via xe-0/0/4.0
10.0.250.127/32    *[BGP/170] 00:43:59, localpref 100
                      AS path: 65001 65000 ?, validation-state: unverified
                      to 10.0.12.0 via xe-0/0/3.0
                    > to 10.0.12.6 via xe-0/0/4.0
10.0.250.129/32    *[BGP/170] 00:43:59, localpref 100, from 10.0.12.6
                      AS path: 65001 65102 ?, validation-state: unverified
                    > to 10.0.12.0 via xe-0/0/3.0
                      to 10.0.12.6 via xe-0/0/4.0
10.0.250.130/32    *[BGP/170] 00:43:59, localpref 100, from 10.0.12.6
                      AS path: 65001 65103 ?, validation-state: unverified
                    > to 10.0.12.0 via xe-0/0/3.0
                      to 10.0.12.6 via xe-0/0/4.0
10.0.250.131/32    *[BGP/170] 00:43:59, localpref 100, from 10.0.12.6
                      AS path: 65001 65000 65002 65201 ?, validation-state: unverified
                    > to 10.0.12.0 via xe-0/0/3.0
                      to 10.0.12.6 via xe-0/0/4.0
</code></pre>

Таблица маршрутизации Leaf2:
<pre><code>
Leaf2# sh ip route 10.0.250.0/24 longer-prefixes bgp-65102 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.250.1/32, ubest/mbest: 1/0
    *via 10.0.12.2, [20/0], 00:20:49, bgp-65102, external, tag 65001
10.0.250.2/32, ubest/mbest: 1/0
    *via 10.0.12.8, [20/0], 00:49:24, bgp-65102, external, tag 65001
10.0.250.3/32, ubest/mbest: 2/0
    *via 10.0.12.2, [20/0], 00:20:49, bgp-65102, external, tag 65001
    *via 10.0.12.8, [20/0], 00:49:24, bgp-65102, external, tag 65001
10.0.250.127/32, ubest/mbest: 2/0
    *via 10.0.12.2, [20/0], 00:20:49, bgp-65102, external, tag 65001
    *via 10.0.12.8, [20/0], 00:49:24, bgp-65102, external, tag 65001
10.0.250.128/32, ubest/mbest: 2/0
    *via 10.0.12.2, [20/0], 00:20:37, bgp-65102, external, tag 65001
    *via 10.0.12.8, [20/0], 00:45:00, bgp-65102, external, tag 65001
10.0.250.130/32, ubest/mbest: 2/0
    *via 10.0.12.2, [20/0], 00:20:46, bgp-65102, external, tag 65001
    *via 10.0.12.8, [20/0], 00:47:19, bgp-65102, external, tag 65001
10.0.250.131/32, ubest/mbest: 2/0
    *via 10.0.12.2, [20/0], 00:20:49, bgp-65102, external, tag 65001
    *via 10.0.12.8, [20/0], 00:49:24, bgp-65102, external, tag 65001
</code></pre>

И Leaf3: 
<pre><code>
Leaf3# show ip route 10.0.250.0/24 longer-prefixes bgp-65103 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.250.1/32, ubest/mbest: 1/0
    *via 10.0.12.4, [20/0], 00:21:27, bgp-65103, external, tag 65001
10.0.250.2/32, ubest/mbest: 1/0
    *via 10.0.12.10, [20/0], 00:47:59, bgp-65103, external, tag 65001
10.0.250.3/32, ubest/mbest: 2/0
    *via 10.0.12.4, [20/0], 00:21:27, bgp-65103, external, tag 65001
    *via 10.0.12.10, [20/0], 00:47:59, bgp-65103, external, tag 65001
10.0.250.127/32, ubest/mbest: 2/0
    *via 10.0.12.4, [20/0], 00:21:27, bgp-65103, external, tag 65001
    *via 10.0.12.10, [20/0], 00:47:59, bgp-65103, external, tag 65001
10.0.250.128/32, ubest/mbest: 2/0
    *via 10.0.12.4, [20/0], 00:21:18, bgp-65103, external, tag 65001
    *via 10.0.12.10, [20/0], 00:45:40, bgp-65103, external, tag 65001
10.0.250.129/32, ubest/mbest: 2/0
    *via 10.0.12.4, [20/0], 00:21:27, bgp-65103, external, tag 65001
    *via 10.0.12.10, [20/0], 00:47:59, bgp-65103, external, tag 65001
10.0.250.131/32, ubest/mbest: 2/0
    *via 10.0.12.4, [20/0], 00:21:27, bgp-65103, external, tag 65001
    *via 10.0.12.10, [20/0], 00:47:59, bgp-65103, external, tag 65001
</code></pre>

На этом задание выполнено: все BGP-соседства установлены и маршрутизаторы "видят" друг друга.
