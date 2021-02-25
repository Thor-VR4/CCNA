# IPSec
Лабараторное задание №14 по работе с IPSec.

## Задание
1. [Настроить работу тунелей поверх IPSec](#chapter-0)


## Схема сети
![alt-текст](https://github.com/Thor-VR4/CCNA/blob/master/HomeWork/%234%20IP/IP.png "Стенд №4")

<a id="chapter-0"></a>
## Настроить работу тунелей поверх IPSec

На R21 настроен центр сертификации:
```
ip domain name some_ca.ru
ip http server
crypto pki server CA
no shut
```
Пример получения сертификата на R15:
```
crypto key generate rsa label VPN modulus 2048
crypto pki trustpoint VPN
enrollment url http://10.1.4.21
subject-name CN=R15,OU=VPN,O=St.P,C=RU
rsakeypair VPN
revocation-check none
```
Полученный на R15 сертификат:
```
R15#show crypto pki certificates 
Certificate
  Status: Available
  Certificate Serial Number (hex): 04
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R15
    hostname=R15
    cn=R15
    ou=VPN
    o=MSK
    c=RU
  Validity Date: 
    start date: 12:24:48 EET Feb 25 2021
    end   date: 12:24:48 EET Feb 25 2022
  Associated Trustpoints: VPN 
  Storage: nvram:CA#4.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA
  Subject: 
    cn=CA
  Validity Date: 
    start date: 11:54:40 EET Feb 25 2021
    end   date: 11:54:40 EET Feb 25 2024
  Associated Trustpoints: VPN 
  Storage: nvram:CA#1CA.cer
```
Пример настройки GRE поверх IPsec на R18:
```
crypto isakmp policy 10
authentication rsa-sig 
crypto ipsec transform-set SET esp-aes esp-sha-hmac
crypto ipsec profile GRE
set transform-set SET
interface Tunnel15
tunnel protection ipsec profile GRE
```
Проверка работоспособности туннеля:
```
R18#ping 172.16.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/7 ms

R18#show crypto isakmp peers 
Peer: 10.0.3.37 Port: 500 Local: 10.3.4.18
 Phase1 id: R15
 
R18#show crypto session 
Crypto session current status

Interface: Tunnel15
Session status: UP-ACTIVE     
Peer: 10.0.3.37 port 500 
  IKEv1 SA: local 10.3.4.18/500 remote 10.0.3.37/500 Active 
  IKEv1 SA: local 10.3.4.18/500 remote 10.0.3.37/500 Active 
  IPSEC FLOW: permit 47 host 10.3.4.18 host 10.0.3.37 
        Active SAs: 2, origin: crypto map
```

Аналогичная настройка для DMVPN. Состояние туннелей на R27:
```
R27#show crypto session detail 
Crypto session current status

Code: C - IKE Configuration mode, D - Dead Peer Detection     
K - Keepalives, N - NAT-traversal, T - cTCP encapsulation     
X - IKE Extended Authentication, F - IKE Fragmentation

Interface: Tunnel1001
Uptime: 00:00:44
Session status: UP-ACTIVE     
Peer: 10.4.4.28 port 500 fvrf: (none) ivrf: (none)
      Phase1_id: R28
      Desc: (none)
  IKEv1 SA: local 10.2.3.26/500 remote 10.4.4.28/500 Active 
          Capabilities:(none) connid:1003 lifetime:23:59:15
  IPSEC FLOW: permit 47 host 10.2.3.26 host 10.4.4.28 
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 4 drop 0 life (KB/Sec) 4244771/3555
        Outbound: #pkts enc'ed 3 drop 0 life (KB/Sec) 4244771/3555

Interface: Tunnel1001
Uptime: 04:04:43
Session status: UP-ACTIVE     
Peer: 10.0.3.37 port 500 fvrf: (none) ivrf: (none)
      Phase1_id: R15
      Desc: (none)
  IKEv1 SA: local 10.2.3.26/500 remote 10.0.3.37/500 Active 
          Capabilities:(none) connid:1001 lifetime:19:55:15
  IPSEC FLOW: permit 47 host 10.2.3.26 host 10.0.3.37 
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 14 drop 0 life (KB/Sec) 4216595/2648
        Outbound: #pkts enc'ed 18 drop 0 life (KB/Sec) 4216595/2648
```