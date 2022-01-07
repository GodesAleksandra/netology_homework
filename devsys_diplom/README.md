1. Создайте виртуальную машину Linux.

    Выполнено
    
    Vagrantfile:
    
        Vagrant.configure("2") do |config|
          config.vm.box = "bento/ubuntu-20.04"
          config.vm.network "forwarded_port", guest: 443, host: 443
          config.vm.network "private_network", ip: "192.168.33.10"
          config.vm.provider "virtualbox" do |v|
               v.memory = 1024
               v.cpus = 2
          end
        end


2. Установите ufw и разрешите к этой машине сессии на порты 22 и 443, при этом трафик на интерфейсе localhost (lo) должен ходить свободно на все порты.

        vagrant@vagrant:~$ sudo apt install ufw
        
        vagrant@vagrant:~$ sudo ufw allow 2222
            Rules updated
            Rules updated (v6)
            
        vagrant@vagrant:~$ sudo ufw allow 443
            Rules updated
            Rules updated (v6)
            
        vagrant@vagrant:~$ sudo ufw enable
            Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
            Firewall is active and enabled on system startup
            
        vagrant@vagrant:~$ sudo ufw status verbose
            Status: active
            Logging: on (low)
            Default: deny (incoming), allow (outgoing), disabled (routed)
            New profiles: skip

            To                         Action      From
            --                         ------      ----
            2222                       ALLOW IN    Anywhere
            443                        ALLOW IN    Anywhere
            2222 (v6)                  ALLOW IN    Anywhere (v6)
            443 (v6)                   ALLOW IN    Anywhere (v6)
            
     Проверка траффика с localhost на остальные порты:
     
        vagrant@vagrant:~$ ping -I 127.0.0.1 10.0.2.15
            PING 10.0.2.15 (10.0.2.15) from 127.0.0.1 : 56(84) bytes of data.
            ...
            5 packets transmitted, 5 received, 0% packet loss, time 4366ms
        
        vagrant@vagrant:~$ ping -I 127.0.0.1 192.168.33.10
            PING 192.168.33.10 (192.168.33.10) from 127.0.0.1 : 56(84) bytes of data.
            ...
            3 packets transmitted, 3 received, 0% packet loss, time 2044ms

3. Установите hashicorp vault.

        vagrant@vagrant:~$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
            OK

        vagrant@vagrant:~$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
            ...
            Fetched 9,132 kB in 14s (665 kB/s)
            Reading package lists... Done

        vagrant@vagrant:~$ sudo apt-get update && sudo apt-get install vault
            ...
            Vault TLS key and self-signed certificate have been generated in '/opt/vault/tls'.
            
     Проверка:
     
        vagrant@vagrant:~$ vault
            Usage: vault <command> [args]
            ...


4. Cоздайте центр сертификации по инструкции и выпустите сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - месяц).

    

5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.

    

6. Установите nginx.

    

7. По инструкции настройте nginx на https, используя ранее подготовленный сертификат:
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;
  - можно использовать и другой html файл, сделанный вами;

    

8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

    

9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.

    

10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

    
