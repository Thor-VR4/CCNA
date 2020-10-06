# STP
Лабараторное задание №2 по работе с STP.

## Задание
1. Создать сеть и настроить основные устройства
2. [Наблюдение за ввыборами корневого моста](#chapter-0)
3. [Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов](#chapter-1)
4. [Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов](#chapter-2)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%232%20STP/STP.png "Стенд №2")

## Таблица адресации

Устройство | IP-адрес | MAC-адрес
--- | --- | ---
S1 | 192.168.1.1 | 0002.17DB.59D2
S2 | 192.168.1.2 | 0090.0CC4.12D2
S3 | 192.168.1.3 | 0060.7025.6CE9

<a id="chapter-0"></a>
## Наблюдение за ввыборами корневого моста

В начале на коммутаторах были отключены все порты и переведены вв режим транка. Затем на всех устройствах были включены интерфейсы F0/2,F0/4.
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
Это произошло из-за того, что его BridgeID на всех устройствах оказался одинаковым, но MAC-адрес S1 оказался наименьшим.
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
На коммутаторе S3 порт F0/4 перешел в состяние альтернативного и был заблокирован, т.к. сам коммутатор имеет наибольший MAC-адрес, а стоимость корнегого пути через F0/2 ниже.