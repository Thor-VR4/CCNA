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
R2 G0/0/1 | 2001:db8:acad:2::3/64, fe80::1

<a id="chapter-0"></a>
## Наблюдение за выборами корневого моста

В начале на коммутаторах были отключены все порты и переведены в режим транка. Затем на всех устройствах были включены интерфейсы F0/2,F0/4.
В результате работы протокола STP корневым мостом стал коммутатор S1.
```
S1#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0002.17DB.59D2
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p	
```
Это произошло из-за того, что приоритет BridgeID на всех устройствах оказался одинаковым, но MAC-адрес S1 оказался наименьшим.
Порты F0/4 и F0/2 перешли в статус назначенных.
Состояние коммутаторов S2 и S3:
```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0CC4.12D2
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```
```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.7025.6CE9
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```
На коммутаторе S2 порт F0/4 перешел в состяние альтернативного и был заблокирован, т.к. сам коммутатор имеет наибольший MAC-адрес, а стоимость корнегого пути через F0/2 ниже.

<a id="chapter-1"></a>
## Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

Изменим стоимость порта F0/2 коммутатора S2.
```
S2(config)# interface f0/2
S2(config-if)# spanning-tree vlan1 cost 18
```
В состоянии портов на S2 и S3 произошли именения:
```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        18
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0CC4.12D2
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 18        128.2    P2p
Fa0/4            Desg LSN 19        128.4    P2p
```
```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.7025.6CE9
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Altn BLK 19        128.2    P2p
```
Ранее заблокированнй порт F0/4 на коммутаторе S2 перешел в состояние назначенного, а порт F0/2 коммутатора S3 в состояние альтернативного.
Это произошло из-за того, что стоимость пути через S2 оказалась меньше чем стоимость пути через S3, в результате уменьшения стоимости порта F0/2 коммутатора S2.

<a id="chapter-2"></a>
## Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

Вернем стоимость порта F0/2 коммутатора S2 к изначальному значению. Включим порты f0/1,f0/3 на всех устройствах.
В результате на коммутаторах S2 и S3 изменился порт корневого моста.
```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0CC4.12D2
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/3            Altn BLK 19        128.3    P2p
```
```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.17DB.59D2
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.7025.6CE9
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
```
Это произошло в результате того, что стал доступен альтернативный путь через порт с меньшим номером.

## Заключение

Данная лабораторня работа показала процессы выбора корневого моста и определения состояния портов на устройствах входящих в STP. Так же было видно влияния таких параметров как BridgeID, стоимость и номер, на выбор роли порта.