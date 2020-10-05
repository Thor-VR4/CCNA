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
Изменения на маршрутизаторах сводятся к:
1. Настройке удаленного доступа и параметров авторизации.
   К примеру:
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
    !
    ```

## Итоговая конфигурация
1. [R1](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/R1.txt)
1. [S1](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/S1.txt)
1. [S2](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%231%20Vlan/config/S2.txt)
