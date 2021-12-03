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

        

6. Повторите задание 5 в утилите `mtr`. На каком участке наибольшая задержка - delay?
7. Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой `dig`
8. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой `dig`
