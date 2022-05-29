# Домашнее задание к занятию "3.4. Операционные системы, лекция 2" - yakovlev_vs

#### 1.На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:

- поместите его в автозагрузку,
- предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron),
- удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, 
а после перезагрузки автоматически поднимается.

Решение 

Процесс установки
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz
--2022-05-10 19:46:13--  https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz
Распознаётся github.com (github.com)… 140.82.121.4
Подключение к github.com (github.com)|140.82.121.4|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 302 Found
Адрес: https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/f9ad431c-bedd-4b4e-8d3c-c9c8f8279da6?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220510%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220510T164439Z&X-Amz-Expires=300&X-Amz-Signature=a1bfaa1adba5df4a73eef7783e36b838a1a5ce1f6570b0d2dd8f491dbc7b705c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.3.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [переход]
--2022-05-10 19:46:13--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/f9ad431c-bedd-4b4e-8d3c-c9c8f8279da6?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220510%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220510T164439Z&X-Amz-Expires=300&X-Amz-Signature=a1bfaa1adba5df4a73eef7783e36b838a1a5ce1f6570b0d2dd8f491dbc7b705c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.3.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Распознаётся objects.githubusercontent.com (objects.githubusercontent.com)… 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Подключение к objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 9030402 (8,6M) [application/octet-stream]
Сохранение в: «node_exporter-1.3.0.linux-amd64.tar.gz»

node_exporter-1.3.0.linux-amd64.tar.gz         100%[=================================================================================================>]   8,61M  4,16MB/s    за 2,1s

2022-05-10 19:46:16 (4,16 MB/s) - «node_exporter-1.3.0.linux-amd64.tar.gz» сохранён [9030402/9030402]

root@debian-srv:~# tar zxvf node_exporter-*.linux-amd64.tar.gz
node_exporter-1.3.0.linux-amd64/
node_exporter-1.3.0.linux-amd64/LICENSE
node_exporter-1.3.0.linux-amd64/NOTICE
node_exporter-1.3.0.linux-amd64/node_exporter
root@debian-srv:~# cd node_exporter-*.linux-amd64
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# cp node_exporter /usr/local/bin/
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# useradd --no-create-home --shell /bin/false nodeusr
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# chown -R nodeusr:nodeusr /usr/local/bin/node_exporter
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# nano /etc/systemd/system/node_exporter.service
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# nano /etc/systemd/system/node_exporter.service
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# systemctl daemon-reload
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# systemctl enable node_exporter
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /etc/systemd/system/node_exporter.service.
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# systemctl start node_exporter
root@debian-srv:~/node_exporter-1.3.0.linux-amd64# systemctl status node_exporter
● node_exporter.service - Node Exporter Service
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-05-10 19:49:39 MSK; 6s ago
   Main PID: 2514 (node_exporter)
      Tasks: 4 (limit: 4660)
     Memory: 4.8M
        CPU: 5ms
     CGroup: /system.slice/node_exporter.service
             └─2514 /usr/local/bin/node_exporter
```
unit file для нового сервиса выглядит так:
```bash
[Unit]
Description=Node Exporter Service
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
После перезагрузки сервис так же работает. 

Возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron) проверил таким образом

В файл `/etc/default/cron` вносится переменная `MY_OPTIONS="--collector.cpu.guest --collector.cpu.info"` После этого нужно поправить файл `unit` для демона `node_exporter` и перезапустить конфиг юнитов. 

Измененный файл `unit` 
```bash
[Unit]
Description=Node Exporter Service
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
EnvironmentFile=/etc/default/cron
ExecStart=/usr/local/bin/node_exporter $MY_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Теперь проверяем 
```bash
root@debian-srv:~# systemctl daemon-reload
root@debian-srv:~# systemctl start node_exporter
root@debian-srv:~# systemctl status node_exporter
● node_exporter.service - Node Exporter Service
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-05-29 22:20:25 MSK; 2s ago
   Main PID: 2099 (node_exporter)
      Tasks: 4 (limit: 4660)
     Memory: 4.7M
        CPU: 5ms
     CGroup: /system.slice/node_exporter.service
             └─2099 /usr/local/bin/node_exporter --collector.cpu.guest --collector.cpu.info

мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=thermal_zone
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=time
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=timex
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=udp_queues
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=uname
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=vmstat
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=xfs
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:115 level=info collector=zfs
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
мая 29 22:20:25 debian-srv node_exporter[2099]: ts=2022-05-29T19:20:25.080Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```


#### 2. Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

Решение

Количество измеряемых метрик довольно велико.

Наиболее полезные:
```bash
CPU:
    node_cpu_seconds_total{cpu="0",mode="idle"} 2819.48
    node_cpu_seconds_total{cpu="0",mode="system"} 6.18
    node_cpu_seconds_total{cpu="0",mode="user"} 14.32
    process_cpu_seconds_total
    
Memory:
    node_memory_MemAvailable_bytes  
    node_memory_MemFree_bytes  
    node_memory_MemTotal_bytes
    node_memory_SwapCached_bytes  
    node_memory_SwapFree_bytes 
    node_memory_SwapTotal_bytes
    
Disk(если несколько дисков то для каждого):
    node_disk_io_time_seconds_total{device="sda"} 
    node_disk_read_bytes_total{device="sda"} 
    node_disk_read_time_seconds_total{device="sda"} 
    node_disk_write_time_seconds_total{device="sda"}
    
Network(так же для каждого активного адаптера):
    node_network_receive_errs_total
    node_network_transmit_errs_total
    node_network_flags
    node_network_iface_id
    node_network_iface_link
    node_network_mtu_bytes
    node_network_receive_bytes_total
    node_network_speed_bytes
    node_network_transmit_bytes_total
```
#### 3. Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata). После успешной установки:

- в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
- добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте vagrant reload:
```bash
config.vm.network "forwarded_port", guest: 19999, host: 19999
```
После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) 
вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

Решение 

![](picture/debian-srv_netdata_dashboard.jpg)

Все получилось. Только машина на которой я делаю работу не Vagrant и подключался к dashboard я из локальной сети.

#### 4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Решение

Ответ - ДА!
```bash
root@debian-srv:~# dmesg > dmesg && cat dmesg | grep virtual*
[    0.001051] CPU MTRRs all blank - virtualized system.
[    0.097366] Booting paravirtualized kernel on KVM
[    0.260019] Performance Events: PMU not available due to virtualization, using software events only.
[    4.064669] systemd[1]: Detected virtualization oracle.
```
#### 5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа `(ulimit --help)`?
Решение 
```bash
root@debian-srv:~# /sbin/sysctl -n fs.nr_open
1048576
```
fs.nr_open - максимальное количество файлов, которое может быть выделено одним процессом. Максимальное количество файловых дескрипторов, поддерживаемых ядром, то есть максимальное количество файловых дескрипторов, используемых процессом. 
Число задается кратное 1024, в данном случае =1024*1024.

Максимальный предел ОС можно посмотреть так:
```bash
root@debian-srv:~# cat /proc/sys/fs/file-max
9223372036854775807
```
мягкий лимит (так же ulimit -n)на пользователя (может быть увеличен процессов в процессе работы)
```bash
root@debian-srv:~# ulimit -Sn
1024
```
жесткий лимит на пользователя (не может быть увеличен, только уменьшен)
```bash
root@debian-srv:~# ulimit -Hn
1048576
```

Оба ulimit -n не могут превысить системный fs.nr_open

#### 6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root `(sudo -i)`. Под обычным пользователем требуются дополнительные опции `(--map-root-user)` и т.д.

Решение
```bash
root@debian-srv:~# ps -e |grep sleep
   3830 pts/0    00:00:00 sleep
root@debian-srv:/# nsenter -t 3830 -p -m
root@debian-srv:/# ps
    PID TTY          TIME CMD
      1 pts/0    00:00:00 sleep
      2 pts/0    00:00:00 bash
      5 pts/0    00:00:00 ps
```
#### 7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Решение 

:(){ :|:& };: = это форк-бомба, функция, которая порождает себя n-раз, до исчерпания ресурсов системы.

Механизм автоматической стабилизации - fork rejected by pids controller in /user.slice/user-1000.slice/session-3.scope

Ограничение максимального числа задач.
По умолчанию TaskMax равен 33% (от ограничений user.slice и зависит от размера оперативной памяти выделенной ОС), его можно увеличить в файле /usr/lib/systemd/system/user-.slice.d/10-defaults.conf
```bash
vagrant@vagrant:~$ systemctl status
 vagrant
    State: running
     Jobs: 0 queued
   Failed: 0 units
    Since: Sat 2021-06-26 04:40:28 UTC; 1h 12min ago
   CGroup: /
           ├─user.slice
           │ └─user-1000.slice
```