# PIM Sparse Mode

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/HW5/HW5_topo.png)

В исходной схеме дано 6 маршрутизаторов (3 Juniper vMX и 3 Cisco IOL), 3 мультикастовых источника и один приемник. Source3 и Receiver находятся в broadcast-среде с двумя роутерами. В сети с Source3 vMX5 должен стать Designated Router, а в сети с Receiver DR должен быть R3. Необходимо настроить RP таким образом, что роутер vMX2 должен стать Rendezvous Point для мультикастовых групп с адресами 239.72/16, а R3 - RP для 239.73/16. Между маршрутизаторами настроен протокол OSPF, все интерфейсы в сторону роутеров в режиме p2p, сети источников или приемников мультикаста анонсируются в OSPF, но находятся в passive-режиме. 

Распределение Loopback-адресов:
| Адрес | Устройство | Интерфейс |
| ------ | ------ | ------ |
| 10.255.0.1/32 | vMX1 | Loopback0.0 |
| 10.255.0.2/32 | vMX2 | Loopback0.0 |
| 10.255.0.3/32 | R3 | Loopback 0 |
| 10.255.0.4/32 | R4 | Loopback 0 |
| 10.255.0.5/32 | vMX5 | Loopback0.0 |
| 10.255.0.6/32 | R6 | Loopback 0 |

Распределение p2p-адресов
| Host | Iface | Address | Description |
| ------ | ------ | ------ | ------ |
| vMX1 | ge-0/0/5.111 | 172.17.111.2/24 | to_Source1_vl111 |
| vMX1 | ge-0/0/5.112 | 172.17.112.2/24 | to_Source1_vl112 |
| vMX1 | ge-0/0/3 | 10.0.0.0/31 | to_R3 |
| vMX1 | ge-0/0/4 | 10.0.0.2/31 | to_vMX2 |
| vMX2 | ge-0/0/4 | 10.0.0.3/31 | to_vMX1 |
| vMX2 | ge-0/0/3 | 10.0.0.10/31 | to_R4 |
| R3 | e0/0 | 10.0.0.1/31 | to_vMX1 |
| R3 | e0/1 | 192.168.1.2/24 | to_Receiver |
| R3 | e0/2 | 10.0.0.6/31 | to_R4 |
| R3 | e0/3 | 10.0.0.4/31 | to_vMX5 |
| R4 | e0/0 | 10.0.0.12/31 | to_R5 |
| R4 | e0/1 | 10.0.0.11/31 | to_vMX2 |
| R4 | e0/2 | 10.0.0.7/31 | to_R3 |
| R4 | e0/3.113 | 172.17.113.1/24 | to_Source2_vl113 |
| R4 | e0/3.114 | 172.17.114.1/24 | to_Source2_vl114 |
| vMX5 | ge-0/0/4 | 10.0.0.5/31 | to_R3 |
| vMX5 | ge-0/0/5 | 10.0.0.8/31 | to_R6 |
| vMX5 | ge-0/0/2 | 192.168.1.1/24 | to_Receiver |
| vMX5 | ge-0/0/3.111 | 172.18.111.2/24 | to_Source3_vl111 |
| vMX5 | ge-0/0/3.112 | 172.18.112.2/24 | to_Source3_vl112 |
| R6 | e0/0 | 10.0.0.13/31 | to_R4 |
| R6 | e0/1 | 10.0.0.9/31 | to_vMX5 |
| R6 | e0/2.111 | 172.18.111.3/24 | to_Source3_vl111 |
| R6 | e0/2.112 | 172.18.112.3/24 | to_Source3_vl112 |

Конфигурация серверов вещания:
| Host | Iface | Address | Multicast Group |
| ------ | ------ | ------ | ------ |
| Source1 | vlan111 | 172.17.111.1/24 | 239.72.111.1 |
| Source1 | vlan112 | 172.17.112.1/24 | 239.72.112.1 |
| Source2 | vlan113 | 172.17.113.1/24 | 239.72.113.1 |
| Source2 | vlan114 | 172.17.114.1/24 | 239.72.114.1 |
| Source3 | vlan111 | 172.18.111.1/24 | 239.73.111.1 |
| Source3 | vlan112 | 172.18.112.1/24 | 239.73.112.1 |

IP адрес на Receiver:
| Host | Iface | Address |
| ------ | ------ | ------ |
| Receiver | ens3 | 192.168.1.100/24 |


Конфигурация сети на источниках мультикаста: 
<details>
  <summary>Source1</summary>

```
  auto lo0
  iface lo0 inet loopback

  auto ens3
  iface ens3 inet manual
  
  auto vlan111
  iface vlan111 inet static
     address 172.17.111.1
     netmask 255.255.255.0
     vlan_raw_device ens3
     
  auto vlan112
  iface vlan112 inet static
     address 172.17.112.1
     netmask 255.255.255.0
     vlan_raw_device ens3
```
</details>

<details>
  <summary>Source2</summary>

