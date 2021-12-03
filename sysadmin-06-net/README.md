1. Работа c HTTP через телнет.
- Подключитесь утилитой телнет к сайту stackoverflow.com
`telnet stackoverflow.com 80`
- отправьте HTTP запрос
```bash
GET /questions HTTP/1.0
HOST: stackoverflow.com
[press enter]
[press enter]
```
- В ответе укажите полученный HTTP код, что он означает?

      vagrant@vagrant:~$ telnet stackoverflow.com 80
      Trying 151.101.65.69...
      Connected to stackoverflow.com.
      Escape character is '^]'.
      GET /questions HTTP/1.0
      HOST: stackoverflow.com

      HTTP/1.1 301 Moved Permanently
      cache-control: no-cache, no-store, must-revalidate
      location: https://stackoverflow.com/questions
      x-request-guid: 7facec5f-e6cd-4d59-85a2-6f3583398f9f
      feature-policy: microphone 'none'; speaker 'none'
      content-security-policy: upgrade-insecure-requests; frame-ancestors 'self' https://stackexchange.com
      Accept-Ranges: bytes
      Date: Fri, 03 Dec 2021 19:24:20 GMT
      Via: 1.1 varnish
      Connection: close
      X-Served-By: cache-ams21082-AMS
      X-Cache: MISS
      X-Cache-Hits: 0
      X-Timer: S1638559460.244161,VS0,VE75
      Vary: Fastly-SSL
      X-DNS-Prefetch-Control: off
      Set-Cookie: prov=e2d7a334-f652-d3e4-5265-1f3223c2f7fb; domain=.stackoverflow.com; expires=Fri, 01-Jan-2055 00:00:00 GMT; path=/; HttpOnly
      
   Получена ошибка с кодом 301, которая означает, что запрошенный ресурс был на постоянной основе перемещён в новое месторасположение, и указывающий на то, что текущие ссылки, использующие данный URL, должны быть обновлены. Новое расположение ресурса: https://stackoverflow.com/questions

2. Повторите задание 1 в браузере, используя консоль разработчика F12.
- откройте вкладку `Network`
- отправьте запрос http://stackoverflow.com
- найдите первый ответ HTTP сервера, откройте вкладку `Headers`
- укажите в ответе полученный HTTP код.
- проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
- приложите скриншот консоли браузера в ответ.

      Request URL: http://stackoverflow.com/
      Request Method: GET
      Status Code: 307 Internal Redirect
      Referrer Policy: strict-origin-when-cross-origin
      Location: https://stackoverflow.com/
      Non-Authoritative-Reason: HSTS
      Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
      DNT: 1
      Upgrade-Insecure-Requests: 1
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36
      
   Запрос с ошибкой обрабатывался 4 мс, а следующий запрос к сайту https://stackoverflow.com/questions обрабатывался долше всего: 482 мс

3. Какой IP адрес у вас в интернете?

    Свой внешний ip-адрес можно узнать, выполнив команду:
        
        vagrant@vagrant:~$ wget -qO- ipinfo.io/ip

4. Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой `whois`

        vagrant@vagrant:~$ whois XXX.XX.XXX.XX
    
    Провайдер: INSYS LLC
    
    AS: AS28890

5. Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой `traceroute`

            vagrant@vagrant:~$ sudo traceroute -An 8.8.8.8
            traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
             1  10.0.2.2 [*]  0.218 ms  0.192 ms  0.179 ms
             2  * * *
             3  * * *
             4  * * *
             5  * * *
             6  * * *
             7  * * *
             8  * * *
             9  * * *
            10  * * *
            11  * * *
            12  * * *
            13  * * *
            14  * * *
            15  * * *
            16  * * *
            17  * * *
            18  * * *
            19  * * *
            20  * * *
            21  * * *
            22  * * *
            23  * * *
            24  * * *
            25  * * *
            26  * * *
            27  * * *
            28  * * *
            29  * * *
            30  * * *
            
      Командой traceroute не удается узнать через какие сети проходит пакет, отправленный с моего компьютера на адрес 8.8.8.8

