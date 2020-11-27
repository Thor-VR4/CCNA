# EIGRP
Лабараторное задание №8 по работе с EIGRP.

## Задание
Настроить EIGRP офисе C.-Петербург согласно заданию.

1. [Настройка EIGRP на устройстввах](#chapter-0)
2. [Применеие политик маршрутизации](#chapter-1)
3. [Проверка работоспособности](#chapter-2)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%238%20EIGRP/EIGRP.png "Стенд №8")

<a id="chapter-0"></a>
## Настройка EIGRP на устройстввах
Для реализации задания был выбран протокол EIGRP named mode.
Пример конфигурации R18 для ipv4:
```
router eigrp SP
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  af-interface default
   shutdown
  exit-af-interface
  !
  af-interface Ethernet0/1
   no shutdown
  exit-af-interface
  !
  af-interface Ethernet0/0
   no shutdown
  exit-af-interface
  !
  af-interface Loopback0
   no shutdown
   passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.3.3.0 0.0.0.255
  network 10.3.4.0 0.0.0.255
  eigrp router-id 0.0.0.18
 exit-address-family
 !
```
Состояние соседства на R18 для IPv6:
```
EIGRP-IPv6 VR(SP) Address-Family Neighbors for AS(2042)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   Link-local address:     Et0/0                    14 00:02:14    1   100  0  552601
    FE80::16
0   Link-local address:     Et0/1                    11 00:02:19    6   100  0  134
    FE80::17         
```

<a id="chapter-1"></a>
## Применеие политик маршрутизации
Согласно заданию, R16 и R17 должны анонсировать только суммарные префиксы.
Пример суммаризации IPv6 на R17:
```
  !
  af-interface Ethernet0/1
   summary-address 2A03:5A00:1F:300::/62
   no shutdown
  exit-af-interface
  !
  af-interface Ethernet0/2
   summary-address 2A03:5A00:1F:300::/56
   no shutdown
  exit-af-interface
  !
  af-interface Ethernet0/0
   summary-address 2A03:5A00:1F:300::/56
   no shutdown
  exit-af-interface
  !
```
Таблица маршрутизации IPv6 на SW9:
```
D   2A03:5A00:1F:300::/56 [90/1024640]
     via FE80::17, Ethernet0/3
     via FE80::16, Ethernet1/0
D   2A03:5A00:1F:301::/64 [90/15360]
     via FE80::10, Vlan10
D   2A03:5A00:1F:302::10/128 [90/15360]
     via FE80::10, Vlan10
```
Для передачи на R32 только маршрута по умолчанию, на R16 была использована следующая конструкция:
```
  !
  af-interface Ethernet0/3
   summary-address 0.0.0.0 0.0.0.0
  exit-af-interface
  !       
  topology base
   summary-metric 0.0.0.0/0 distance 255
  exit-af-topology
```
Таблица маршрутизации IPv4 на R32:
```
Gateway of last resort is 10.3.3.5 to network 0.0.0.0

D*    0.0.0.0/0 [90/1024640] via 10.3.3.5, 00:05:12, Ethernet0/0
```

<a id="chapter-2"></a>
## Проверка работоспособности
Маршруты полученные через EIGRP на R16:
```
RR16# show ip route eigrp 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 20 subnets, 5 masks
D        10.3.0.0/16 is a summary, 02:05:19, Null0
D        10.3.0.0/22 is a summary, 02:05:22, Null0
D        10.3.0.0/24 [90/1029120] via 10.3.3.10, 02:05:24, Ethernet0/2
D        10.3.1.0/24 [90/1029120] via 10.3.3.2, 02:05:24, Ethernet0/0
D        10.3.2.0/24 [90/1029120] via 10.3.3.10, 02:05:24, Ethernet0/2
                     [90/1029120] via 10.3.3.2, 02:05:24, Ethernet0/0
D        10.3.3.16/30 [90/1536000] via 10.3.3.10, 02:05:24, Ethernet0/2
D        10.3.3.20/30 [90/1536000] via 10.3.3.2, 02:05:24, Ethernet0/0
D        10.3.3.24/30 [90/1536000] via 10.3.3.14, 00:19:40, Ethernet0/1
D        10.3.4.17/32 [90/1536640] via 10.3.3.14, 00:19:40, Ethernet0/1
D        10.3.4.18/32 [90/1024640] via 10.3.3.14, 00:19:24, Ethernet0/1
D        10.3.4.32/32 [90/1024640] via 10.3.3.6, 02:05:25, Ethernet0/3
					  
R16# show ipv6 route eigrp 
IPv6 Routing Table - default - 11 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
D   2A03:5A00:1F:300::/56 [90/1536640]
     via FE80::9, Ethernet0/2
D   2A03:5A00:1F:300::/62 [5/1029120]
     via Null0, directly connected
D   2A03:5A00:1F:300::/64 [90/1029120]
     via FE80::9, Ethernet0/2
D   2A03:5A00:1F:301::/64 [90/1029120]
     via FE80::10, Ethernet0/0
D   2A03:5A00:1F:302::9/128 [90/1029120]
     via FE80::9, Ethernet0/2
D   2A03:5A00:1F:302::10/128 [90/1029120]
     via FE80::10, Ethernet0/0
D   2A03:5A00:1F:304::17/128 [90/1536640]
     via FE80::18, Ethernet0/1
D   2A03:5A00:1F:304::18/128 [90/1024640]
     via FE80::18, Ethernet0/1
D   2A03:5A00:1F:304::32/128 [90/1024640]
     via FE80::32, Ethernet0/3
```
Проверим доступность маршрутизатора R32 с SW9:
```
SW9#ping 10.3.4.32
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.4.32, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

Трассировка до R18 с SW10:
```
SW10#traceroute 2A03:5A00:1F:304::18    
Type escape sequence to abort.
Tracing the route to 2A03:5A00:1F:304::18

  1 2A03:5A00:1F:304::16 1 msec
    2A03:5A00:1F:304::17 1 msec
    2A03:5A00:1F:304::16 0 msec
  2 2A03:5A00:1F:304::18 1 msec 0 msec 0 msec
```
