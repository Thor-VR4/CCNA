# BGP filtering
Лабараторное задание №11 по работе с фильтрацией в BGP.

## Задание
1. [Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)](#chapter-0)
2. [Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)](#chapter-1)
3. [Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию](#chapter-2)
4. [Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург](#chapter-3)

## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)

Для фильтрации всех тразитных префиксов использован ip as-path access-list.  
Пример конфигурации на R14:
```
ip as-path access-list 1 permit ^$
!
router bgp 1001
 !
 address-family ipv4
  neighbor 10.0.3.46 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2A03:5A00:1F:3::22 filter-list 1 out
 exit-address-family
```
Список префиксов полученных от R14 на R22:
```
R22#show ip bgp neighbors 10.0.3.45 routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.0.0/16      10.0.3.45                0             0 1001 1001 i

Total number of prefixes 1 



R22#show ip bgp ipv6 uni neighbors 2A03:5A00:1F:3::14 routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2A03:5A00:1F::/56
                       2A03:5A00:1F:3::14
                                                0             0 1001 1001 i

Total number of prefixes 1 
```

<a id="chapter-1"></a>
## Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)

Для фильтрации всех тразитных префиксов использован prefix-list.  
Конфигурация на R18:
```
ip prefix-list 2042_out_v4 seq 5 permit 10.3.0.0/16 le 24
ipv6 prefix-list 2042_out_v6 seq 5 permit 2A03:5A00:1F:300::/56 le 64
!
router bgp 2042
 !
 address-family ipv4
  neighbor 10.3.3.30 prefix-list 2042_out_v4 out
  neighbor 10.3.3.34 prefix-list 2042_out_v4 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2A03:5A00:1F:203::26 prefix-list 2042_out_v6 out
  neighbor 2A03:5A00:1F:303::24 prefix-list 2042_out_v6 out
 exit-address-family
!
```
Список префиксов полученных от R18 на R26:
```
R26#show ip bgp neighbors 10.3.3.33 routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.3.0.0/16      10.3.3.33                0             0 2042 i

Total number of prefixes 1 


R26#show ip bgp ipv6 uni neighbors 2A03:5A00:1F:203::18 routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2A03:5A00:1F:300::/56
                       2A03:5A00:1F:203::18
                                                0             0 2042 i

Total number of prefixes 1 
```

<a id="chapter-2"></a>
## Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию

Маршрут по-умолчанию отдается через default-originate, фильтрация осуществляется с помощью prefix-list.
```
 !
 address-family ipv4
  neighbor 10.0.3.45 default-originate
  neighbor 10.0.3.45 prefix-list 1001_out_v4 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2A03:5A00:1F:3::14 default-originate
  neighbor 2A03:5A00:1F:3::14 prefix-list to_1001 out
 exit-address-family
!
ip prefix-list 1001_out_v4 seq 5 permit 0.0.0.0/0
!
ipv6 prefix-list to_1001 seq 5 permit ::/0
```
Префиксы получаемые R14 от R22:
```
R14#show ip bgp neighbors 10.0.3.46  routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *   0.0.0.0          10.0.3.46                              0 101 i

Total number of prefixes 1 


R14#show ip bgp ipv6 uni neighbors 2A03:5A00:1F:3::22  routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  ::/0             2A03:5A00:1F:3::22
                                                              0 101 i

Total number of prefixes 1 
```

<a id="chapter-3"></a>
## Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург

Маршрут по-умолчанию отдается через default-originate, фильтрация осуществляется с помощью as-path access-list.

```
ip as-path access-list 1 permit _2042$
 !
 address-family ipv4
  neighbor 10.0.3.37 default-originate
  neighbor 10.0.3.37 filter-list 1 out
 exit-address-family
 !
 address-family ipv6
  neighbor 2A03:5A00:1F:3::15 default-originate
  neighbor 2A03:5A00:1F:3::15 filter-list 1 out
 exit-address-family
!
```
Префиксы получаемые R15 от R21:
```
R15#show ip bgp neighbors 10.0.3.38 routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          10.0.3.38                     200      0 301 i
 *>  10.3.0.0/16      10.0.3.38                     200      0 301 520 2042 i

Total number of prefixes 2 


R15#show ip bgp ipv6 uni neighbors 2A03:5A00:1F:3::21  routes 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  ::/0             2A03:5A00:1F:3::21
                                                              0 301 i
 *>  2A03:5A00:1F:300::/56
                       2A03:5A00:1F:3::21
                                                              0 301 520 2042 i

Total number of prefixes 2 
```