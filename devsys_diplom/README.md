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

        root@vagrant:/home/vagrant# sudo apt-get install jq
        
    Запуск vault от root в фоновом режиме:
    
        root@vagrant:/home/vagrant# screen
        
        root@vagrant:/home/vagrant# vault server -dev -dev-root-token-id=root
        
    Нажать Ctrl + a + d, чтобы отделить процесс от сессии терминала.
        
        root@vagrant:/home/vagrant# export VAULT_ADDR=http://127.0.0.1:8200
        
        root@vagrant:/home/vagrant# export VAULT_TOKEN=root
        
        root@vagrant:/home/vagrant# tee admin-policy.hcl <<EOF
            > # Read system health check
            > path "sys/health"
            > {
            >   capabilities = ["read", "sudo"]
            > }
            >
            > # Create and manage ACL policies broadly across Vault
            >
            > # List existing policies
            > path "sys/policies/acl"
            > {
            >   capabilities = ["list"]
            > }
            >
            > # Create and manage ACL policies
            > path "sys/policies/acl/*"
            > {
            >   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
            > }
            >
            > # Enable and manage authentication methods broadly across Vault
            >
            > # Manage auth methods broadly across Vault
            > path "auth/*"
            > {
            >   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
            > }
            >
            > # Create, update, and delete auth methods
            > path "sys/auth/*"
            > {
            >   capabilities = ["create", "update", "delete", "sudo"]
            > }
            >
            > # List auth methods
            > path "sys/auth"
            > {
            >   capabilities = ["read"]
            > }
            >
            > # Enable and manage the key/value secrets engine at `secret/` path
            >
            > # List, create, update, and delete key/value secrets
            > path "secret/*"
            > {
            >   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
            > }
            >
            > # Manage secrets engines
            > path "sys/mounts/*"
            > {
            >   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
            > }
            >
            > # List existing secrets engines.
            > path "sys/mounts"
            > {
            >   capabilities = ["read", "list"]
            > }
            > 
            > # Work with pki secrets engine
            > path "pki*" {
            >   capabilities = [ "create", "read", "update", "delete", "list", "sudo" ]
            > }
            > EOF

        root@vagrant:/home/vagrant# vault policy write admin admin-policy.hcl
            Success! Uploaded policy: admin
            
    Генерируем корневой сертификат CA_cert.crt:
            
        root@vagrant:/home/vagrant# vault secrets enable pki
            Success! Enabled the pki secrets engine at: pki/
        
        root@vagrant:/home/vagrant# vault secrets tune -max-lease-ttl=87600h pki
            Success! Tuned the secrets engine at: pki/
    
        root@vagrant:/home/vagrant# vault write -field=certificate pki/root/generate/internal \
        >      common_name="example.com" \
        >      ttl=87600h > CA_cert.crt

        root@vagrant:/home/vagrant# vault write pki/config/urls \
        >      issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
        >      crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
        Success! Data written to: pki/config/urls
        
    Генерируем промежуточный сертификат intermediate.cert.pem:
    
        root@vagrant:/home/vagrant# vault secrets enable -path=pki_int pki
            Success! Enabled the pki secrets engine at: pki_int/
        
        root@vagrant:/home/vagrant# vault secrets tune -max-lease-ttl=43800h pki_int
            Success! Tuned the secrets engine at: pki_int/
        
        root@vagrant:/home/vagrant# vault write -format=json pki_int/intermediate/generate/internal \
            >      common_name="example.com Intermediate Authority" \
            >      | jq -r '.data.csr' > pki_intermediate.csr
        
        root@vagrant:/home/vagrant# vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \
            >      format=pem_bundle ttl="43800h" \
            >      | jq -r '.data.certificate' > intermediate.cert.pem
        
        root@vagrant:/home/vagrant# vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem
            Success! Data written to: pki_int/intermediate/set-signed
            
    Создаем роль, которая разрешает субдомены:
    
        root@vagrant:/home/vagrant# vault write pki_int/roles/example-dot-com \
        >      allowed_domains="example.com" \
        >      allow_subdomains=true \
        >      max_ttl="720h"
            Success! Data written to: pki_int/roles/example-dot-com
            
    Создаем 30-дневный сертификат test.example.com, основанный на роли example-dot-com и записываем в файл test_example.crt:
    
        root@vagrant:/home/vagrant# vault write -format=json pki_int/issue/example-dot-com common_name="test.example.com" ttl="720h" > /etc/ssl/test_example.crt
        
    Запись сертификата в файл test_example.crt.pem:
        
        root@vagrant:/home/vagrant# cat /etc/ssl/test_example.crt | jq -r .data.certificate > /etc/ssl/test_example.crt.pem

        root@vagrant:/home/vagrant# cat /etc/ssl/test_example.crt | jq -r .data.ca_chain[] >> /etc/ssl/test_example.crt.pem
        
    Запись private ключа в файл test_example.key:
    
        root@vagrant:/home/vagrant# cat /etc/ssl/test_example.crt | jq -r .data.private_key > /etc/ssl/test_example.key

5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.

        root@vagrant:/home/vagrant# cp CA_cert.crt /vagrant
    
    Сертификат установлен вручную через Менеджер сертификатов certmgr.msc в Windows
    
    ![image1](http://joxi.ru/Q2KOG6YHyV6532.jpg)

6. Установите nginx.

        root@vagrant:/home/vagrant# apt install nginx
        
        root@vagrant:/home/vagrant# systemctl status nginx
            ● nginx.service - A high performance web server and a reverse proxy server
                 Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
                 Active: active (running) since Fri 2022-01-07 22:44:57 UTC; 6min ago
                   Docs: man:nginx(8)
               Main PID: 3969 (nginx)
                  Tasks: 3 (limit: 1071)
                 Memory: 4.3M
                 CGroup: /system.slice/nginx.service
                         ├─3969 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                         ├─3970 nginx: worker process
                         └─3971 nginx: worker process

        root@vagrant:/home/vagrant# sudo mkdir -p /var/www/example/html
        
        root@vagrant:/home/vagrant# sudo chown -R $USER:$USER /var/www/example/html
        
        root@vagrant:/home/vagrant# sudo chmod -R 755 /var/www/example
    
    Создадим домашнюю страницу:
        
        root@vagrant:/home/vagrant# nano /var/www/example/html/index.html
        
        <html>
            <head>
                <title>Welcome!</title>
            </head>
            <body>
                <h1>Success! Server is available!</h1>
            </body>
        </html>
        

7. По инструкции настройте nginx на https, используя ранее подготовленный сертификат:
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;
  - можно использовать и другой html файл, сделанный вами;

        root@vagrant:/home/vagrant# nano /etc/nginx/sites-available/example
        
        server {
              listen              80;
              listen              443 ssl;

              server_name         www.test.example.com test.example.com;
              ssl_certificate     /etc/ssl/test_example.crt.pem;
              ssl_certificate_key /etc/ssl/test_example.key;

               root /var/www/example/html;
               index index.html;

               location / {
                       try_files $uri $uri/ =404;
               }
        }
        
        root@vagrant:/home/vagrant# ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/
        
        root@vagrant:/home/vagrant# nginx -t
            nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
            nginx: configuration file /etc/nginx/nginx.conf test is successful
            
        root@vagrant:/home/vagrant# systemctl restart nginx
        
8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

        ![image1](http://joxi.ru/8230E3xtaNQVOA.jpg)

9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.

    

10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

    
