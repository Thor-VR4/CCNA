# ISIS
Лабараторное задание №7 по работе с ISIS.

## Задание
Настроить ISIS офисе Триада и разделить сеть на зоны согласно заданию.

1. [Создание зоны 2222](#chapter-0)
2. [Создание зоны 24](#chapter-1)
3. [Создание зоны 26](#chapter-2)
4. [Проверка работоспособности](#chapter-3)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%237%20ISIS/isis.png "Стенд №7")

<a id="chapter-0"></a>
## Создание зоны 2222
Для реализации задания был выбран протокол ISIS с уровнем взаидействия 2
Пример конфигурации R25:
```
router isis 520
 net 49.2222.0000.0000.0025.00
 is-type level-2-only
!
interface Ethernet0/0
 description R23
 ip address 10.2.3.18 255.255.255.252
 ip router isis 520
 ipv6 address FE80::25 link-local
 ipv6 router isis 520
```
Состояние соседства на R23:
```
Tag 520:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.2.3.9        UP    7        R24.02             
R25            L2   Et0/1       10.2.3.18       UP    8        R25.01             
```

<a id="chapter-1"></a>
## Создание зоны 24
В зону 24 входит маршрутизатор R24:
```
router isis 520
 net 49.0024.0000.0000.0024.00
 is-type level-2-only
```

<a id="chapter-2"></a>
## Создание зоны 26
В зону 26 входит маршрутизатор R26:
```
router isis 520
 net 49.0026.0000.0000.0026.00
 is-type level-2-only
```

<a id="chapter-3"></a>
## Проверка работоспособности
Маршруты полученные через ISIS на R23:
```
R23#show ip route isis 
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

      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
i L2     10.2.3.0/30 [115/20] via 10.2.3.9, 00:28:37, Ethernet0/2
i L2     10.2.3.20/30 [115/20] via 10.2.3.18, 00:40:40, Ethernet0/1
i L2     10.2.4.24/32 [115/20] via 10.2.3.9, 00:28:37, Ethernet0/2
i L2     10.2.4.25/32 [115/20] via 10.2.3.18, 00:40:40, Ethernet0/1
i L2     10.2.4.26/32 [115/30] via 10.2.3.18, 00:26:46, Ethernet0/1
                      [115/30] via 10.2.3.9, 00:26:46, Ethernet0/2
					  
R23#show ipv6 route isis 
IPv6 Routing Table - default - 5 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
I2  2A03:5A00:1F:204::24/128 [115/20]
     via FE80::24, Ethernet0/2
I2  2A03:5A00:1F:204::25/128 [115/20]
     via FE80::25, Ethernet0/1
I2  2A03:5A00:1F:204::26/128 [115/30]
     via FE80::24, Ethernet0/2
     via FE80::25, Ethernet0/1
```
Проверим доступность маршрутизатора R26 с R23:
```
R23#ping 10.2.4.26         
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.4.26, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

Трассировка до R26 с R23:
```
R23#traceroute 2A03:5A00:1F:204::26
Type escape sequence to abort.
Tracing the route to 2A03:5A00:1F:204::26

  1 2A03:5A00:1F:204::24 2 msec
    2A03:5A00:1F:204::25 0 msec
    2A03:5A00:1F:204::24 1 msec
  2 2A03:5A00:1F:204::26 1 msec 0 msec 1 msec
```
