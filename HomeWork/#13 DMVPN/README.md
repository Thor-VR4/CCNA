# BGP filtering
Лабараторное задание №13 по работе с DMVPN.

## Задание
1. [Настроите GRE между офисами Москва и С.-Петербург](#chapter-0)
2. [Настроите DMVMN между Москва и Чокурдах, Лабытнанги](#chapter-1)


## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настроите GRE между офисами Москва и С.-Петербург
Туннель настроен между R18 и R14/R15.
Настройка на R18:
```
!
interface Tunnel14
 ip address 172.16.0.0 255.255.255.254
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source Loopback0
 tunnel destination 10.0.3.45
!
interface Tunnel15
 ip address 172.16.0.3 255.255.255.254
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source Loopback0
 tunnel destination 10.0.3.37
!
```
Проверка работоспособности на R15:
```
R15#ping 172.16.0.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

<a id="chapter-1"></a>
## Настроите DMVMN между Москва и Чокурдах, Лабытнанги
Настроен DMVPN phase 2 с BGP. HQ - R15 в Москве.
R15:
```
interface Tunnel1001
 ip address 172.16.15.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 1001
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 10.0.4.15
 bgp log-neighbor-changes
 bgp listen range 172.16.15.0/24 peer-group DMVPN
 neighbor DMVPN peer-group
 neighbor DMVPN remote-as 1001
 !
 address-family ipv4
  network 10.0.0.0 mask 255.255.0.0
  neighbor DMVPN activate
  neighbor DMVPN route-reflector-client
  neighbor DMVPN filter-list 1 out
 exit-address-family
!
ip as-path access-list 1 permit ^$
!
ip route 10.4.4.28 255.255.255.255 10.0.3.38
!
```
R28:
```
!
interface Tunnel1001
 ip address 172.16.15.28 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 10.0.3.37
 ip nhrp map 172.16.15.1 10.0.3.37
 ip nhrp network-id 1001
 ip nhrp nhs 172.16.15.1
 ip tcp adjust-mss 1360
 tunnel source Loopback0
 tunnel mode gre multipoint
!
router bgp 1001
 bgp router-id 172.16.15.28
 bgp log-neighbor-changes
 network 10.4.0.0 mask 255.255.0.0
 neighbor 172.16.15.1 remote-as 1001
!
ip route 10.0.3.37 255.255.255.255 10.2.3.29 10 track 1
ip route 10.0.3.37 255.255.255.255 10.2.3.33 10 track 2
ip route 10.4.0.0 255.255.0.0 Null0
```
Таблица маршрутизации на R28:
```
R28#show ip route bgp

Gateway of last resort is 10.2.3.33 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 14 subnets, 4 masks
B        10.0.0.0/16 [200/0] via 172.16.15.1, 00:30:33
```
Туннели на R28:
```
R28#show ip nhrp 
172.16.15.1/32 via 172.16.15.1
   Tunnel1001 created 21:53:58, never expire 
   Type: static, Flags: used 
   NBMA address: 10.0.3.37 
172.16.15.27/32 via 172.16.15.27
   Tunnel1001 created 00:02:30, expire 01:57:29
   Type: dynamic, Flags: router used 
   NBMA address: 10.2.3.26 
```
Трассировка с VPC30 до VPC7:
```
VPCS> trace 10.0.1.11 -P 1
trace to 10.0.1.11, 8 hops max (ICMP), press Ctrl+C to stop
 1   10.4.0.1   1.640 ms  1.031 ms  0.610 ms
 2   172.16.15.1   3.778 ms  1.976 ms  1.885 ms
 3   10.0.3.29   2.351 ms  2.091 ms  1.805 ms
 4   10.0.3.6   2.636 ms  5.979 ms  2.146 ms
 5   10.0.1.11   4.620 ms  4.199 ms  2.230 ms
```