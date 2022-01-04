1. Установите Bitwarden плагин для браузера. Зарегестрируйтесь и сохраните несколько паролей.

  Выполнено
  
  http://joxi.ru/BA0gNZ0cvL09V2

2. Установите Google authenticator на мобильный телефон. Настройте вход в Bitwarden акаунт через Google authenticator OTP.

  Выполнено
  
  http://joxi.ru/gmvkzwvFd9pZz2
  
  http://joxi.ru/DmB6qGLCgEX5Km
  
  http://joxi.ru/EA4EjOzcvgda5r

3. Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.

        vagrant@vagrant:~$ sudo apt install apache2
          Reading package lists... Done
          Building dependency tree
          Reading state information... Done
          The following additional packages will be installed:
          apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libjansson4
          liblua5.2-0 ssl-cert
          ...

        vagrant@vagrant:~$ sudo a2enmod ssl
          Considering dependency setenvif for ssl:
          Module setenvif already enabled
          Considering dependency mime for ssl:
          Module mime already enabled
          ...
          
        vagrant@vagrant:~$ sudo systemctl restart apache2

4. Проверьте на TLS уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК ... и тому подобное).

  

5. Установите на Ubuntu ssh сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

  
 
6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH клиента, так чтобы вход на удаленный сервер осуществлялся по имени сервера.

  

7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.

  
