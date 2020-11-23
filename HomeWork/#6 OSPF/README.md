# OSPF
Лабараторное задание №6 по работе с OSPF.

## Задание
Настроить OSPF офисе Москва, hазделить сеть на зоны и настроить фильтрацию между зонами.
1. [Создание зоны 0](#chapter-0)
2. [Создание зоны 10](#chapter-1)
3. [Создание зоны 101](#chapter-2)
4. [Создание зоны 102](#chapter-2)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%236%20OSPF/OSPF.png "Стенд №6")

<a id="chapter-0"></a>
## Создание зоны 0
Участники зоны 0


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