```
  auto lo0
  iface lo0 inet loopback

  auto ens3
  iface ens3 inet manual
  
  auto vlan113
  iface vlan113 inet static
     address 172.17.113.1
     netmask 255.255.255.0
     vlan_raw_device ens3
     
  auto vlan114
  iface vlan114 inet static
     address 172.17.114.1
     netmask 255.255.255.0
     vlan_raw_device ens3
```
</details>
  
  <details>
  <summary>Source3</summary>

```
  auto lo0
  iface lo0 inet loopback

  auto ens3
  iface ens3 inet manual
  
  auto vlan111
  iface vlan111 inet static
     address 172.18.111.1
     netmask 255.255.255.0
     vlan_raw_device ens3
     
  auto vlan112
  iface vlan112 inet static
     address 172.18.112.1
     netmask 255.255.255.0
     vlan_raw_device ens3
```
</details>

Конфигурация сети Receiver выглядит так: 
<details>
  <summary>Receiver</summary>

```
  auto lo0
  iface lo0 inet loopback
  
  ### NAT-интерфейс для доступа в интернет
  auto ens3
  iface ens3 inet dhcp
  
  auto ens4
  iface ens4 inet static
     address 192.168.1.100
     netmask 255.255.255.0
     
     ### Маршрут для просмотра мультикаста
     post-up /usr/sbin/ip route add 239.0.0.0/8 dev ens4
     
     ### Маршрут для доступа к IP-адресам роутеров, для проверки ICMP-связности
     post-up /usr/sbin/ip route add 10.0.0.0/8 dev ens4   
```
</details>

Конфигурация маршрутизаторов: 
<details>
  <summary>vMX1</summary>

```
interfaces {
    ge-0/0/3 {
        unit 0 {
            description to_R3;          
            family inet {
                address 10.0.0.0/31;
            }
        }
    }
    ge-0/0/4 {
        unit 0 {
            description to_vMX2;
            family inet {
                address 10.0.0.2/31;
            }
        }
    }
    ge-0/0/5 {
        vlan-tagging;
        unit 111 {
            description to_Source1_vl111;
            vlan-id 111;
            family inet {
                address 172.17.111.2/24;
            }
        }
        unit 112 {                      
            description to_Source1_vl112;
            vlan-id 112;
            family inet {
                address 172.17.112.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.0.1/32;
            }
        }
    }
}
routing-options {
    router-id 10.255.0.1;
}
protocols {
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;                
            }
            interface ge-0/0/5.111 {
                passive;
            }
            interface ge-0/0/5.112 {
                passive;
            }
            interface ge-0/0/3.0 {
                interface-type p2p;
            }
            interface ge-0/0/4.0 {
                interface-type p2p;
            }
        }
    }
}
```
</details>

<details>
  <summary>vMX2</summary>

```
interfaces {
    ge-0/0/3 {
        unit 0 {
            description to_R4;          
            family inet {
                address 10.0.0.10/31;
            }
        }
    }
    ge-0/0/4 {
        unit 0 {
            description to_vMX1;
            family inet {
                address 10.0.0.3/31;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.0.2/32;
            }
        }
    }
}
routing-options {
    router-id 10.255.0.2;               
}
protocols {
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface ge-0/0/4.0 {
                interface-type p2p;
            }
            interface ge-0/0/3.0 {
                interface-type p2p;
            }
        }
    }
}
```
</details>

<details>
  <summary>R3</summary>

```
interface Loopback0
 ip address 10.255.0.3 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_vMX1
 ip address 10.0.0.1 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_Receiver
 ip address 192.168.1.2 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_R4
 ip address 10.0.0.6 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/3
 description to_vMX5
 ip address 10.0.0.4 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.255.0.3
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
```
</details>

<details>
  <summary>R4</summary>

```
interface Loopback0
 ip address 10.255.0.4 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_R5
 ip address 10.0.0.12 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_vMX2
 ip address 10.0.0.11 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_R3
 ip address 10.0.0.7 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/3.113
 description to_Source2_vl113
 encapsulation dot1Q 113
 ip address 172.17.113.2 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/3.114
 description to_Source2_vl114
 encapsulation dot1Q 114
 ip address 172.17.114.2 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.255.0.4
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
```
</details>

<details>
  <summary>vMX5</summary>

```
interfaces {
    ge-0/0/2 {
        unit 0 {
            description to_Receiver;    
            family inet {
                address 192.168.1.1/24;
            }
        }
    }
    ge-0/0/3 {
        vlan-tagging;
        unit 111 {
            description to_Source3_vl111;
            vlan-id 111;
            family inet {
                address 172.18.111.2/24;
            }
        }
        unit 112 {
            description to_Source3_vl112;
            vlan-id 112;
            family inet {
                address 172.18.112.2/24;
            }
        }
    }
    ge-0/0/4 {                          
        unit 0 {
            description to_R3;
            family inet {
                address 10.0.0.5/31;
            }
        }
    }
    ge-0/0/5 {
        unit 0 {
            description to_R6;
            family inet {
                address 10.0.0.8/31;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.255.0.5/32;
            }
        }
    }
}                                       
routing-options {
    router-id 10.255.0.5;
}
protocols {
    ospf {
        area 0.0.0.0 {
            interface lo0.0 {
                passive;
            }
            interface ge-0/0/3.111 {
                passive;
            }
            interface ge-0/0/3.112 {
                passive;
            }
            interface ge-0/0/4.0 {
                interface-type p2p;
            }
            interface ge-0/0/5.0 {
                interface-type p2p;
            }
            interface ge-0/0/2.0 {
                passive;                
            }
        }
    }
}
```
</details>

