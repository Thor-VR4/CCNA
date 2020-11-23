# OSPF
Лабараторное задание №6 по работе с OSPF.

## Задание
Настроить OSPF офисе Москва, hазделить сеть на зоны и настроить фильтрацию между зонами согласно заданию.

1. [Создание зоны 0](#chapter-0)
2. [Создание зоны 10](#chapter-1)
3. [Создание зоны 101](#chapter-2)
4. [Создание зоны 102](#chapter-3)
5. [Проверка работоспособности](#chapter-3)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%236%20OSPF/OSPF.png "Стенд №6")

Для реализации задания был выбран протокол ospfv3 с двумя address family

<a id="chapter-0"></a>
## Создание зоны 0
Участники зоны 0
Устройство | Интерфейс 
--- | --- 
R14 | Ethernet0/0
R14 | Ethernet0/1
R14 | Loopback 0
R15 | Ethernet0/0
R15 | Ethernet0/1
R15 | Loopback 0
R12 | Ethernet0/2
R12 | Ethernet0/3
R12 | Loopback 0
R13 | Ethernet0/2
R13 | Ethernet0/3
R13 | Loopback 0

Пример конфигурации на R14:
```
router ospfv3 1001
 router-id 0.0.0.14
 !
 address-family ipv4 unicast
  default-information originate always
  area 101 stub no-summary
 exit-address-family
 !
 address-family ipv6 unicast
  default-information originate always
  area 101 stub no-summary
 exit-address-family
 !
interface Ethernet0/0
 description R12
 ip address 10.0.3.14 255.255.255.252
 ipv6 address FE80::14 link-local
 ospfv3 network point-to-point
 ospfv3 1001 ipv6 area 0
 ospfv3 1001 ipv4 area 0
```
Соседство R14 c другими маршрутизаторами:
```
R14#show ospfv3 neighbor 

          OSPFv3 1001 address-family ipv4 (router-id 0.0.0.14)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
0.0.0.13          0   FULL/  -        00:00:31    6               Ethernet0/1
0.0.0.12          0   FULL/  -        00:00:35    5               Ethernet0/0
0.0.0.19          0   FULL/  -        00:00:32    3               Ethernet0/3

          OSPFv3 1001 address-family ipv6 (router-id 0.0.0.14)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
0.0.0.13          0   FULL/  -        00:00:35    6               Ethernet0/1
0.0.0.12          0   FULL/  -        00:00:32    5               Ethernet0/0
0.0.0.19          0   FULL/  -        00:00:31    3               Ethernet0/3
```

<a id="chapter-1"></a>
## Создание зоны 10
Так как взаимодействие с другими сетями в зоне 10 происходит исключительно через OSPF зону 0, то area 10 создана как Totaly Stub.
Участники зоны 10
Устройство | Интерфейс 
--- | --- 
R12 | Ethernet0/0
R12 | Ethernet0/1
R13 | Ethernet0/0
R13 | Ethernet0/1
SW4 | Ethernet1/0
SW4 | Ethernet1/1
SW4 | Vlan 10
SW4 | Vlan 20
SW4 | Vlan 21
SW5 | Ethernet1/0
SW5 | Ethernet1/1
SW5 | Vlan 10
SW5 | Vlan 20
SW5 | Vlan 21

Пример конфигурации на SW4:
```
router ospfv3 1001
 router-id 0.0.0.4
 !
 address-family ipv4 unicast
  passive-interface Vlan10
  passive-interface Vlan20
  passive-interface Vlan21
  area 10 stub
 exit-address-family
 !
 address-family ipv6 unicast
  passive-interface Vlan10
  passive-interface Vlan20
  passive-interface Vlan21
  area 10 stub
 exit-address-family
```
Для уменьшения маршрутной информации, передаваемой в другие зоны, на R12 и R13 используется суммаризация:
```
area 10 range 10.0.0.0 255.255.252.0
area 10 range 2A03:5A00:1F::/62
```
Так как суммарный маршрут имеет больший диапазон, чем используемые сети в area 10 и присутствует маршрут по умолчанию, для предотвращения петель используются тупиковые роуты на R12 и R13:
```
ip route 10.0.3.0 255.255.255.0 Null0
ipv6 route 2A03:5A00:1F:3::/64 Null0
``` 

<a id="chapter-2"></a>
## Создание зоны 101
Исходя из условий задания, зона 101 создана как Totaly Stub
Пример конфигурации на R19:
```
router ospfv3 1001
 router-id 0.0.0.19
 !
 address-family ipv4 unicast
  area 101 stub
 exit-address-family
 !
 address-family ipv6 unicast
  area 101 stub
 exit-address-family
```
Маршрутная информация, полученная по ospf, на R19 по обоим протоколам:
```
R19#show ip route ospfv3 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.0.3.41 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.0.3.41, 02:46:38, Ethernet0/0

R19#show ipv6 route ospf   
IPv6 Routing Table - default - 3 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
OI  ::/0 [110/11]
     via FE80::14, Ethernet0/0
```

<a id="chapter-3"></a>
## Создание зоны 102
Исходя из условий задания, зона 102 создана как обычная area.
Для выполнения поставленных условий на R15 применена фильтрация:
```
ip prefix-list 101_deny: 3 entries
   seq 4 deny 10.0.3.40/30
   seq 5 deny 10.0.4.19/32
   seq 10 permit 0.0.0.0/0 le 32
ipv6 prefix-list 101_deny_v6: 2 entries
   seq 5 deny 2A03:5A00:1F:4::19/128
   seq 10 permit ::/0 le 128
!
router ospfv3 1001
 router-id 0.0.0.15
 !
 address-family ipv4 unicast
  default-information originate always
  area 102 filter-list prefix 101_deny in
 exit-address-family
 !
 address-family ipv6 unicast
  default-information originate always
  area 102 filter-list prefix 101_deny_v6 in
 exit-address-family
```
Маршрутная информация, полученная по ospf, на R20 по обоим протоколам:
```
R20#show ipv6 route ospf          
IPv6 Routing Table - default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
OE2 ::/0 [110/1], tag 1001
     via FE80::15, Ethernet0/0
OI  2A03:5A00:1F::/62 [110/31]
     via FE80::15, Ethernet0/0
OI  2A03:5A00:1F:4::12/128 [110/20]
     via FE80::15, Ethernet0/0
OI  2A03:5A00:1F:4::13/128 [110/20]
     via FE80::15, Ethernet0/0
OI  2A03:5A00:1F:4::14/128 [110/30]
     via FE80::15, Ethernet0/0
OI  2A03:5A00:1F:4::15/128 [110/10]
     via FE80::15, Ethernet0/0
	 
R20#show ip route ospfv3          
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.0.3.33 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.3.33, 02:05:59, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 12 subnets, 3 masks
O IA     10.0.0.0/22 [110/40] via 10.0.3.33, 01:56:34, Ethernet0/0
O IA     10.0.3.8/30 [110/20] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.3.12/30 [110/30] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.3.24/30 [110/30] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.3.28/30 [110/20] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.4.12/32 [110/20] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.4.13/32 [110/20] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.4.14/32 [110/30] via 10.0.3.33, 02:06:04, Ethernet0/0
O IA     10.0.4.15/32 [110/10] via 10.0.3.33, 02:06:04, Ethernet0/0
```

<a id="chapter-4"></a>
## Проверка работоспособности
Проверим доступность маршрутизатора R19 с VPC7:
```
84 bytes from 10.0.4.19 icmp_seq=1 ttl=252 time=2.958 ms
84 bytes from 10.0.4.19 icmp_seq=2 ttl=252 time=2.446 ms
84 bytes from 10.0.4.19 icmp_seq=3 ttl=252 time=4.604 ms
84 bytes from 10.0.4.19 icmp_seq=4 ttl=252 time=8.409 ms
84 bytes from 10.0.4.19 icmp_seq=5 ttl=252 time=2.058 ms
```

Трассировка до R20 с VPC1:
```
trace to 2A03:5A00:1F:4::20, 64 hops max
 1 2a03:5a00:1f::4   1.126 ms  0.557 ms  0.521 ms
 2 2a03:5a00:1f:4::12   1.817 ms  1.110 ms  0.872 ms
 3 2a03:5a00:1f:4::15   1.710 ms  1.180 ms  1.037 ms
 4 2a03:5a00:1f:4::20   1.307 ms  1.028 ms  1.008 ms
```

Трассировка до внешней сети с VPC1:
```
trace to 8.8.8.8, 8 hops max (ICMP), press Ctrl+C to stop
 1   10.0.0.5   0.660 ms  0.519 ms  0.568 ms
 2   10.0.3.21   1.437 ms  1.138 ms  0.781 ms
 3   10.0.3.14   1.433 ms  1.766 ms  1.007 ms
 4   *10.0.3.30   0.894 ms (ICMP type:3, code:1, Destination host unreachable)  *
 5   *10.0.3.14   1.564 ms (ICMP type:3, code:1, Destination host unreachable)  *
 6   *10.0.3.30   1.790 ms (ICMP type:3, code:1, Destination host unreachable)  *
 7   *10.0.3.14   2.015 ms (ICMP type:3, code:1, Destination host unreachable)  *
 8   *10.0.3.30   1.450 ms (ICMP type:3, code:1, Destination host unreachable)  *
```