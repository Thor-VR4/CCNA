<a id="chapter-0"></a>
# VLAN
Лабараторное задание №1 по работе с виртуальными сетями.

## Задание
1. Создать сеть и настроить основные устройства
2. Создать Vlan-ы и настроить порты
3. Создать 802.1Q транк между свичами
4. Настроить маршрутизацию между Vlan на роутере
5. Проверить работоспособность маршрутизации.

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/Vlan.png "Стенд №1")

## Настройка компонентов
Изменения на устройствах сводятся к:
1. Настройке удаленного доступа и параметров авторизации. К примеру:
   ```
   service password-encryption
   enable secret *****
   !
   line con 0
    password *****
    login
   !
   line vty 0 4
    password *****
    login
   !
   banner motd $ Authorized Access Only! $
   ```
2. Настройке Vlan на магистральных и конечных портах. К примеру настройка vlan 3 и соответствующего SVI на коммутаторе S1:
    ```
    vlan 3
      name  Management
    !
    interface FastEthernet0/6
      switchport access vlan 3
      switchport mode access
    !
    interface FastEthernet0/1
      switchport trunk allowed vlan add 3
    !
    interface FastEthernet0/5
      switchport trunk allowed vlan add 3
    !
    interface Vlan3
       ip address 192.168.3.11 255.255.255.0
       no shutdown
    !
    ```
3. Настройке маршрутизации на роутере R1. К примеру:
   ```
   interface GigabitEthernet0/0/1.3
      description Management
      encapsulation dot1Q 3
   ip address 192.168.3.1 255.255.255.0
   !
   interface GigabitEthernet0/0/1.4
      description Operations
      encapsulation dot1Q 4
      ip address 192.168.4.1 255.255.255.0
   !
   ```
## Проверка работоспособности
Проверка проводилась с помощью команды ping на подключенных к сети устройствах. Были получены ответы от всех хостов, что свидетельствует о корректной настройке оборудования.  
Доступность default gateway с PC-A
```
C:\>ping 192.168.3.1

Pinging 192.168.3.1 with 32 bytes of data:

Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
Reply from 192.168.3.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.3.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
Доступность PC-B c PC-A
```
C:\>ping 192.168.4.3

Pinging 192.168.4.3 with 32 bytes of data:

Reply from 192.168.4.3: bytes=32 time=1ms TTL=127
Reply from 192.168.4.3: bytes=32 time=1ms TTL=127
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.4.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```
Доступность коммутатора S2 с PC-A
```
C:\>ping 192.168.3.12

Pinging 192.168.3.12 with 32 bytes of data:

Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time=1ms TTL=255

Ping statistics for 192.168.3.12:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```
Трассировка от PC-B к PC-A
```
Packet Tracer PC Command Line 1.0
C:\>tracert 192.168.3.3

Tracing route to 192.168.3.3 over a maximum of 30 hops: 

  1   4 ms      0 ms      0 ms      192.168.4.1
  2   0 ms      0 ms      0 ms      192.168.3.3

Trace complete.
```

## Итоговая конфигурация
1. [R1](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/R1.txt)
1. [S1](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/S1.txt)
1. [S2](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/S2.txt)
