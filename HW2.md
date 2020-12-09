# Underlay. OSPF

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/HW2_topo.png)


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

Для установления соседств также настроим аутентификацию при помощи md5. Пароли для area 0, area 1 и area 2 будут otus_area0, otus_area1 и otus_area2 соответственно.

Конфигурация оборудования
<details>
  <summary>Core</summary>
<pre><code>
service password-encryption
router ospf 1
 router-id 10.0.250.127
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
!
interface Loopback0
 ip address 10.0.250.127 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_Spine1
 ip address 10.0.10.0 255.255.255.254
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 1418061E1F3B2B39212963
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_Spine2
 ip address 10.0.10.2 255.255.255.254
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 09435A1C0A3A16000E0D54
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_Spine3
 ip address 10.0.10.4 255.255.255.254
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 09435A1C0A3A16000E0D54
 ip ospf network point-to-point
 ip ospf 1 area 0
</code></pre>
</details>

<details>
  <summary>Spine1</summary>
<pre><code>
routing-options {
    router-id 10.0.250.1;
}
protocols {
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface xe-0/0/4.0 {      
                interface-type p2p;
                authentication {
                    md5 1 key "$9$8Nt7VYaZjm5FjHyevMN-HqmfT36/tpO1"; ## SECRET-DATA
                }
            }
        }
        area 0.0.0.1 {
            interface xe-0/0/1.0 {
                interface-type p2p;
                authentication {
                    md5 1 key "$9$qfz6tu1SlM1RZDiH5TRhSreWLX-Vs2"; ## SECRET-DATA
                }
            }
            interface xe-0/0/2.0 {
                interface-type p2p;
                authentication {
                    md5 1 key "$9$jaqfz/CuRcluO4JGU.mO1RhSevMX7db"; ## SECRET-DATA
                }
            }
            interface xe-0/0/3.0 {      
                interface-type p2p;
                authentication {
                    md5 1 key "$9$HmT3CtOhyKO1aGDjPf1RhSrvW87-bs"; ## SECRET-DATA
                }
            }
        }
    }
}
</code></pre>
</details>

<details>
  <summary>Spine2</summary>
<pre><code>
feature ospf
router ospf 1
  router-id 10.0.250.2
  passive-interface default
!
interface Ethernet1/1
  description to_Leaf1
  no switchport
  ip address 10.0.12.6/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/2
  description to_Leaf2
  no switchport
  ip address 10.0.12.8/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/3
  description to_Leaf3
  no switchport
  ip address 10.0.12.10/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.3/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521ff80b26794e247e53
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.0
  no shutdown
</code></pre>
</details>

<details>
  <summary>Spine3</summary>
<pre><code>
feature ospf 
router ospf 1
  router-id 10.0.250.3
  passive-interface default
!
interface loopback0
  ip address 10.0.250.3/32
  ip router ospf 1 area 0.0.0.0

interface Ethernet1/1
  description to_Leaf4
  no switchport
  ip address 10.0.12.12/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f606bd3c2eb9153bc
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.2
  no shutdown
!
interface Ethernet1/4
  description to_Core
  no switchport
  ip address 10.0.10.5/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521ff80b26794e247e53
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.0
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
    ospf {
        area 0.0.0.1 {
            interface lo0.0 {
                passive;
            }
            interface xe-0/0/2.0 {
                interface-type p2p;
                authentication {
                    md5 1 key "$9$Bk1EyKLxd2oGdb9tu0hcbs24aUji.PTz"; ## SECRET-DATA
                }
            }
            interface xe-0/0/3.0 {
                interface-type p2p;     
                authentication {
                    md5 1 key "$9$wogJDqmT6CuTzXNdV4oz369A0B1hSlK"; ## SECRET-DATA
                }
            }
            interface xe-0/0/4.0 {
                interface-type p2p;
                authentication {
                    md5 1 key "$9$2MaUHP5FCpBFnNVwsJZn/Ctu1REylvW"; ## SECRET-DATA
                }
            }
        }
    }
}
</code></pre>
</details>

<details>
  <summary>Leaf2</summary>
<pre><code>
feature ospf 
router ospf 1
  router-id 10.0.250.129
  passive-interface default
!
interface loopback0
  ip address 10.0.250.129/32
  ip router ospf 1 area 0.0.0.1
