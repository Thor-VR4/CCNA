# IP
Лабараторное задание №9 по работе с EBGP.

## Задание
1. [Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас](#chapter-0)
2. [Настроить eBGP между провайдерами Киторн и Ламас, и Ламас и Триада](#chapter-1)
3. [Настроить eBGP между офисом С.-Петербург и провайдером Триада](#chapter-2)
4. [Организовать IP доступность между офисами Москва и С.-Петербург](#chapter-3)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настройка eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас

Настраиваться будут R14, R15 из офиса Москва c AS1001, R22 из AS101 и R21 из AS301.
Взатмодействие происходит через IPv4 пириг по обоим address family. AS1001 анонсирует 10.0.0.0/16 и  2A03:5A00:1F::/56.

Пример настройки на R14:
```
!
router bgp 1001
 bgp router-id 10.0.4.14
 bgp log-neighbor-changes
 neighbor 10.0.3.46 remote-as 101
 neighbor 10.0.3.46 description R22
 !
 address-family ipv4
  network 10.0.0.0 mask 255.255.0.0
  neighbor 10.0.3.46 activate
 exit-address-family
 !
 address-family ipv6
  network 2A03:5A00:1F::/56
  neighbor 10.0.3.46 activate
  neighbor 10.0.3.46 route-map 101_out out
 exit-address-family
!
```
Для передачи корректного next-hop в IPv6 используется route-map.
Пример route-map R21:
```
!
ipv6 prefix-list to_1001 seq 5 permit ::/0 le 64
route-map 1001_out permit 5
 match ipv6 address prefix-list to_1001
 set ipv6 next-hop 2A03:5A00:1F:3::21
!
```
Для прохождения проверок на валидность анонсов на R14 и R15 добавлены маршруты:
```
ip route 10.0.0.0 255.255.0.0 Null0
ipv6 route 2A03:5A00:1F::/56 Null0
```

<a id="chapter-1"></a>
## Настройка eBGP между провайдерами Киторн и Ламас, и Ламас и Триада

BGP настраиваено между R22 AS101 и R21 AS301, и R21 AS301 и R24 AS520
Для взаимодействия в каждой address family, настроено соответстввующее соседство.
Пример настройки R21
```
!
router bgp 301
 bgp router-id 10.1.4.21
 bgp log-neighbor-changes
 neighbor 10.0.3.37 remote-as 1001
 neighbor 10.0.3.37 description R15
 neighbor 10.1.3.2 remote-as 101
 neighbor 10.1.3.2 description R22
 neighbor 10.2.3.5 remote-as 520
 neighbor 10.2.3.5 description R24
 neighbor 2A03:5A00:1F:103::1 remote-as 101
 neighbor 2A03:5A00:1F:103::1 description R22
 neighbor 2A03:5A00:1F:103::3 remote-as 520
 neighbor 2A03:5A00:1F:103::3 description R24
 !
 address-family ipv4
  neighbor 10.0.3.37 activate
  neighbor 10.1.3.2 activate
  neighbor 10.2.3.5 activate
  no neighbor 2A03:5A00:1F:103::1 activate
  no neighbor 2A03:5A00:1F:103::3 activate
 exit-address-family
 !
 address-family ipv6
  neighbor 10.0.3.37 activate
  neighbor 10.0.3.37 route-map 1001_out out
  neighbor 2A03:5A00:1F:103::1 activate
  neighbor 2A03:5A00:1F:103::3 activate
 exit-address-family
!
```

<a id="chapter-2"></a>
## Настройка eBGP между офисом С.-Петербург и провайдером Триада

Настроены взаимодействие R18 AS2042 c R24 и R26 из AS520.
Для взаимодействия в каждой address family, настроено соответстввующее соседство.
R18 анонсирует 10.3.0.0/16 и 2A03:5A00:1F:300::/56

<a id="chapter-3"></a>
## Организция IP доступности между офисами Москва и С.-Петербург

Для организации IP связанности между офисами, в С.-Петербург на маршрутизатор R18 добавлен анонс маршрута по умолчанию через EIGRP на роутеры R17 и R16.
```
  !
  af-interface Ethernet0/1
   summary-address 0.0.0.0 0.0.0.0
   no shutdown
  exit-af-interface
  !
  af-interface Ethernet0/0
   summary-address 0.0.0.0 0.0.0.0
   no shutdown
  exit-af-interface
  !
```
Аналогично для IPv6.
Проверка доступности адресов С.-Петербурга из Москвы посредствам ping c VPC1 на VPC8:
```
VPCS> ping 10.3.0.11

84 bytes from 10.3.0.11 icmp_seq=1 ttl=56 time=7.160 ms
84 bytes from 10.3.0.11 icmp_seq=2 ttl=56 time=3.487 ms
84 bytes from 10.3.0.11 icmp_seq=3 ttl=56 time=3.194 ms
84 bytes from 10.3.0.11 icmp_seq=4 ttl=56 time=13.669 ms
84 bytes from 10.3.0.11 icmp_seq=5 ttl=56 time=3.867 ms
```
Трассировка с тех же устройств:
```
VPCS> trace 2a03:5a00:1f:300:2050:79ff:fe66:6808

trace to 2a03:5a00:1f:300:2050:79ff:fe66:6808, 64 hops max
 1 2a03:5a00:1f::5   1.089 ms  0.738 ms  0.502 ms
 2 2a03:5a00:1f:4::12   1.459 ms  1.330 ms  1.056 ms
 3 2a03:5a00:1f:3::14   1.820 ms  1.348 ms  1.544 ms
 4 2a03:5a00:1f:3::22   19.391 ms  1.650 ms  1.114 ms
 5 2a03:5a00:1f:103::   37.722 ms  19.977 ms  1.821 ms
 6 2a03:5a00:1f:103::3   19.641 ms  3.011 ms  1.842 ms
 7 2a03:5a00:1f:303::18   11.430 ms  2.107 ms  1.385 ms
 8 2a03:5a00:1f:304::17   20.508 ms  20.472 ms  2.150 ms
 9 2a03:5a00:1f:302::9   36.247 ms  2.920 ms  2.374 ms
10 2a03:5a00:1f:300:2050:79ff:fe66:6808   12.257 ms  3.168 ms  3.052 ms
```
Маршрутная информация на R21:
```
R21#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
B        10.0.0.0/16 [20/0] via 10.0.3.37, 23:10:38
B        10.3.0.0/16 [20/0] via 10.2.3.5, 21:36:58

R21#show ipv6 route bgp
IPv6 Routing Table - default - 10 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
B   2A03:5A00:1F::/56 [20/0]
     via FE80::15, Ethernet0/0
B   2A03:5A00:1F:300::/56 [20/0]
     via FE80::24, Ethernet0/2
```
