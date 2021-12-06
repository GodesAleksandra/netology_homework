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

            root@vagrant:~# modprobe -v dummy numdummies=1
            insmod /lib/modules/5.4.0-80-generic/kernel/drivers/net/dummy.ko numdummies=2 numdummies=0 numdummies=1

            root@vagrant:~# lsmod | grep dummy
            dummy                  16384  0

            root@vagrant:~# ip -c -br link | grep dummy0
            dummy0           DOWN           62:4c:c2:76:20:bb <BROADCAST,NOARP>

            root@vagrant:~# ip link set dummy0 up

            root@vagrant:~# ip route add 172.16.10.0/24 dev dummy0
            root@vagrant:~# ip route add 172.16.12.0/24 dev dummy0

            root@vagrant:~# ip route show | grep dummy0
            172.16.10.0/24 dev dummy0 scope link
            172.16.12.0/24 dev dummy0 scope link
            192.168.1.0/24 dev dummy0 proto kernel scope link src 192.168.1.150

3. Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.

            root@vagrant:~# ss -plnt
            State       Recv-Q      Send-Q           Local Address:Port           Peer Address:Port      Process
            LISTEN      0           4096                   0.0.0.0:111                 0.0.0.0:*          users:(("rpcbind",pid=555,fd=4),("systemd",pid=1,fd=35))
            LISTEN      0           4096             127.0.0.53%lo:53                  0.0.0.0:*          users:(("systemd-resolve",pid=556,fd=13))
            LISTEN      0           128                    0.0.0.0:22                  0.0.0.0:*          users:(("sshd",pid=3343,fd=3))
            LISTEN      0           4096                      [::]:111                    [::]:*          users:(("rpcbind",pid=555,fd=6),("systemd",pid=1,fd=37))
            LISTEN      0           128                       [::]:22                     [::]:*          users:(("sshd",pid=3343,fd=4))
      
      Порт 22 использует SSH, порт 53 - DNS, порт 111 - SUNRPC (Sun Remote Procedure Call)

4. Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?

            root@vagrant:~# ss -plnu
            State       Recv-Q      Send-Q            Local Address:Port           Peer Address:Port     Process
            UNCONN      0           0                 127.0.0.53%lo:53                  0.0.0.0:*         users:(("systemd-resolve",pid=556,fd=12))
            UNCONN      0           0                10.0.2.15%eth0:68                  0.0.0.0:*         users:(("systemd-network",pid=398,fd=20))
            UNCONN      0           0                       0.0.0.0:111                 0.0.0.0:*         users:(("rpcbind",pid=555,fd=5),("systemd",pid=1,fd=36))
            UNCONN      0           0                          [::]:111                    [::]:*         users:(("rpcbind",pid=555,fd=7),("systemd",pid=1,fd=38))
      
      Порт 68 использует BOOTPC (Bootstrap Protocol Client) — для клиентов бездисковых рабочих станций, загружающихся с сервера BOOTP; также используется DHCP (Dynamic Host        Configuration Protocol)

5. Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали. 

      Изображение домашней сети в приложении к этому домашнему заданию