!
interface Ethernet1/2
  description to_Leaf1
  no switchport
  mac-address 000c.2984.fce1
  ip address 10.0.13.1/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  mac-address 000c.2984.fce2
  ip address 10.0.12.3/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  mac-address 000c.2984.fce3
  ip address 10.0.12.9/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf3</summary>
<pre><code>
feature ospf 
router ospf 1
  router-id 10.0.250.130
  passive-interface default
!
interface loopback0
  ip address 10.0.250.130/32
  ip router ospf 1 area 0.0.0.1
!
interface Ethernet1/3
  description to_Spine1
  no switchport
  ip address 10.0.12.5/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
!
interface Ethernet1/4
  description to_Spine2
  no switchport
  mac-address 000c.2985.fce0
  ip address 10.0.12.11/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f76bd6fbfadc85b2b
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.1
  no shutdown
</code></pre>
</details>

<details>
  <summary>Leaf4</summary>
<pre><code>
feature ospf 
router ospf 1
  router-id 10.0.250.131
  passive-interface default
!
interface loopback0
  ip address 10.0.250.131/32
  ip router ospf 1 area 0.0.0.2
!
interface Ethernet1/4
  description to_Spine3
  no switchport
  mac-address 000c.2984.ffe0
  ip address 10.0.12.13/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 30619c71daee521f606bd3c2eb9153bc
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 1 area 0.0.0.2
  no shutdown
!
</code></pre>
</details>

Убедимся, что на Core видны Loopback-адреса всех устройств в сети:
<details>
  <summary>Core#sh ip route ospf | include /32</summary>
<pre><code>
O        10.0.250.1/32 [110/10] via 10.0.10.1, 00:26:14, Ethernet0/0
O        10.0.250.2/32 [110/11] via 10.0.10.3, 00:24:03, Ethernet0/1
O        10.0.250.3/32 [110/11] via 10.0.10.5, 00:22:50, Ethernet0/2
O IA     10.0.250.128/32 [110/11] via 10.0.10.1, 00:21:03, Ethernet0/0
O IA     10.0.250.129/32 [110/12] via 10.0.10.1, 00:01:53, Ethernet0/0
O IA     10.0.250.130/32 [110/12] via 10.0.10.1, 00:01:37, Ethernet0/0
O IA     10.0.250.131/32 [110/51] via 10.0.10.5, 00:01:11, Ethernet0/2
</code></pre>
</details>
Далее проверим, что все Spine установили ospf-соседства с Leaf:
<details>
  <summary>root@Spine1> show ospf neighbor </summary>
<pre><code>
Address          Interface              State     ID               Pri  Dead
10.0.10.0        xe-0/0/4.0             Full      10.0.250.127       1    37
10.0.12.1        xe-0/0/1.0             Full      10.0.250.128     128    38
10.0.12.3        xe-0/0/2.0             Full      10.0.250.129       1    32
10.0.12.5        xe-0/0/3.0             Full      10.0.250.130       1    32
</code></pre>
</details>

<details>
  <summary>Spine2# sh ip ospf neighbors</summary>
<pre><code>
 OSPF Process ID 1 VRF default
 Total number of neighbors: 4
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.250.128    128 FULL/ -          00:24:54 10.0.12.7       Eth1/1 
 10.0.250.129      1 FULL/ -          00:21:34 10.0.12.9       Eth1/2 
 10.0.250.130      1 FULL/ -          00:19:20 10.0.12.11      Eth1/3 
 10.0.250.127      1 FULL/ -          00:28:07 10.0.10.2       Eth1/4 
</code></pre>
</details>

<details>
  <summary>Spine3# sh ip ospf neighbors </summary>
<pre><code>
 OSPF Process ID 1 VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.250.131      1 FULL/ -          00:17:11 10.0.12.13      Eth1/1 
 10.0.250.127      1 FULL/ -          00:27:19 10.0.10.4       Eth1/4 
</code></pre>
</details>

Последним этапом проверим установление OSPF-сессии между Leaf1 и Leaf2:
 
<details>
  <summary>root@Leaf1> show ospf neighbor</summary>
<pre><code>
Address          Interface              State     ID               Pri  Dead
10.0.13.1        xe-0/0/2.0             Full      10.0.250.129       1    32
10.0.12.0        xe-0/0/3.0             Full      10.0.250.1       128    33
10.0.12.6        xe-0/0/4.0             Full      10.0.250.2         1    32
</code></pre>
</details>
