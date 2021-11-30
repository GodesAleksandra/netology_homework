1. Узнайте о sparse (разряженных) файлах.
  
      Выполнено

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

      Нет, не могут, т.к. жесткая ссылка это ссылка на одну и ту же inode.
      
      Проверка:
      
        vagrant@vagrant:~$ touch file1

        vagrant@vagrant:~$ stat file1
          File: file1
          Size: 0               Blocks: 0          IO Block: 4096   regular empty file
        Device: fd00h/64768d    Inode: 131096      Links: 1
        Access: (0664/-rw-rw-r--)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
        Access: 2021-11-30 16:59:43.876005760 +0000
        Modify: 2021-11-30 16:59:43.876005760 +0000
        Change: 2021-11-30 16:59:43.876005760 +0000
         Birth: -

        vagrant@vagrant:~$ ln file1 file1_hardlink

        vagrant@vagrant:~$ sudo chown root file1

        vagrant@vagrant:~$ sudo chmod a+x file1_hardlink

        vagrant@vagrant:~$ stat file1_hardlink
          File: file1_hardlink
          Size: 0               Blocks: 0          IO Block: 4096   regular empty file
        Device: fd00h/64768d    Inode: 131096      Links: 2
        Access: (0775/-rwxrwxr-x)  Uid: (    0/    root)   Gid: ( 1000/ vagrant)
        Access: 2021-11-30 16:59:43.876005760 +0000
        Modify: 2021-11-30 16:59:43.876005760 +0000
        Change: 2021-11-30 17:11:51.856031674 +0000
         Birth: -

        vagrant@vagrant:~$ stat file1
          File: file1
          Size: 0               Blocks: 0          IO Block: 4096   regular empty file
        Device: fd00h/64768d    Inode: 131096      Links: 2
        Access: (0775/-rwxrwxr-x)  Uid: (    0/    root)   Gid: ( 1000/ vagrant)
        Access: 2021-11-30 16:59:43.876005760 +0000
        Modify: 2021-11-30 16:59:43.876005760 +0000
        Change: 2021-11-30 17:11:51.856031674 +0000
         Birth: -

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

8. Создайте 2 независимых PV на получившихся md-устройствах.

9. Создайте общую volume-group на этих двух PV.

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

11. Создайте `mkfs.ext4` ФС на получившемся LV.

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

14. Прикрепите вывод `lsblk`.

15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

17. Сделайте `--fail` на устройство в вашем RAID1 md.

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

20. Погасите тестовый хост, `vagrant destroy`.

 