<details>
  <summary>R6</summary>

```
interface Loopback0
 ip address 10.255.0.6 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_R4
 ip address 10.0.0.13 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_vMX5
 ip address 10.0.0.9 255.255.255.254
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/2
 no ip address
!
interface Ethernet0/2.111
 description to_Source3_vl111
 encapsulation dot1Q 111
 ip address 172.18.111.3 255.255.255.0
 ip ospf 1 area 0
!         
interface Ethernet0/2.112
 description to_Source3_vl112
 encapsulation dot1Q 112
 ip address 172.18.112.3 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.255.0.6
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
```
</details>

Убедимся, что все роутеры у нас перешли в FULL-состояние со своими OSPF-соседями:

**root@vMX1> show ospf neighbor**
```
Address          Interface              State     ID               Pri  Dead
10.0.0.1         ge-0/0/3.0             Full      10.255.0.3         1    37
10.0.0.3         ge-0/0/4.0             Full      10.255.0.2       128    33
```
**root@vMX2> show ospf neighbor**
```
Address          Interface              State     ID               Pri  Dead
10.0.0.11        ge-0/0/3.0             Full      10.255.0.4         1    32
10.0.0.2         ge-0/0/4.0             Full      10.255.0.1       128    39
```
**R3#sh ip ospf neighbor**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.5        0   FULL/  -        00:00:31    10.0.0.5        Ethernet0/3
10.255.0.4        0   FULL/  -        00:00:34    10.0.0.7        Ethernet0/2
10.255.0.1        0   FULL/  -        00:00:32    10.0.0.0        Ethernet0/0
```
**R4#sh ip ospf neighbor** 
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.3        0   FULL/  -        00:00:39    10.0.0.6        Ethernet0/2
10.255.0.2        0   FULL/  -        00:00:33    10.0.0.10       Ethernet0/1
10.255.0.6        0   FULL/  -        00:00:35    10.0.0.13       Ethernet0/0
```
**root@vMX5> show ospf neighbor**
```
Address          Interface              State     ID               Pri  Dead
10.0.0.4         ge-0/0/4.0             Full      10.255.0.3         1    35
10.0.0.9         ge-0/0/5.0             Full      10.255.0.6         1    34
```
**R6#show ip ospf neighbor** 
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.5        0   FULL/  -        00:00:33    10.0.0.8        Ethernet0/1
10.255.0.4        0   FULL/  -        00:00:38    10.0.0.12       Ethernet0/0
```

Как видно выше, все соседства успешно установлены. Проверим на любом из роутеров, что в таблице маршрутизации есть все сети (loopback, Sources, Receiver):
**root@vMX2> show route 172/8** 
```

inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.17.111.0/24    *[OSPF/10] 01:17:04, metric 2
                    > to 10.0.0.2 via ge-0/0/4.0
172.17.112.0/24    *[OSPF/10] 01:17:04, metric 2
                    > to 10.0.0.2 via ge-0/0/4.0
172.17.113.0/24    *[OSPF/10] 01:10:28, metric 11
                    > to 10.0.0.11 via ge-0/0/3.0
172.17.114.0/24    *[OSPF/10] 01:09:49, metric 11
                    > to 10.0.0.11 via ge-0/0/3.0
172.18.111.0/24    *[OSPF/10] 00:57:21, metric 13
                    > to 10.0.0.2 via ge-0/0/4.0
172.18.112.0/24    *[OSPF/10] 00:57:21, metric 13
                    > to 10.0.0.2 via ge-0/0/4.0
```
Проверим наличие всех Loopback-адресов:
**root@vMX2> show route 10.255/16**
```
inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.255.0.1/32      *[OSPF/10] 01:18:11, metric 1
                    > to 10.0.0.2 via ge-0/0/4.0
10.255.0.2/32      *[Direct/0] 01:18:21
                    > via lo0.0
10.255.0.3/32      *[OSPF/10] 01:16:16, metric 3
                    > to 10.0.0.2 via ge-0/0/4.0
10.255.0.4/32      *[OSPF/10] 01:12:26, metric 2
                    > to 10.0.0.11 via ge-0/0/3.0
10.255.0.5/32      *[OSPF/10] 00:58:28, metric 12
                    > to 10.0.0.2 via ge-0/0/4.0
10.255.0.6/32      *[OSPF/10] 01:03:37, metric 12
                    > to 10.0.0.11 via ge-0/0/3.0
```
и теперь сеть Reveiver'а: 
**root@vMX2> show route 192.168.1/24 
```
inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.0/24     *[OSPF/10] 01:16:29, metric 12
                    > to 10.0.0.2 via ge-0/0/4.0
```
Поскольку все префиксы присутствуют в таблице маршрутизации, базовую связность можно считать обеспеченной. Переходим к основной части задания - настройке PIM Sparse Mode
