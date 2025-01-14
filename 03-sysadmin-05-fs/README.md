# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
Ответ: Разрежённый файл (англ. sparse file) — файл, в котором последовательности нулевых байтов[1] заменены на информацию об этих последовательностях (список дыр).

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
Ответ: не могут, т.к. для жестких ссылок действуют те же права и данные, что и для объекта. 

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
Ответ: sudo fdisk dev/sdb
n #создаем новый раздел, далее стандартные параметры, в конце указываем размер +2G
#повторяем операцию еще раз, но в конце оставляем значение по умолчанию
#Итого два раздела на 2гб и 511мб

5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
Ответ: дал себе права администратора, потому что без него не выполнялись команды
sudo -i
запустил копирование:
sfdisk -d /dev/sdb | sfdisk /dev/sdc

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
Ответ: mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
Ответ: mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks

md1 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]

unused devices: <none>
    
8. Создайте 2 независимых PV на получившихся md-устройствах.
    Ответ: pvcreate /dev/md1 /dev/md0
  Physical volume "/dev/md1" successfully created.
  Physical volume "/dev/md0" successfully created.
root@vagrant:~# pvscan
  PV /dev/sda5   VG vgvagrant       lvm2 [<63.50 GiB / 0    free]
  PV /dev/md0                       lvm2 [1018.00 MiB]
  PV /dev/md1                       lvm2 [<2.00 GiB]
  Total: 3 [<66.49 GiB] / in use: 1 [<63.50 GiB] / in no VG: 2 [2.99 GiB]

9. Создайте общую volume-group на этих двух PV.
    Ответ: vgcreate volume-group /dev/md1 /dev/md0
  Volume group "volume-group" successfully created

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
    Ответ: lvcreate -L 100M volume-group /dev/md0
  Logical volume "lvol0" created.
    

11. Создайте `mkfs.ext4` ФС на получившемся LV.
    Ответ: mkfs.ext4 /dev/volume-group/lvol0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
    Ответ: mkdir /tmp/new #создаем директорию
    mount /dev/volume-group/lvol0 /tmp/new #монтируем раздел в директорию

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
    Ответ: root@vagrant:/tmp/new# ls
lost+found  test.gz

14. Прикрепите вывод `lsblk`.
    Ответ: lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0  512M  0 part  /boot/efi
├─sda2                      8:2    0    1K  0 part
└─sda5                      8:5    0 63.5G  0 part
  ├─vgvagrant-root        253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1      253:1    0  980M  0 lvm   [SWAP]
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md1                     9:1    0    2G  0 raid1
└─sdb2                      8:18   0  511M  0 part
  └─md0                     9:0    0 1018M  0 raid0
    └─volume--group-lvol0 253:2    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md1                     9:1    0    2G  0 raid1
└─sdc2                      8:34   0  511M  0 part
  └─md0                     9:0    0 1018M  0 raid0
    └─volume--group-lvol0 253:2    0  100M  0 lvm   /tmp/new
    

15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
    Ответ: gzip -t /tmp/new/test.gz
root@vagrant:/tmp/new# echo $?
0

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
    Ответ: pvmove /dev/md0 /dev/md1
  /dev/md0: Moved: 56.00%
  /dev/md0: Moved: 100.00%
    

17. Сделайте `--fail` на устройство в вашем RAID1 md.
    Ответ:  mdadm --fail /dev/md1  /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md1
root@vagrant:~# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks

md1 : active raid1 sdc1[1](F) sdb1[0]
      2094080 blocks super 1.2 [2/1] [U_]

unused devices: <none>

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
    Ответ: [20262.310558] md/raid1:md1: Disk failure on sdc1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
    Ответ: gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0

20. Погасите тестовый хост, `vagrant destroy`.

 
 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате Slack.

---
