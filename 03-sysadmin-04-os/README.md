# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
    
Ответ:sudo nano /etc/systemd/system/node_exporter.service
Внес следующие данные:                                 
[Unit]
Description=node_exporter_logs

[Service]
ExecStart=/home/vagrant/node_exporter-1.2.2.linux-amd64/node_exporter
Restart=Always

[Install]
WantedBy=default.target

Добавил сервис в автозагрузку, запустил, проверил:
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter

 node_exporter.service - node_exporter_logs
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-09-23 12:52:16 UTC; 42min ago
   Main PID: 587 (node_exporter)
      Tasks: 4 (limit: 1071)
     Memory: 14.3M
     CGroup: /system.slice/node_exporter.service
             └─587 /home/vagrant/node_exporter-1.2.2.linux-amd64/node_exporter

Sep 23 12:52:17 vagrant node_exporter[587]: level=info ts=2021-09-23T12:52:17.370Z caller=node_exporter.go:115 collecto>
Sep 23 12:52:17 vagrant node_exporter[587]: level=info ts=2021-09-23T12:52:17.370Z caller=node_exporter.go:115 collecto>
Sep 23 12:52:17 vagrant node_exporter[587]: level=info ts=2021-09-23T12:52:17.370Z caller=node_exporter.go:115 collecto>

После перезапуска VM процесс запускается корректно.

2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
Ответ: CPU
node_cpu_seconds_total{cpu="0",mode="idle"} 4527.99
node_cpu_seconds_total{cpu="0",mode="iowait"} 0.74
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 0

Memory
process_virtual_memory_max_bytes 1.8446744073709552e+19
process_virtual_memory_bytes 7.3506816e+08
process_resident_memory_bytes 1.6543744e+07

Disk
node_disk_written_bytes_total{device="dm-0"} 1.427456e+07
node_disk_written_bytes_total{device="dm-1"} 0
node_disk_written_bytes_total{device="sda"} 1.4066688e+07

Network
node_sockstat_sockets_used 139
node_sockstat_UDP_mem_bytes 4096
node_sockstat_UDP_mem 1
node_sockstat_UDP_inuse 3

3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.
Ответ: вопроса не было, но все получилось)

4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
Да, например эта информация об этом говорит, насколько я понял. Если что прошу уточнить.
BOOT_IMAGE=/boot/vmlinuz-5.4.0-80-generic root=/dev/mapper/vgvagrant-root ro net.ifnames=0 biosdevname=0 quiet
[    8.483537] 11:40:33.586130 main     Executable: /opt/VBoxGuestAdditions-6.1.24/sbin/VBoxService
[    5.022145] systemd[1]: Mounting POSIX Message Queue File System...
[    5.081217] systemd[1]: Mounted POSIX Message Queue File System.

5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
fs.nr_open = 1048576 
Это означает максимальное количество дескрипторов файлов, которое может выделить процесс.
ulimit -Hn

6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
Ответ: не понял, как выводить процесс в новый неймспейс

7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Ответ: Это функция, которая параллельно пускает два своих экземпляра. Каждый пускает ещё по два и т.д. При отсутствии лимита на число процессов машина быстро исчерпывает физическую память.

Процесс не прерывается. Или это и есть "стабильность"? Ну полагаю, если изменить лимит процессов, то это изменит ситуацию, с помощью ulimit -u 

-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable

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
