# DHCP
Лабараторное задание №3 по работе с DHCP.

## Задание
1. Создать сеть и настроить основные устройства
2. [Сконфигурировать DHCP_v4 сервер](#chapter-0)
3. [Сконфигурировать DHCP_v4 relay](#chapter-1)
4. [Наблюдение за процессом SLAAC при получении IPv6 адреса](#chapter-2)
5. [Сконфигурировать Stateless DHCP_v6 сервер](#chapter-3)
6. [Сконфигурировать Stateful DHCP_v6 сервер](#chapter-4)
7. [Сконфигурировать DHCP_v6 relay](#chapter-5)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%233%20DHCP/DHCP.png "Стенд №3")

## Адресное пространство

Название | Сеть 
--- | ---
Subnet A | 192.168.1.0/26
Subnet B | 192.168.1.64/27
Subnet C | 192.168.1.96/28

## Таблица адресации IPv4

Интерфейс | IP-адрес
--- | ---
R1 G0/0/0 | 10.0.0.1/30
R1 G0/0/1.100 | 192.168.1.1/26
R1 G0/0/1.200 | 192.168.1.65/27
S1 VLAN 1 | 192.168.1.66/27
S1 VLAN 200 | 192.168.1.2/26
R2 G0/0/0 | 10.0.0.2/30
R2 G0/0/1 | 192.168.1.97/28

## Таблица адресации IPv6

Интерфейс | IP-адрес
--- | ---
R1 G0/0/0 | 2001:db8:acad:2::1/64, fe80::1
R1 G0/0/1 | 2001:db8:acad:1::1/64, fe80::1
R2 G0/0/0 | 2001:db8:acad:2::2/64, fe80::2
R2 G0/0/1 | 2001:db8:acad:3::1/64, fe80::1

## Таблица VLAN для актуальных интерфйсов при работе с IPv4

Устройство | Интерфейс | VLAN
--- | --- | ---
S1 | F0/5 | trunk 100,200,1000 native 1000
S1 | F0/6 | access 100
S2 | F0/18 | access 1
S2 | F0/5 | access 1

## Таблица VLAN для актуальных интерфйсов при работе с IPv6

Устройство | Интерфейс | VLAN
--- | --- | ---
S1 | F0/5 | access 1
S1 | F0/6 | access 1
S2 | F0/18 | access 1
S2 | F0/5 | access 1

<a id="chapter-0"></a>
## Сконфигурировать DHCP_v4 сервер

Создадим на R1 два DHCP пула для сетей Subnet A и Subnet C
Конфигурация пула A:
```
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp pool A
 network 192.168.1.0 255.255.255.192
 default-router 192.168.1.1
 domain-name ccna-lab.com
 lease 2 12 30	
```
Конфигурация пула С аналогична.
PC-A смог получить IPv4 адрес от R1
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: ccna-lab.com
   Link-local IPv6 Address.........: FE80::201:42FF:FEB0:6616
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: FE80::1
                                     192.168.1.1
```
Информация о выданном адресе на R1
```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0001.42B0.6616          Oct 15 2020 11:56 PM    Automatic
```
Проверка доступности R1 с PC-A
```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

```

<a id="chapter-1"></a>
## Сконфигурировать DHCP_v4 relay

Настроим DHCP_v4 relay на интерейсе G0/0/1 маршрутизотора R2 для передачи DHCP запросов на R1
```
conf t
	interface GigabitEthernet0/0/1
	 ip helper-address 10.0.0.1
```
PC-B смог получить IP-адрес
```
C:\>ipconfig /renew

   IP Address......................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: 192.168.1.97
   DNS Server......................: 0.0.0.0
```
Информация о выданном адресе на R1
```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0001.42B0.6616          Oct 15 2020 11:56 PM    Automatic
192.168.1.102    0050.0F44.7214          Oct 15 2020 11:59 PM    Automatic
```
Проверка доступности R2 с PC-A
```
C:\>ping 192.168.1.97

Pinging 192.168.1.97 with 32 bytes of data:

Reply from 192.168.1.97: bytes=32 time=1ms TTL=255
Reply from 192.168.1.97: bytes=32 time<1ms TTL=255
Reply from 192.168.1.97: bytes=32 time=1ms TTL=255
Reply from 192.168.1.97: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.97:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

<a id="chapter-2"></a>
## Наблюдение за процессом SLAAC при получении IPv6 адреса

Собираем стенд в соответствии со спецификацией к лабараторной работе с IPv6
На интерфейсе R1 G0/0/1 устанавливаем IP-адрес в соответствии с таблицей
```
R1#show ipv6 interface brief 
GigabitEthernet0/0/0       [up/up]
    FE80::1
    2001:DB8:ACAD:2::1
GigabitEthernet0/0/1       [up/up]
    FE80::1
    2001:DB8:ACAD:1::1
```
PC-A получил IP-адрес методом SLAAC
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::201:42FF:FEB0:6616
   IPv6 Address....................: 2001:DB8:ACAD:1:201:42FF:FEB0:6616
   Autoconfiguration IPv4 Address..: 169.254.102.22
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```
Необходимые данные были получены от R1 в сообщении RA
Проверим доступность R1
```
C:\>ping 2001:DB8:ACAD:1::1

Pinging 2001:DB8:ACAD:1::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=255

Ping statistics for 2001:DB8:ACAD:1::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

<a id="chapter-3"></a>
## Сконфигурировать Stateless DHCP_v6 сервер

Создадим на R1 пул R1-STATELESS
```
ipv6 dhcp pool R1-STATELESS
 dns-server 2001:DB8:ACAD::254
 domain-name STATELESS.com
```
Настроим интерфейс для добавления O флага в RA сообщения
```
interface GigabitEthernet0/0/1
 ipv6 nd other-config-flag
 ipv6 dhcp server R1-STATELESS
```
Теперь PC-A получил информацию о DNS сервере, использую Stateless DHCP_v6
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0001.42B0.6616
   Link-local IPv6 Address.........: FE80::201:42FF:FEB0:6616
   IPv6 Address....................: 2001:DB8:ACAD:1:201:42FF:FEB0:6616
   Autoconfiguration IP Address....: 169.254.102.22
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 571485163
   DHCPv6 Client DUID..............: 00-01-00-01-93-C9-AD-EC-00-01-42-B0-66-16
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```

<a id="chapter-4"></a>
## Сконфигурировать Stateful DHCP_v6 сервер

Добавим пул R2-STATEFUL на R1
```
ipv6 dhcp pool R2-STATEFUL
 address prefix 2001:db8:acad:3:aaa::/80
 dns-server 2001:DB8:ACAD::254
 domain-name STATEFUL.com
```
В пуле для выдачи назначена сеть 2001:db8:acad:3:aaa::/80

<a id="chapter-5"></a>
## Сконфигурировать DHCP_v6 relay

Создадим DHCP_v6 relay на R2
```
interface GigabitEthernet0/0/1
 ipv6 nd managed-config-flag
 ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
```
PC-B получил новый адрес из сети 2001:db8:acad:3:aaa::/80
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATEFUL.com 
   Link-local IPv6 Address.........: FE80::250:FFF:FE44:7214
   IPv6 Address....................: 2001:DB8:ACAD:3:AAA:9E23:9E23:9E23
   Autoconfiguration IPv4 Address..: 169.254.114.20
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0

```
Проверим доступность R1 с PC-B
```
C:\>ping 2001:DB8:ACAD:1::1

Pinging 2001:DB8:ACAD:1::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:1::1: bytes=32 time=1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:1::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

## Заключение

В лабараторной работе были показаны различные способы выдачи IP-адресов, как непосредственно с сервера DHCP, так и при помощи релея. 