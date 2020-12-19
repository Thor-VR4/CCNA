# iBGP
Лабараторное задание №10 по работе с iBGP.

## Задание
1. [Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15](#chapter-0)
2. [Настроить iBGP в провайдере Триада](#chapter-1)
3. [Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас](#chapter-2)
4. [Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно](#chapter-3)
5. [Настроить IP связность всех сетей](#chapter-4)
6. [Проверка работоспособности](#chapter-5)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настройка iBGP в офисе Москва между маршрутизаторами R14 и R15
Настройка производилась, используя адреса lo0 интерфейсов и технологию multihop. 
Пример конфигурации R14:
```
router bgp 1001
 neighbor 10.0.4.15 remote-as 1001
 neighbor 10.0.4.15 update-source Loopback0
 neighbor 2A03:5A00:1F:4::15 remote-as 1001
!
address-family ipv4
 neighbor 10.0.4.15 activate
 neighbor 10.0.4.15 next-hop-self
!
address-family ipv6
 neighbor 2A03:5A00:1F:4::15 activate
 neighbor 2A03:5A00:1F:4::15 next-hop-self
```

<a id="chapter-1"></a>
## Настройка iBGP в провайдере Триада

Конфигурация аналогична iBGP между маршрутизаторами R14 и R15 в Москве.

<a id="chapter-2"></a>
## Настройка офиса Москва так, чтобы приоритетным провайдером стал Ламас.

Во-первых увеличим приоритет для маршрутов получаемых от R21. Для этого на R15 установим local preference равным 200.
```
router bgp 1001
!
 address-family ipv4
  neighbor 10.0.3.38 route-map 301_IN in
!
 address-family ipv6
  neighbor 10.0.3.38 route-map 301_IN in
!
route-map 301_IN permit 10
 set local-preference 200
```
Теперь на R14 маршруты от R15 более приоритетны чем от R22
```
     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.0.0.0/16      10.0.4.15                0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 10.1.0.0/16      10.0.4.15                0    200      0 301 i
 *                    10.0.3.46                0             0 101 i
 *>i 10.2.0.0/16      10.0.4.15                0    200      0 301 520 i
 *                    10.0.3.46                              0 101 520 i
 *>i 10.3.0.0/16      10.0.4.15                0    200      0 301 520 2042 i
 *                    10.0.3.46                              0 101 520 2042 i
 *>i 10.4.0.0/16      10.0.4.15                0    200      0 301 520 i
 *                    10.0.3.46                              0 101 520 i
```
Во-вторых уменьшим приоритет для маршрутов анонсируемых R14
```
router bgp 1001
!
 address-family ipv4
  neighbor 10.0.3.46 route-map 101_out_v4 out
!
 address-family ipv6  
  neighbor 10.0.3.46 route-map 101_out out
!  
route-map 101_out permit 10
 match ipv6 address prefix-list own_out
 set as-path prepend 1001
 set ipv6 next-hop 2A03:5A00:1F:3::14
!
route-map 101_out_v4 permit 10
 set as-path prepend 1001  
```
Маршрутная таблица до 10.0.0.0/16 на R23
```
 *>i 10.0.0.0/16      10.2.4.24                0    100      0 301 1001 i
 *                    10.2.3.14                              0 101 1001 1001 i
```
В-третьих уменьшим приоритет маршрута по умолчанию, анонсируемого R14 по OSPF.
```
router ospfv3 1001
 !
 address-family ipv4 unicast
  default-information originate always metric 2
 !
 address-family ipv6 unicast
  default-information originate always metric 2
!
```
БД OSPF на R12
```
          OSPFv3 1001 address-family ipv6 (router-id 0.0.0.12)

                Type-5 AS External Link States

  Routing Bit Set on this LSA
  LS age: 13
  LS Type: AS External Link
  Link State ID: 0
  Advertising Router: 0.0.0.14
  LS Seq Number: 8000005B
  Checksum: 0xB507
  Length: 32
  Prefix Address: ::
  Prefix Length: 0, Options: None
  Metric Type: 2 (Larger than any link state path)
  Metric: 2 
  External Route Tag: 1001

  Routing Bit Set on this LSA
  LS age: 696
  LS Type: AS External Link
  Link State ID: 0
  Advertising Router: 0.0.0.15
  LS Seq Number: 8000005A
  Checksum: 0xAB12
  Length: 32
  Prefix Address: ::
  Prefix Length: 0, Options: None
  Metric Type: 2 (Larger than any link state path)
  Metric: 1 
  External Route Tag: 1001
```

<a id="chapter-3"></a>
## Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно

Реализовано с использованием BGP multipath на R18
```
     Network          Next Hop            Metric LocPrf Weight Path
 *m  10.0.0.0/16      10.3.3.34                              0 520 301 1001 i
 *>                   10.3.3.30                              0 520 301 1001 i
 *m  10.1.0.0/16      10.3.3.30                              0 520 301 i
 *>                   10.3.3.34                              0 520 301 i
 *m  10.2.0.0/16      10.3.3.30                              0 520 i
 *>                   10.3.3.34                              0 520 i
 *>  10.3.0.0/16      0.0.0.0                  0         32768 i
 *m  10.4.0.0/16      10.3.3.34                0             0 520 i
 *>                   10.3.3.30                              0 520 i
```

<a id="chapter-4"></a>
## Настроить IP связность всех сетей

Общий префикс каждой сети 10.x.0.0/16 анонсируется по BGP соответствующими маршрутизаторами. Исключение составляет 10.4.0.0/16, закреплённый за Чокурдах.
Этот маршрут анонсируется R25 и R26 из AS520. 

<a id="chapter-5"></a>
## Проверка работоспособности

Ping R27 c VPC1
```
VPCS> ping  10.2.4.27

84 bytes from 10.2.4.27 icmp_seq=1 ttl=248 time=28.675 ms
84 bytes from 10.2.4.27 icmp_seq=2 ttl=248 time=24.663 ms
84 bytes from 10.2.4.27 icmp_seq=3 ttl=248 time=3.834 ms
84 bytes from 10.2.4.27 icmp_seq=4 ttl=248 time=5.218 ms
84 bytes from 10.2.4.27 icmp_seq=5 ttl=248 time=5.011 ms
```
Трассировка VPC c VPC31
```
trace to 2a03:5a00:1f:301:2050:79ff:fe66:680b, 64 hops max
 1 2a03:5a00:1f:401::1   1.386 ms  0.662 ms  0.558 ms
 2 2a03:5a00:1f:204:26::1   1.507 ms  1.649 ms  1.080 ms
 3 2a03:5a00:1f:203::18   2.694 ms  1.381 ms  1.993 ms
 4 2a03:5a00:1f:304::16   2.385 ms  2.733 ms  2.366 ms
 5 2a03:5a00:1f:302::10   3.705 ms  5.215 ms  2.883 ms
 6 2a03:5a00:1f:301:2050:79ff:fe66:680b   2.635 ms  2.623 ms  2.224 ms
```
