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
    
      Выполнено:
      
        vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        sdc                    8:32   0  2.5G  0 disk

4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

        vagrant@vagrant:~$ sudo fdisk /dev/sdb

        Command (m for help): n
        Partition type
           p   primary (0 primary, 0 extended, 4 free)
           e   extended (container for logical partitions)
        Select (default p): p
        Partition number (1-4, default 1): 1
        First sector (2048-5242879, default 2048): 2048
        Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

        Created a new partition 1 of type 'Linux' and of size 2 GiB.

        Command (m for help): n
        Partition type
           p   primary (1 primary, 0 extended, 3 free)
           e   extended (container for logical partitions)
        Select (default p): p
        Partition number (2-4, default 2): 2
        First sector (4196352-5242879, default 4196352):  4196352
        Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 5242879

        Created a new partition 2 of type 'Linux' and of size 511 MiB.

        Command (m for help): w
        The partition table has been altered.
        Calling ioctl() to re-read partition table.
        Syncing disks.

        vagrant@vagrant:~$ sudo fdisk -l
        .......
        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdb1          2048 4196351 4194304    2G 83 Linux
        /dev/sdb2       4196352 5242879 1046528  511M 83 Linux
        .......

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

 
