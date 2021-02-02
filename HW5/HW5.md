# PIM Sparse Mode

Топология сети выглядит следующим образом
![Топология сети](https://github.com/bonishvarik/otus-net-arch/raw/main/HW5/HW5_topo.png)

В исходной схеме дано 6 маршрутизаторов (3 Juniper vMX и 3 Cisco IOL), 3 мультикастовых источника и один приемник. Необходимо настроить RP таким образом, что роутер vMX2 должен стать Rendezvous Point для мультикастовых групп с адресами 239.72/16, а R3 - RP для 239.73/16. Между маршрутизаторами настроен протокол OSPF: все интерфейсы в сторону соседей имеют тип сети p2p, сети источников или приемников мультикаста анонсируются в OSPF, но сами интерфейсы находятся в passive-режиме. 

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
| vMX5 | ge-0/0/3.111 | 172.18.111.2/24 | to_Source3_vl111 |
| vMX5 | ge-0/0/3.112 | 172.18.112.2/24 | to_Source3_vl112 |
| R6 | e0/0 | 10.0.0.13/31 | to_R4 |
| R6 | e0/1 | 10.0.0.9/31 | to_vMX5 |

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
  
  auto ens3
  iface ens3 inet static
     address 192.168.1.100
     netmask 255.255.255.0
     gateway 192.168.1.1
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

**root@vMX2> show route 192.168.1/24** 
```
inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.0/24     *[OSPF/10] 01:16:29, metric 12
                    > to 10.0.0.2 via ge-0/0/4.0
```
Поскольку все префиксы присутствуют в таблице маршрутизации, базовую связность можно считать обеспеченной. Переходим к основной части задания - настройке PIM Sparse Mode

------------------------------------------------------------------------------------------------------
Обмениваться информацией о RP маршрутизаторы будут с помощью протокола bootstrap. Согласно заданию, vMX2 выступает в роли RP для групп с префикса 239.72.0.0/16, а R3 - RP для 239.73.0.0/16. На Cisco и Juniper в дефолтное значение приоритета в bootstrap равно 100, а префикс по умолчанию содержит в себе все мультикастовые адреса: 224.0.0.0/4. Менять данную политику не будем, изменим лишь дефолтное значение приоритета на 180, а на vMX2 и R3 выставим Priority равным 50. 

Конфигурация PIM на Juniper vMX2 выглядит следующим образом: 
```
protocols {
    pim {
        rp {
            bootstrap {
                family inet {
                    priority 50;  ### настройка приоритета для Bootstrap IPv4
                }
            }
            local {                     
                address 10.255.0.2; ### адрес кандидата в RP (Loopback-адрес)
                group-ranges {      ### здесь объявляются префиксы для данной RP
                    239.72.0.0/16;
                }
            }
        }
        interface ge-0/0/4.0 {      ### добавление интерфейсов оборудования в PIM
            mode sparse;
        }
        interface ge-0/0/3.0 {
            mode sparse;
        }
    }
}

```

Конфигурация PIM на Cisco R3:
```
### активация мультикастовой маршрутизации
ip multicast-routing

### access-list для префиксов в данном RP
ip access-list standard MCAST_239.73.0.0
 permit 239.73.0.0 0.0.255.255
 deny   any

### Настройка Lo0-интерфейса в качестве BSR с приоритетом 50 для указанных
### в rp-candidate ACL групп
ip pim bsr-candidate Loopback0 0 50
ip pim rp-candidate Loopback0 group-list MCAST_239.73.0.0 priority 100

### Добавление интерфейсов оборудования в PIM-домен
interface Loopback0
 ip pim sparse-mode
!
interface Ethernet0/0
 ip pim sparse-mode
!
interface Ethernet0/1
 ip pim sparse-mode
!
interface Ethernet0/2
 ip pim sparse-mode
!
interface Ethernet0/3
 ip pim sparse-mode
```

Аналогичным образом настраивается оставшееся оборудование. Интерфейсы в сторону источников и приемника также добавляются в PIM, но в passive режиме для Cisco. На Juniper такая возможность тоже есть, но в более свежих версиях Junos. 
<details>
  <summary>Настройка PIM vMX1</summary>

```
protocols {
    pim {
        rp {
            bootstrap {
                family inet {
                    priority 180;
                }
            }
            local {                     
                address 10.255.0.1;
            }
        }
        interface ge-0/0/5.111 {
            mode sparse;
        }
        interface ge-0/0/5.112 {
            mode sparse;
        }
        interface ge-0/0/3.0 {
            mode sparse;
        }
        interface ge-0/0/4.0 {
            mode sparse;
        }
    }
}
```
</details>

<details>
  <summary>Настройка PIM R4</summary>

```
ip multicast-routing 

ip pim bsr-candidate Loopback0 0 180
ip pim rp-candidate Loopback0

interface Loopback0
 ip pim sparse-mode
!
interface Ethernet0/0
 ip pim sparse-mode
!
interface Ethernet0/1
 ip pim sparse-mode
!
interface Ethernet0/2
 ip pim sparse-mode
!
interface Ethernet0/3.113
 ip pim passive
!
interface Ethernet0/3.114
 ip pim passive
!
```
</details>

<details>
  <summary>Настройка PIM vMX5</summary>

```
protocols {
    pim {
        rp {
            bootstrap {
                family inet {
                    priority 180;
                }
            }
            local {
                address 10.255.0.5;
            }
        }
        interface ge-0/0/4.0 {
            mode sparse;
        }                               
        interface ge-0/0/5.0 {
            mode sparse;
        }
        interface ge-0/0/3.111 {
            mode sparse;
        }
        interface ge-0/0/3.112 {
            mode sparse;
        }
        interface ge-0/0/2.0 {
            mode sparse;
        }
    }
}
```
</details>

<details>
  <summary>Настройка PIM R6</summary>

```
ip multicast-routing 

ip pim bsr-candidate Loopback0 0 180
ip pim rp-candidate Loopback0

interface Loopback0
 ip pim sparse-mode
!
interface Ethernet0/0
 ip pim sparse-mode
!
interface Ethernet0/1
 ip pim sparse-mode
```
</details>

Убедимся, что PIM-соседства успешно установлены. Для этого достаточно будет проверить vMX1, R4 и vMX5:
**root@vMX1> show pim neighbors**

```
B = Bidirectional Capable, G = Generation Identifier
H = Hello Option Holdtime, L = Hello Option LAN Prune Delay,
P = Hello Option DR Priority, T = Tracking Bit

Instance: PIM.master
Interface           IP V Mode        Option       Uptime Neighbor addr
ge-0/0/3.0           4 2             HPG        00:32:16 10.0.0.1       
ge-0/0/4.0           4 2             HPLGT      00:32:10 10.0.0.3   
```

**R4#show ip pim neighbor** 

```
PIM Neighbor Table
Mode: B - Bidir Capable, DR - Designated Router, N - Default DR Priority,
      P - Proxy Capable, S - State Refresh Capable, G - GenID Capable
Neighbor          Interface                Uptime/Expires    Ver   DR
Address                                                            Prio/Mode
10.0.0.13         Ethernet0/0              00:34:47/00:01:23 v2    1 / DR S P G
10.0.0.10         Ethernet0/1              00:32:39/00:01:29 v2    1 / G
10.0.0.6          Ethernet0/2              00:34:47/00:01:24 v2    1 / S P G
```
**root@vMX5> show pim neighbors**
```
B = Bidirectional Capable, G = Generation Identifier
H = Hello Option Holdtime, L = Hello Option LAN Prune Delay,
P = Hello Option DR Priority, T = Tracking Bit

Instance: PIM.master
Interface           IP V Mode        Option       Uptime Neighbor addr
ge-0/0/4.0           4 2             HPG        00:32:59 10.0.0.4       
ge-0/0/5.0           4 2             HPG        00:32:59 10.0.0.9   
```

Все соседства установлены успешно, следующим этапом проверим все доступные RP в нашем PIM-домене:
<details>
  <summary>root@vMX1> show pim rps </summary>

```
Instance: PIM.master

address-family INET
RP address      Type        Mode   Holdtime Timeout Groups Group prefixes
10.255.0.1      bootstrap   sparse      150     118      0 224.0.0.0/4
10.255.0.2      bootstrap   sparse      150     118      0 239.72.0.0/16
10.255.0.3      bootstrap   sparse      150     118      0 239.73.0.0/16
10.255.0.4      bootstrap   sparse      150     118      0 224.0.0.0/4
10.255.0.5      bootstrap   sparse      150     118      0 224.0.0.0/4
10.255.0.6      bootstrap   sparse      150     118      0 224.0.0.0/4
10.255.0.1      static      sparse      150    None      0 224.0.0.0/4
```
</details>

<details>
  <summary>R4#show ip pim rp mapping</summary>

```
PIM Group-to-RP Mappings
This system is a candidate RP (v2)

Group(s) 224.0.0.0/4
  RP 10.255.0.6 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:37:00, expires: 00:02:20
  RP 10.255.0.4 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:36:58, expires: 00:02:20
  RP 10.255.0.5 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 1, holdtime 150
         Uptime: 00:34:57, expires: 00:02:23
  RP 10.255.0.1 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 1, holdtime 150
         Uptime: 00:34:57, expires: 00:02:21
Group(s) 239.72.0.0/16
  RP 10.255.0.2 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 1, holdtime 150
         Uptime: 00:34:57, expires: 00:02:23
Group(s) 239.73.0.0/16
  RP 10.255.0.3 (?), v2
    Info source: 10.255.0.6 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:36:59, expires: 00:02:23
```
</details>

Как видно выше, выводы на Juniper vMX1 и Cisco R4 совпадают: адрес 10.255.0.2 является RP для 239.72.0.0/16, а 10.255.0.3 - RP для 239.73.0.0/16. Теперь попробуем подписаться на группу 239.72.112.1:1234 с нашего приемника. Первым делом убедимся, что vMX1 действительно получает мультикаст от Source1:

<details>
  <summary>root@vMX1> show route table inet.1</summary>

```
inet.1: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

224.0.0.0/4        *[Multicast/180] 00:44:47
                      MultiResolve
224.0.0.0/24       *[Multicast/180] 00:44:47
                      MultiDiscard
232.0.0.0/8        *[Multicast/180] 00:45:24
                      MultiResolve
239.72.0.0/16      *[Multicast/180] 00:44:22
                      MultiResolve
239.72.111.1,172.17.111.1/64*[PIM/105] 00:10:09
                      Multicast (IPv4) Composite
239.72.112.1,172.17.112.1/64*[PIM/105] 00:10:09
                      Multicast (IPv4) Composite
239.73.0.0/16      *[Multicast/180] 00:43:36
                      MultiResolve
```
</details>
Маршруты есть, теперь пробуем увидеть картинку. Для этого в VLC Media Player заходим в Media -> Open Network Stream и в качетве URL вводим адрес группы в следующем формате **udp://@239.72.112.1:1234**

![Тест группы 239.72.112.1](https://github.com/bonishvarik/otus-net-arch/raw/main/HW5/test1.png)

Картинка есть, проверим мультикастовую таблицу маршрутизации на R3:

<details>
  <summary>R3#sh ip mroute</summary>

```
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

(*, 239.72.112.1), 00:16:00/stopped, RP 10.255.0.2, flags: SJC
  Incoming interface: Ethernet0/0, RPF nbr 10.0.0.0
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse-Dense, 00:01:02/00:01:58

(172.17.112.1, 239.72.112.1), 00:16:00/00:01:38, flags: JT
  Incoming interface: Ethernet0/0, RPF nbr 10.0.0.0
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse-Dense, 00:01:02/00:01:58

(*, 224.0.1.40), 01:00:49/00:02:11, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse, 01:00:48/00:02:11
```
</details>
Подписка есть, интерфейс в сторону Receiver находится в OIL. Входящий интерфейс e0/0 - в сторону vMX1. Посмотрим, как ходит трафик через него: 
<details>
  <summary>root@vMX1> show pim join 239.72.112.1 detail </summary>

```
Instance: PIM.master Family: INET
R = Rendezvous Point Tree, S = Sparse, W = Wildcard

Group: 239.72.112.1
    Source: *
    RP: 10.255.0.2
    Flags: sparse,rptree,wildcard
    Upstream interface: ge-0/0/4.0            
    Downstream neighbors:
        Interface: ge-0/0/3.0             

Group: 239.72.112.1
    Source: 172.17.112.1
    Flags: sparse,spt
    Upstream interface: ge-0/0/5.112          
    Downstream neighbors:
        Interface: ge-0/0/3.0        
```
</details>

Здесь видно, что vMX2 хоть и является RP для данной мультикастовой группы, поток с vMX1 напрямую пошел в сторону получателя через интерфейс ge-0/0/3, чтобы лишний раз не забивать сеть мультикастом. Это сработал SPT Switchover.

Возьмем теперь группу из Source3 - 239.73.111.1:1234

<details>
  <summary>R3#show ip mroute</summary>

```
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

(*, 239.73.111.1), 00:06:06/stopped, RP 10.255.0.3, flags: SJC
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse-Dense, 00:00:28/00:02:36

(172.18.111.1, 239.73.111.1), 00:02:22/00:00:37, flags: JT
  Incoming interface: Ethernet0/3, RPF nbr 10.0.0.5
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse-Dense, 00:00:28/00:02:36

(*, 224.0.1.40), 01:27:27/00:02:37, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse, 01:27:26/00:02:37
```
</details>
Incoming interface в данном случае e0/1 - интерфейс в сторону vMX5. Проверим vMX5: 
<details>
  <summary>root@vMX5> show route table inet.1 </summary>

```
inet.1: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

224.0.0.0/4        *[Multicast/180] 01:27:09
                      MultiResolve
224.0.0.0/24       *[Multicast/180] 01:27:09
                      MultiDiscard
232.0.0.0/8        *[Multicast/180] 01:27:45
                      MultiResolve
239.72.0.0/16      *[Multicast/180] 01:26:11
                      MultiResolve
239.73.0.0/16      *[Multicast/180] 01:26:11
                      MultiResolve
239.73.111.1,172.18.111.1/64*[PIM/105] 00:05:45
                      Multicast (IPv4) Composite
239.73.112.1,172.18.112.1/64*[PIM/105] 00:05:45
                      Multicast (IPv4) Composite
```
</details>

<details>
  <summary>root@vMX5>root@vMX5> show pim join 239.73.111.1 detail </summary>

```
Instance: PIM.master Family: INET
R = Rendezvous Point Tree, S = Sparse, W = Wildcard

Group: 239.73.111.1
    Source: 172.18.111.1
    Flags: sparse,spt
    Upstream interface: ge-0/0/3.111          
    Downstream neighbors:
        Interface: ge-0/0/4.0    
```
</details>

Судя по выводу консоли, мультикаст успешно "побежал" в сторону Receiver.
Напоследок, проверим наличие мультикастовых маршрутов на R4:

<details>
  <summary>R4#show ip mroute</summary>

```
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

(*, 239.72.114.1), 00:00:58/stopped, RP 10.255.0.2, flags: SPF
  Incoming interface: Ethernet0/1, RPF nbr 10.0.0.10
  Outgoing interface list: Null

(172.17.114.1, 239.72.114.1), 00:00:58/00:02:01, flags: PFT
  Incoming interface: Ethernet0/3.114, RPF nbr 0.0.0.0, Registering
  Outgoing interface list: Null

(*, 239.72.113.1), 00:00:58/stopped, RP 10.255.0.2, flags: SPF
  Incoming interface: Ethernet0/1, RPF nbr 10.0.0.10
  Outgoing interface list: Null

(172.17.113.1, 239.72.113.1), 00:00:58/00:02:01, flags: PFT
  Incoming interface: Ethernet0/3.113, RPF nbr 0.0.0.0, Registering
  Outgoing interface list: Null

(*, 224.0.1.40), 00:29:54/00:02:08, RP 0.0.0.0, flags: DCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    Loopback0, Forward/Sparse, 00:29:53/00:02:08
```
</details>

Маршруты есть. На даннм этапе задание можно считать полностью выполненным. 
