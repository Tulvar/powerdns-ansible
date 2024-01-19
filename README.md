# powerdns-ansible
## Как использовать

## Forward zone

При добавлении новой зоны, чтобы рекурсивный сервер правильно отвечал необходимо добавить в файле `/roles/dns-zones/defaults/rus.yml ` в переменной `pdns_rec_config` в `forward-zones`. 
-  Если зона храниться на той же машине в авторитетном сервер `zone.name={{ ansible_default_ipv4.address }}:5300`
-  Если зона храниться на другом сервер и рекурсивный сервер пересылает запросы туда `example.xyz=10.2.0.1:53;10.2.0.2:53;10.2.0.3:53`


```
forward-zones: example.ru={{ ansible_default_ipv4.address }}:5300, zopa.com={{ ansible_default_ipv4.address }}:5300, example-dev.ru=10.2.0.1:53;10.2.0.2:53;10.2.0.3:53
```

### Добавляем запись
#### A и CNAME
Для добавления записи мы правим файл `/roles/dns-zones/defaults/main.yml ` находим переменную `dns_zones:` там ищем `records:`
и добавляем A или CNAME запись. Переменная `location` указывается только в том случае если DNS сервер находиться в другой локации (стране, планете, ЦОДе) и там остро необходима DNS запись отличная основной локации которой является ЦОД.

```
    records:
      - name: "lupa"
        type: "a"
        ip: "10.5.5.10"
        ttl: "5400"
      - name: "pupa"
        type: "a"        
        ip: "1.4.55.1"
        ipv6: 11:11:11:11
        ttl: "3600"
        location: uzb
      - name: srv777
        type: cname
        target: yandex.ru
```
#### Services
Служебная запись (SRV-запись) — стандарт в DNS, определяющий местоположение, то есть имя хоста и номер порта серверов для определённых служб. Ищем (или добавляем к переменной )`dns_zones:` переменную `services:` и заполняем как указано  примере. Напомню, что `target:` должен резолвиться в какой-то IP (где то должа быть A запись).

```
    services:
      - name: _ldap._tcp
        weight: 100
        port: 88
        priority: 10
        target: dc001.corp.example.com
```
#### MX 
За́пись MX (от англ. mail exchanger) — тип DNS-записи, предназначенный для маршрутизации электронной почты с использованием протокола SMTP. Добавляем в `dns_zones:` переменную `mail_servers:`. Где-то должна быть А запись указывающая на хост в MX.


```
    mail_servers:
      - name: mx1
        priority: 10
      - name: mx2
        priority: 20
```
#### TXT 
TXT (Text string) — запись, которая содержит любую текстовую информацию о домене.  Добавляем в `dns_zones:` переменную `text:`. В поле `text:` указываем любую необходимую информацию

```
    text:
      - name: ololo
        text: atatatattata
```
#### NAPTR
NAPTR-записи чаще всего используются для приложений интернет-телефонии, например при отображении серверов и адресов пользователей в протоколе установления сеанса (SIP). 
Добавляем в `dns_zones:` переменную `naptr:`. Данная запись не тестировалась.

```
    naptr:                      
      - name: "sip"             
        order: 100
        pref: 10
        flags: "S"
        service: "SIP+D2T"
        regex: "!^.*$!sip:customer-service@example.com!"
        replacement: "_sip._tcp.example.com."
```
Для создания новой зоны надо:
1. Определиться с ее названием
2. Определиться с NS серверами

После этого можно в переменной `dns_zones:`указываться имя name: "example.ru" и NS сервера в `name_servers:`. В `location` необходимо описывать все DNS сервера в соответствии с их физическим местоположением (перед этим необходимо ознакомитсья с файлом inventory.ini).
`rus` это основная зона.

```
dns_zones:
  - name: "example.ru"
    name_servers:
      - name: "dev-powerdns01"
        location: rus
      - name: "dev-powerdns02"
        location: rus
      - name: "dev-powerdns03" 
        location: uzb
      - name: "dev-powerdns04"
        location: uzb
```

Пример заполенной зоны:

```
dns_zones:
  - name: "example.ru"
    name_servers:
      - name: "dev-powerdns01"
        location: rus
      - name: "dev-powerdns02"
        location: rus
      - name: "dev-powerdns03" 
        location: uzb
      - name: "dev-powerdns04"
        location: uzb
    mail_servers:
      - name: mx1
        priority: 10
      - name: mx2
        priority: 20
    records:
      - name: "dev-powerdns01"
        type: "a"
        ip: "10.1.11.10"
        ttl: "3600"
      - name: "dev-powerdns02"
        type: "a"
        ip: "10.1.11.11"
        ttl: "3600"
      - name: "dev-powerdns03"
        type: "a"
        ip: "10.1.11.10"
        ttl: "3600"
        location: uzb
      - name: "dev-powerdns04"
        type: "a"
        ip: "10.1.11.14"
        ttl: "3600"
        location: uzb
      - name: "lupa"
        type: "a"
        ip: "10.228.228.228"
        ttl: "5400"
      - name: "pupa"
        type: "a"        
        ip: "1.88.5.1"
        ipv6: 2001:db8::2
        ttl: "3600"
      - name: "pupa"
        type: "a"        
        ip: "1.5.88.1"
        ipv6: zopa:gtr:1:2
        ttl: "3600"
        location: uzb
      - name: "ololoshka"
        type: "a"        
        ip: "1.1.1.21"
        ttl: "3600"
      - name: "ololoshka"
        type: "a"        
        ip: "10.11.11.15"
        ttl: "3600"
      - name: srv777
        type: cname
        target: yandex.ru
    services:
      - name: _ldap._tcp
        weight: 100
        port: 88
        priority: 10
        target: dc001.corp.domen.com
    text:
      - name: ololo
        text: atatatattata
    naptr:                      
      - name: "sip"              
        order: 100
        pref: 10
        flags: "S"
        service: "SIP+D2T"
        regex: "!^.*$!sip:customer-service@example.com!"
        replacement: "_sip._tcp.example.com."

```

## Reverse zone
Обратная зона создается автоматически для А записей если сети описаны в переменной `networks:`. переменную можно найти в `/roles/dns-zones/defaults/main.yml`. В последстии таска найдет эту сеть и вытащит все IP из зон.
 
```
    networks:
      - '10.1.11'
      - '10.1.12'
      - '10.11'
      - '10.228'
```
## Inventory
Тут содержаться DNS сервера согласно их физическому местоположению. Указывется из местоположение допустим `[uzb]`, что условно будет обозначать Узбекистан. Далее описывается переменная `location` для этой зоны. В последствии она будет определять какие конкретные записи для конкретного DNS сервера надо поместить в файл зоны.

```
[rus]
dev-powerdns01 ansible_host=10.1.1.1

[uzb]
dev-powerdns02 ansible_host=10.1.1.2

[rus:vars]
location=rus

[uzb:vars]
location=uzb
```



