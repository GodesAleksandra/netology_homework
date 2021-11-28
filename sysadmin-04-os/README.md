1. На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
   
   Решение:
   
	a) В Vagrantfile был проброшен порт: 
  
	    config.vm.network "forwarded_port", guest: 9100, host: 9100
		
	b) Установлен и запущен Node Explorer:
	 
       vagrant@vagrant:~$ wget https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz

       vagrant@vagrant:~$ tar xvfz node_exporter-1.3.0.linux-amd64.tar.gz

       vagrant@vagrant:~$ cd node_exporter-1.3.0.linux-amd64

       vagrant@vagrant:~$ ./node_exporter
		 
		 
     Проверка в браузере работает, метрики есть: http://localhost:9100/metrics
	
	c) Бинарный файл Node Exporter перенесен в /usr/local/bin, для запуска из этой директории:
	
		vagrant@vagrant:~$ sudo mv node_exporter-1.3.0.linux-amd64/node_exporter /usr/local/bin/
	
	d) Создан unit-файл для запуска Node Explorer, как сервис:
  
		vagrant@vagrant:~$ sudo nano /etc/systemd/system/node_exporter.service
		
		
    Содержимое файла:
    
		
        [Unit]
        Description=Node Exporter

        [Service]
        Environment=OPTION=' '
        ExecStart=/usr/local/bin/node_exporter $OPTION

        [Install]
        WantedBy=default.target
	
		
    В Environment файла node_exporter.service создана опция OPTION, которая передается в запускаемый процесс.
		
	e) Запуск и проверка:
	
		vagrant@vagrant:~$ sudo systemctl daemon-reload
		
		vagrant@vagrant:~$ sudo systemctl start node_exporter
		
		vagrant@vagrant:~$ systemctl status node_exporter.service
		
			● node_exporter.service - Node Exporter
				 Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
				 Active: active (running) since Sun 2021-11-28 13:12:53 UTC; 49min ago
			   Main PID: 1620 (node_exporter)
				  Tasks: 5 (limit: 1071)
				 Memory: 4.6M
				 CGroup: /system.slice/node_exporter.service
						 └─1620 /usr/local/bin/node_exporter
		
		vagrant@vagrant:~$ ps -e |grep node_exporter
		
		 1620 ?        00:00:00 node_exporter
		
		vagrant@vagrant:~$ curl http://localhost:9100/metrics
		
		vagrant@vagrant:~$ sudo cat /proc/1620/environ
		
		LANG=en_US.UTF-8LANGUAGE=en_US:PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
		INVOCATION_ID=b573bf32a8394166854c937425ebd6fcJOURNAL_STREAM=9:33599OPTION= 
		
		
    Переменная окружения выставляется. 
		
		
    После остановки: 	
		
        vagrant@vagrant:~$ sudo systemctl stop node_exporter
		
		
    Сервис стартует и перезапускается корректно.

2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

  CPU:
  
	
      vagrant@vagrant:~$ curl http://localhost:9100/metrics | grep "node_cpu_seconds_total"

      node_cpu_seconds_total{cpu="0",mode="idle"} 29347.24
      node_cpu_seconds_total{cpu="0",mode="iowait"} 4.01
      node_cpu_seconds_total{cpu="0",mode="irq"} 0
      node_cpu_seconds_total{cpu="0",mode="nice"} 0
      node_cpu_seconds_total{cpu="0",mode="softirq"} 2.58
      node_cpu_seconds_total{cpu="0",mode="steal"} 0
      node_cpu_seconds_total{cpu="0",mode="system"} 6.79
      node_cpu_seconds_total{cpu="0",mode="user"} 6.79
      node_cpu_seconds_total{cpu="1",mode="idle"} 29314.67
      node_cpu_seconds_total{cpu="1",mode="iowait"} 2.85
      node_cpu_seconds_total{cpu="1",mode="irq"} 0
      node_cpu_seconds_total{cpu="1",mode="nice"} 0
      node_cpu_seconds_total{cpu="1",mode="softirq"} 1.07
      node_cpu_seconds_total{cpu="1",mode="steal"} 0
      node_cpu_seconds_total{cpu="1",mode="system"} 9.44
      node_cpu_seconds_total{cpu="1",mode="user"} 7.78
    
	
	Memory:
	
  
      vagrant@vagrant:~$ curl http://localhost:9100/metrics | grep "node_memory_Mem"

      node_memory_MemAvailable_bytes 7.51644672e+08
      node_memory_MemFree_bytes 3.5145728e+08
      node_memory_MemTotal_bytes 1.028694016e+09
    
	
	Disks:
	
  
      node_disk_io_time_seconds_total{device="sda"} 
      node_disk_read_bytes_total{device="sda"} 
      node_disk_read_time_seconds_total{device="sda"} 
      node_disk_write_time_seconds_total{device="sda"}
    
	
	Network:
  
  
      node_network_receive_errs_total{device="eth0"} 
      node_network_receive_bytes_total{device="eth0"} 
      node_network_transmit_bytes_total{device="eth0"}
      node_network_transmit_errs_total{device="eth0"}

3. Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.
    
        vagrant@vagrant:~$ sudo lsof -i :19999

        COMMAND PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
        netdata 701 netdata    4u  IPv4  23023      0t0  TCP *:19999 (LISTEN)
        netdata 701 netdata   51u  IPv4  28486      0t0  TCP vagrant:19999->_gateway:58359 (ESTABLISHED)

4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

  Можно.
	
      vagrant@vagrant:~$ dmesg |grep virtual

      [    0.003208] CPU MTRRs all blank - virtualized system.
      [    0.097327] Booting paravirtualized kernel on KVM
      [    3.327625] systemd[1]: Detected virtualization oracle.

5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

  nr_open обозначает максимальное число файловых дескрипторов, которое процесс может выделить.
	Значение по умолчанию 1024*1024 (1048576).
	
      vagrant@vagrant:~$ /sbin/sysctl -n fs.nr_open
      1048576

      ulimit -Sn  #  мягкое ограничение максимального числа дескрипторов открытых файлов
      ulimit -Hn  #  жесткое ограничение максимального числа дескрипторов открытых файлов
	
	
  Мягкий предел может быть изменен в пределах жесткого ограничения.
	Для пользователей без полномочий root можно установить только жесткий предел, меньший, чем исходный.

6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

  

7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?


	Код представляет собой функцию, исполняемую рекурсивно. Тело функции вызывает само себя в фоновом процессе. Двоеточие в конце означает вызов функции.
	
	После запуска команды:
	
		vagrant@vagrant:~$ dmesg -T

		cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-5.scope
		
		hrtimer: interrupt took 307181 ns
	
	Исполнение функции было остановлено из-за ограничения числа пользовательских процессов, которые можно создать в сессии.
	
	Посмотреть лимит процессов:
	
		ulimit -u
	
	Изменить настройку, например на 5000:
	
		ulimit -S -u 5000