6. Повторите задание 5 в утилите `mtr`. На каком участке наибольшая задержка - delay?

            vagrant@vagrant:~$ mtr -zn 8.8.8.8

                                                     My traceroute  [v0.93]
            vagrant (10.0.2.15)                                                2021-12-03T21:07:54+0000
            Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                               Packets               Pings
             Host                                            Loss%   Snt   Last   Avg  Best  Wrst StDev
             1. AS???    10.0.2.2                             0.0%    14    0.7   1.6   0.2  14.8   3.8
             2. AS???    10.0.0.1                             0.0%    14  148.3 149.4 146.4 153.5   2.0
             3. AS28890  217.24.184.33                        0.0%    14  150.8 150.1 147.6 153.3   1.4
             4. AS???    10.66.2.173                          0.0%    14  148.6 149.5 146.9 153.5   2.0
             5. AS???    10.10.21.93                          0.0%    14  153.2 179.7 149.6 336.2  56.2
             6. AS28890  217.24.176.200                       7.1%    14  149.6 158.6 149.6 196.8  13.9
             7. AS12389  188.254.63.213                      76.9%    14  150.1 155.0 150.1 158.5   4.4
             8. AS12389  87.226.183.89                        0.0%    14  174.4 177.2 173.9 183.5   3.7
             9. AS12389  5.143.253.105                       84.6%    14  175.0 175.0 175.0 175.0   0.0
            10. AS15169  108.170.250.66                       0.0%    14  176.7 176.5 174.7 178.9   1.2
            11. AS15169  209.85.255.136                      15.4%    14  191.0 190.8 188.6 198.1   2.5
            12. AS15169  72.14.238.168                        0.0%    13  195.6 203.3 193.5 253.4  17.3
            13. AS15169  216.239.47.165                       0.0%    13  192.8 193.5 191.6 197.9   1.9
            14. (waiting for reply)
            15. (waiting for reply)
            16. (waiting for reply)
            17. (waiting for reply)
            18. (waiting for reply)
            19. (waiting for reply)
            20. (waiting for reply)
            21. (waiting for reply)
            22. (waiting for reply)
            23. AS15169  8.8.8.8                              0.0%    13  191.1 191.0 187.9 192.7   1.6

      Наибольшую среднюю задержку можно определить по столбцу Avg. Соответственно, участок 12 с адресом 72.14.238.168 имеет наибольшую задержку

7. Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой `dig`
            
            vagrant@vagrant:~$ dig +trace @8.8.8.8 dns.google
            
            Корневые DNS сервера:
            
                  b.root-servers.net.
                  k.root-servers.net.
                  l.root-servers.net.
                  i.root-servers.net.
                  h.root-servers.net.
                  a.root-servers.net.
                  e.root-servers.net.
                  f.root-servers.net.
                  d.root-servers.net.
                  g.root-servers.net.
                  j.root-servers.net.
                  m.root-servers.net.
                  c.root-servers.net.
                  
           DNS сервера зоны google:
           
                  ns-tld5.charlestonroadregistry.com.
                  ns-tld2.charlestonroadregistry.com.
                  ns-tld1.charlestonroadregistry.com.
                  ns-tld4.charlestonroadregistry.com.
                  ns-tld3.charlestonroadregistry.com.

           DNS сервера зоны dns.google.:
           
                  ns1.zdns.google.
                  ns4.zdns.google.
                  ns2.zdns.google.
                  ns3.zdns.google.
            
            
           A-записи:
           
                  8.8.4.4
                  8.8.8.8

9. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой `dig`

                  vagrant@vagrant:~$ dig -x 8.8.4.4
                  
                  ; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.4.4
                  ;; global options: +cmd
                  ;; Got answer:
                  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54924
                  ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

                  ;; OPT PSEUDOSECTION:
                  ; EDNS: version: 0, flags:; udp: 65494
                  ;; QUESTION SECTION:
                  ;4.4.8.8.in-addr.arpa.          IN      PTR

                  ;; ANSWER SECTION:
                  4.4.8.8.in-addr.arpa.   4506    IN      PTR     dns.google.

                  ;; Query time: 2208 msec
                  ;; SERVER: 127.0.0.53#53(127.0.0.53)
                  ;; WHEN: Fri Dec 03 21:51:08 UTC 2021
                  ;; MSG SIZE  rcvd: 73
                  
                  
                  vagrant@vagrant:~$ dig -x 8.8.8.8

                  ; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.8.8
                  ;; global options: +cmd
                  ;; Got answer:
                  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24578
                  ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

                  ;; OPT PSEUDOSECTION:
                  ; EDNS: version: 0, flags:; udp: 65494
                  ;; QUESTION SECTION:
                  ;8.8.8.8.in-addr.arpa.          IN      PTR

                  ;; ANSWER SECTION:
                  8.8.8.8.in-addr.arpa.   5920    IN      PTR     dns.google.

                  ;; Query time: 2220 msec
                  ;; SERVER: 127.0.0.53#53(127.0.0.53)
                  ;; WHEN: Fri Dec 03 21:53:19 UTC 2021
                  ;; MSG SIZE  rcvd: 73

      К IP привязано доменное имя dns.google.

