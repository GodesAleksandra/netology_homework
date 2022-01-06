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
        
        vagrant@vagrant:~$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
          > -keyout /etc/ssl/private/apache-selfsigned.key \
          > -out /etc/ssl/certs/apache-selfsigned.crt \
          > -subj "/C=RU/ST=Moscow/L=Moscow/O=Company Name/OU=Org/CN=www.test.ru"
          Generating a RSA private key
          ..+++++
          .....................................................+++++
          writing new private key to '/etc/ssl/private/apache-selfsigned.key'
          -----
          
        vagrant@vagrant:~$ sudo nano /etc/apache2/sites-available/www.test.ru.conf
        
        <VirtualHost *:443>
         ServerName www.test.ru
         DocumentRoot /var/www/www.test.ru
         SSLEngine on
         SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
         SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
        </VirtualHost>
        
        vagrant@vagrant:~$ sudo mkdir /var/www/www.test.ru
        
        vagrant@vagrant:~$ sudo nano /var/www/www.test.ru/index.html
        
        <h1>Holla! It works!</h1>
        
        vagrant@vagrant:~$ sudo a2ensite www.test.ru.conf
          Enabling site www.test.ru.
          To activate the new configuration, you need to run:
            systemctl reload apache2
          
        vagrant@vagrant:~$ sudo apache2ctl configtest
          Syntax OK
          
        vagrant@vagrant:~$ sudo systemctl reload apache2
        
        
     В Vagrantfile:
        
         config.vm.network "forwarded_port", guest: 443, host: 443
         
         config.vm.network "private_network", ip: "192.168.33.10"
         
        
     В etc\hosts:
        
        192.168.33.10   test.ru

        
     Проверяю в браузере: https://test.ru/
        
     Результат:     http://joxi.ru/BA0gNZ0cvyVkQ2

4. Проверьте на TLS уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК ... и тому подобное).

        vagrant@vagrant:~$ git clone --depth 1 https://github.com/drwetter/testssl.sh.git
        
        vagrant@vagrant:~$ cd testssl.sh

        vagrant@vagrant:~/testssl.sh$ ./testssl.sh -U --sneaky https://www.netology.ru/
          ...
          Testing all IPv4 addresses (port 443): 104.22.40.171 104.22.41.171 172.67.21.207
          ------------------------------------------------------------------
           Start 2022-01-05 21:07:38        -->> 104.22.40.171:443 (www.netology.ru) <<--
           ...
            Testing vulnerabilities

             Heartbleed (CVE-2014-0160)                not vulnerable (OK), no heartbeat extension
             CCS (CVE-2014-0224)                       not vulnerable (OK)
             Ticketbleed (CVE-2016-9244), experiment.  not vulnerable (OK), no session tickets
             ROBOT                                     not vulnerable (OK)
             Secure Renegotiation (RFC 5746)           OpenSSL handshake didn't succeed
             Secure Client-Initiated Renegotiation     not vulnerable (OK)
             CRIME, TLS (CVE-2012-4929)                not vulnerable (OK)
             BREACH (CVE-2013-3587)                    no gzip/deflate/compress/br HTTP compression (OK)  - only supplied "/" tested
             POODLE, SSL (CVE-2014-3566)               not vulnerable (OK)
             TLS_FALLBACK_SCSV (RFC 7507)              Downgrade attack prevention supported (OK)
             SWEET32 (CVE-2016-2183, CVE-2016-6329)    VULNERABLE, uses 64 bit block ciphers
             FREAK (CVE-2015-0204)                     not vulnerable (OK)
             DROWN (CVE-2016-0800, CVE-2016-0703)      not vulnerable on this host and port (OK)
                                                       make sure you don't use this certificate elsewhere with SSLv2 enabled services
                                                       https://censys.io/ipv4?q=0E745E5E77A60345EB6E6B33B99A36286C2203D687F3377FBC685B2434518C53 could help you to find out
             LOGJAM (CVE-2015-4000), experimental      not vulnerable (OK): no DH EXPORT ciphers, no DH key detected with <= TLS 1.2
             BEAST (CVE-2011-3389)                     TLS1: ECDHE-RSA-AES128-SHA AES128-SHA
                                                             ECDHE-RSA-AES256-SHA AES256-SHA DES-CBC3-SHA
                                                       VULNERABLE -- but also supports higher protocols  TLSv1.1 TLSv1.2 (likely mitigated)
             LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches
             Winshock (CVE-2014-6321), experimental    not vulnerable (OK)
             RC4 (CVE-2013-2566, CVE-2015-2808)        no RC4 ciphers detected (OK)
           ...
           
     Выявлены уязвимости: 
      - Secure Renegotiation (у нарушителя появляется возможность добавить произвольное сообщение перед настоящими сообщениями клиента и заполучить у сервера полное доверие к  этим сообщениям)
      - SWEET32 (позволяет расшифровать данные, используя birthday attack в отношении продолжительно существующих, зашифрованных  сессий)
      - BEAST (уязвимость на стороне клиента)
      - потенциально LUCKY13 (связана с недоработкой в спецификациях TLS)


5. Установите на Ubuntu ssh сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

          vagrant@vagrant:~$ sudo apt install openssh-server
          
          vagrant@vagrant:~$ sudo systemctl start sshd.service
          
          vagrant@vagrant:~$ sudo systemctl enable sshd.service
          
          vagrant@vagrant:~$ ssh-keygen
            Generating public/private rsa key pair.
            Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
            Enter passphrase (empty for no passphrase):
            Enter same passphrase again:
            Your identification has been saved in /home/vagrant/.ssh/id_rsa
            Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
            The key fingerprint is:
            SHA256:l6n6/lHBftNpihIFIG/wWemCObTD9TCRsOYZMnauy8U vagrant@vagrant
            The key's randomart image is:
            +---[RSA 3072]----+
            |     +.o+o.      |
            |     .*=o...     |
            |   +o====  .o    |
            |  . BB+. o.+ . ..|
            |     +o S.+ o oo.|
            |    o    o....o. |
            |   . E  .... .   |
            |  . o  .  ..     |
            |   o  .oo..      |
            +----[SHA256]-----+
          
          vagrant@vagrant:~$ ssh-copy-id vagrant@127.0.0.1
            /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
            /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
            /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
            vagrant@127.0.0.1's password:

            Number of key(s) added: 1

            Now try logging into the machine, with:   "ssh 'vagrant@127.0.0.1'"
            and check to make sure that only the key(s) you wanted were added.
            
          
          
 
6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH клиента, так чтобы вход на удаленный сервер осуществлялся по имени сервера.

  

7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.

  
