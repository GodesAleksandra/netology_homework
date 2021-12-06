1. Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
```
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32
```

      route-views>show ip route x.x.x.x
      Routing entry for x.x.y.0/20, supernet
        Known via "bgp 6447", distance 20, metric 0
        Tag 6939, type external
        Last update from 64.71.137.241 2w4d ago
        Routing Descriptor Blocks:
        * 64.71.137.241, from 64.71.137.241, 2w4d ago
            Route metric is 0, traffic share count is 1
            AS Hops 3
            Route tag 6939
            MPLS label: none

      route-views>show bgp  x.x.x.x/32
      % Network not in table

      route-views>show bgp x.x.x.x
      BGP routing table entry for x.x.y.0/20, version 1374483228
      Paths: (24 available, best #23, table default)
        Not advertised to any peer
        Refresh Epoch 1
        4901 6079 9002 9002 28890, (aggregated by 28890 x.x.y.y)
          162.250.137.254 from 162.250.137.254 (162.250.137.254)
            Origin IGP, localpref 100, valid, external, atomic-aggregate
            Community: 65000:10100 65000:10300 65000:10400
            path 7FE0124DDC08 RPKI State not found
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 3
        ........

2. Создайте dummy0 интерфейс в Ubuntu. Добавьте несколько статических маршрутов. Проверьте таблицу маршрутизации.



3. Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.

4. Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?

5. Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали. 

6*. Установите Nginx, настройте в режиме балансировщика TCP или UDP.

7*. Установите bird2, настройте динамический протокол маршрутизации RIP.

8*. Установите Netbox, создайте несколько IP префиксов, используя curl проверьте работу API.

