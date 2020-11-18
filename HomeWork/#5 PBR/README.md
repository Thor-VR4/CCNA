# DHCP
Лабараторное задание №5 по работе с PBR.

## Задание
1. [Настроить политику маршрутизации для сетей офиса Чокурдах и распределить трафик между двумя линками с провайдером](#chapter-0)
2. [Настроить отслеживание линка через технологию IP SLA](#chapter-1)
3. [Настройть для офиса Лабытнанги маршрут по-умолчанию](#chapter-2)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настроить политику маршрутизации для сетей офиса Чокурдах и распределить трафик между двумя линками с провайдером

В оффисе Чокурдах присутствуют сети 10.4.0.0/16 и 2A03:5A00:1F:4/56. Все они подключены напрямую к R28
v4:
```
C        10.4.0.0/24 is directly connected, Ethernet0/2.20
L        10.4.0.1/32 is directly connected, Ethernet0/2.20
C        10.4.1.0/24 is directly connected, Ethernet0/2.21
L        10.4.1.1/32 is directly connected, Ethernet0/2.21
C        10.4.2.0/24 is directly connected, Ethernet0/2.10
L        10.4.2.1/32 is directly connected, Ethernet0/2.10
C        10.4.4.28/32 is directly connected, Loopback0
```
v6:
```
C   2A03:5A00:1F:400::/64 [0/0]     via Ethernet0/2.20, directly connected
L   2A03:5A00:1F:400::1/128 [0/0]     via Ethernet0/2.20, receive
C   2A03:5A00:1F:401::/64 [0/0]     via Ethernet0/2.21, directly connected
L   2A03:5A00:1F:401::1/128 [0/0]     via Ethernet0/2.21, receive
C   2A03:5A00:1F:402::/64 [0/0]     via Ethernet0/2.10, directly connected
L   2A03:5A00:1F:402::28/128 [0/0]     via Ethernet0/2.10, receive
LC  2A03:5A00:1F:404::28/128 [0/0]     via Loopback0, receive
```
Для распределения трафика между линками с провайдерами, используються маршруты по умолчанию с одинаковой метрикой:
```
S*    0.0.0.0/0 [10/0] via 10.2.3.33
                [10/0] via 10.2.3.29
S   ::/0 [10/0]
     via FE80::25, Ethernet0/1
     via FE80::26, Ethernet0/0
``` 
Для клиентских сетей создана политика, указывающая next-hop в зависимости от сети источника.
К примеру:
```
access-list 1 permit 10.4.0.0 0.0.0.255
!
route-map Clients permit 10
 match ip address 1
 set ip next-hop 10.2.3.29
```

<a id="chapter-1"></a>
## Настроить отслеживание линка через технологию IP SLA
На маршрутизаторе R28 настроено отслеживание доступности шлюзов по умолчанию по средствам протокола ICMP и технологии IP SLA.
К примеру, отслеживание v4 шлюза на маршрутизаторе R25:
```
ip sla 1
 icmp-echo 10.2.3.29 source-ip 10.2.3.30
 frequency 5
 threshold 2000
 timeout 2000
ip sla schedule 1 life forever start-time now
```
На основе данных IP SLA реализовано включение или отключение маршрутов через track.
Для маршрутов v4:
```
ip route 0.0.0.0 0.0.0.0 10.2.3.29 10 track 1
ip route 0.0.0.0 0.0.0.0 10.2.3.33 10 track 2
``` 
Для маршрутов v6:
```
event manager applet track3_up
 event track 3 state up
 action 1 cli command "en"
 action 2 cli command "conf t"
 action 3 cli command "ipv6 route ::/0 Ethernet0/1 FE80::25 10"
event manager applet track3_down
 event track 3 state down
 action 1 cli command "en"
 action 2 cli command "conf t"
 action 3 cli command "no ipv6 route ::/0 Ethernet0/1 FE80::25 10"
```

<a id="chapter-2"></a>
## Настройть для офиса Лабытнанги маршрут по-умолчанию

На R27 настроен маршрут по умолчанию:
```
ip route 0.0.0.0 0.0.0.0 10.2.3.25
ipv6 route ::/0 ethernet 0/1 FE80::25
```