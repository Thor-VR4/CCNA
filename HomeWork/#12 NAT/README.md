# BGP filtering
Лабараторное задание №12 по работе с NAT.

## Задание
1. [Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001](#chapter-0)
2. [Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042](#chapter-1)
3. [Настроить статический NAT для R20](#chapter-2)
4. [Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления](#chapter-3)
5. [Настроить статический NAT(PAT) для офиса Чокурдах](#chapter-4)
6. [Настроить для IPv4 DHCP сервер в офисе Москва](#chapter-5)
7. [Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13](#chapter-6)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001
Пример настройки на R14:
```
interface Ethernet0/0
	ip nat inside
!
interface Ethernet0/1
	ip nat inside
!
interface Ethernet0/2
	ip nat outside
!
interface Ethernet0/3
	ip nat inside
!
ip nat inside source list 1 interface Ethernet0/2 overload
!
access-list 1 deny   10.0.4.20
access-list 1 deny   10.0.4.19
access-list 1 deny   10.0.3.42
access-list 1 deny   10.0.3.34
access-list 1 permit 10.0.0.0 0.0.255.255
```
Таблица трансляций на R15:
```
R15#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.0.3.37:34545   10.0.0.11:34545    10.2.4.23:34545    10.2.4.23:34545
icmp 10.0.3.37:34801   10.0.0.11:34801    10.2.4.23:34801    10.2.4.23:34801
icmp 10.0.3.37:35057   10.0.0.11:35057    10.2.4.23:35057    10.2.4.23:35057
icmp 10.0.3.37:35313   10.0.0.11:35313    10.2.4.23:35313    10.2.4.23:35313
icmp 10.0.3.37:35569   10.0.0.11:35569    10.2.4.23:35569    10.2.4.23:35569
--- 10.0.5.0           10.0.3.34          ---                ---
tcp 10.0.5.19:2030     10.0.4.19:23       ---                ---
--- 10.0.5.1           10.0.4.20          ---                ---
```


<a id="chapter-1"></a>
## Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042
Настройка на R18:
```
!
interface Ethernet0/0
	ip nat inside
!
interface Ethernet0/1
	ip nat inside
!
interface Ethernet0/2
	ip nat outside
!
interface Ethernet0/0
	ip nat outside
!
ip nat pool POOL2042 10.3.5.0 10.3.5.4 prefix-length 23
ip nat inside source list 1 pool POOL2042
!
access-list 1 deny   10.3.3.28 0.0.0.3
access-list 1 deny   10.3.3.32 0.0.0.3
access-list 1 permit 10.3.0.0 0.0.255.255
```
Таблица трансляций на R18:
```
R18#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.3.5.0:723      10.3.0.11:723      10.2.4.23:723      10.2.4.23:723
icmp 10.3.5.0:979      10.3.0.11:979      10.2.4.23:979      10.2.4.23:979
icmp 10.3.5.0:1235     10.3.0.11:1235     10.2.4.23:1235     10.2.4.23:1235
icmp 10.3.5.0:1491     10.3.0.11:1491     10.2.4.23:1491     10.2.4.23:1491
icmp 10.3.5.0:1747     10.3.0.11:1747     10.2.4.23:1747     10.2.4.23:1747
--- 10.3.5.0           10.3.0.11          ---                ---
```

<a id="chapter-2"></a>
## Настроить статический NAT для R20
Пример настройки на R14:
```
ip nat inside source static 10.0.3.34 10.0.5.0
ip nat inside source static 10.0.4.20 10.0.5.1
```

<a id="chapter-3"></a>
## Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления
Пример настройки на R14:
```
ip nat inside source static tcp 10.0.4.19 23 10.0.5.19 2030 extendable
```
Подключение к R19 c R23:
```
R23#telnet 10.0.5.19 23
Trying 10.0.5.19 ... 
% Destination unreachable; gateway or host down

R23#telnet 10.0.5.19 2030
Trying 10.0.5.19, 2030 ... Open
 Authorized Access Only! 

User Access Verification

Password: 
R19>
```

<a id="chapter-4"></a>
## Настроить статический NAT(PAT) для офиса Чокурдах
Настройка на R28:
```
interface Ethernet0/0
	ip nat outside
!
interface Ethernet0/1
	ip nat outside
!
interface Ethernet0/2.10
	ip nat inside
!
interface Ethernet0/2.20
	ip nat inside
!
interface Ethernet0/2.21
	ip nat inside
!
ip nat inside source route-map to_R25 interface Ethernet0/1 overload
ip nat inside source route-map to_R26 interface Ethernet0/0 overload
!
route-map to_R25 permit 10
 match ip address 3
 match interface Ethernet0/1
!
route-map to_R26 permit 10
 match ip address 3
 match interface Ethernet0/0
!
access-list 3 permit 10.4.0.0 0.0.255.255
```
Таблица трансляций на R28, при обоих рабочих линках:
```
R28#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.2.3.30:35575   10.4.0.30:35575    10.2.4.23:35575    10.2.4.23:35575
icmp 10.2.3.30:35831   10.4.0.30:35831    10.2.4.23:35831    10.2.4.23:35831
icmp 10.2.3.30:36087   10.4.0.30:36087    10.2.4.23:36087    10.2.4.23:36087
icmp 10.2.3.30:36343   10.4.0.30:36343    10.2.4.23:36343    10.2.4.23:36343
icmp 10.2.3.30:36599   10.4.0.30:36599    10.2.4.23:36599    10.2.4.23:36599
icmp 10.2.3.34:37367   10.4.1.31:37367    10.2.4.23:37367    10.2.4.23:37367
icmp 10.2.3.34:37623   10.4.1.31:37623    10.2.4.23:37623    10.2.4.23:37623
icmp 10.2.3.34:37879   10.4.1.31:37879    10.2.4.23:37879    10.2.4.23:37879
icmp 10.2.3.34:38135   10.4.1.31:38135    10.2.4.23:38135    10.2.4.23:38135
icmp 10.2.3.34:38391   10.4.1.31:38391    10.2.4.23:38391    10.2.4.23:38391
```
Таблица трансляций на R28, при одном рабочем линке:
```
R28#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.2.3.30:4344    10.4.0.30:4344     10.2.4.23:4344     10.2.4.23:4344
icmp 10.2.3.30:4600    10.4.0.30:4600     10.2.4.23:4600     10.2.4.23:4600
icmp 10.2.3.30:4856    10.4.0.30:4856     10.2.4.23:4856     10.2.4.23:4856
icmp 10.2.3.30:5112    10.4.0.30:5112     10.2.4.23:5112     10.2.4.23:5112
icmp 10.2.3.30:5368    10.4.0.30:5368     10.2.4.23:5368     10.2.4.23:5368
icmp 10.2.3.30:2296    10.4.1.31:2296     10.2.4.23:2296     10.2.4.23:2296
icmp 10.2.3.30:2552    10.4.1.31:2552     10.2.4.23:2552     10.2.4.23:2552
icmp 10.2.3.30:2808    10.4.1.31:2808     10.2.4.23:2808     10.2.4.23:2808
icmp 10.2.3.30:3064    10.4.1.31:3064     10.2.4.23:3064     10.2.4.23:3064
icmp 10.2.3.30:3320    10.4.1.31:3320     10.2.4.23:3320     10.2.4.23:3320
```
<a id="chapter-5"></a>
## Настроить для IPv4 DHCP сервер в офисе Москва
Пример настройки на SW4:
```
!
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.0.1.1 10.0.1.10
!
ip dhcp pool Vlan20
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1 
!
ip dhcp pool Vlan21
 network 10.0.1.0 255.255.255.0
 default-router 10.0.1.1 
!
```
Получение адреса на VPC1:
```
DDORA IP 10.3.1.11/24 GW 10.3.1.1
```

<a id="chapter-6"></a>
## Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13
Настройка сервера:
```
ntp master 2
```
Настройка клиентов:
```
ntp server 10.0.4.13
```
Добавление NTP сервера в DHCP:
```
!
ip dhcp pool Vlan20
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1 
 option 42 ip 10.0.4.13 
!
ip dhcp pool Vlan21
 network 10.0.1.0 255.255.255.0
 default-router 10.0.1.1 
 option 42 ip 10.0.4.13 
!
```