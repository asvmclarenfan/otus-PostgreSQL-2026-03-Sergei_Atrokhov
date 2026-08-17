###
OTUS – PostgreSQL для администраторов баз данных и разработчиков
ntcn
Проектная работа по теме
###

##
Создание и тестирование высоконагруженного отказоустойчивого кластера PostgreSQL на базе Patroni
##

####
Содержимое

1. Настройка кластера ETCD
2. Настройка кластера PostgreSQL
3. Настройка кластера Patroni
4. Настройка pgbouncer
5. Настройка HAProxy
6. Настройка keepalived на нодах с HAProxy
7. Тестирование
####

####
В результате настройки в инфраструктуре получилось 8 ВМ в Yandex Cloud
####
<img width="2899" height="1914" alt="pr26_deploy" src="https://github.com/user-attachments/assets/6dd89ce5-9266-473d-a5fe-2232853b6ff9" />

###
Далее по каждому пункту представлены более подробные комментарии 
###

###
1. Настройка кластера ETCD
###
###



<img width="1728" height="972" alt="pr1_deploy" src="https://github.com/user-attachments/assets/dd2d3840-e80a-454e-b260-e089804d2225" />

<img width="1728" height="1014" alt="pr2_deploy" src="https://github.com/user-attachments/assets/bf2d299c-e451-4ea2-88b0-03b9ddfba5c8" />

<img width="2880" height="1620" alt="pr3_deploy" src="https://github.com/user-attachments/assets/61bba5c2-694d-4728-9790-d9505a2590ed" />

<img width="2681" height="972" alt="pr4_deploy" src="https://github.com/user-attachments/assets/b6e7e240-b108-4766-aa56-12688b206261" />

<img width="1728" height="972" alt="pr5_deploy" src="https://github.com/user-attachments/assets/39c9f702-c3f6-4a84-a5a8-6d38ee05cfd0" />

<img width="1728" height="972" alt="pr15_deploy" src="https://github.com/user-attachments/assets/189456b1-f3d0-46fd-87cb-2781dc999175" />

<img width="1728" height="972" alt="pr16_deploy" src="https://github.com/user-attachments/assets/817dae71-d5cc-4926-b719-d7a22634cda6" />

<img width="1728" height="972" alt="pr8_deploy" src="https://github.com/user-attachments/assets/730f481e-bb46-4574-b7a4-f9c7dd646b7e" />

<img width="2691" height="972" alt="pr9_deploy" src="https://github.com/user-attachments/assets/ef0592d4-5575-4749-8a2e-a1db1b2b6663" />

<img width="2687" height="972" alt="pr10_deploy" src="https://github.com/user-attachments/assets/e7ce50c1-4a41-4f5f-8d05-e79ab8095f8f" />

<img width="1728" height="972" alt="pr13_deploy" src="https://github.com/user-attachments/assets/fb437c63-4c1e-4358-a95d-f6f74daf0a06" />

<img width="1728" height="972" alt="pr14_deploy" src="https://github.com/user-attachments/assets/e2d6b2ce-eb48-4a20-a1f1-14ff7477715c" />

<img width="1728" height="972" alt="pr16_deploy" src="https://github.com/user-attachments/assets/1436415b-b7d7-461f-bd38-024c9aef3a62" />

<img width="1728" height="972" alt="pr17_deploy" src="https://github.com/user-attachments/assets/c73f1b3c-301a-49ff-8255-e033c2ded377" />

<img width="1728" height="972" alt="pr18_deploy" src="https://github.com/user-attachments/assets/abc17714-a5cb-4bcb-be31-6afd8594ab57" />

<img width="1728" height="972" alt="pr19_deploy" src="https://github.com/user-attachments/assets/dde6036e-563c-4a83-93b0-4b1d4bf23f7e" />

<img width="1728" height="972" alt="pr20_deploy" src="https://github.com/user-attachments/assets/5e1d003c-2d80-4137-8690-7e8547847794" />

<img width="1728" height="972" alt="pr21_deploy" src="https://github.com/user-attachments/assets/a15eacd2-cfe9-49e0-8e23-b6bdbd3ccfcb" />

<img width="1728" height="972" alt="pr22_deploy" src="https://github.com/user-attachments/assets/bfa74d39-776b-4e55-8ac0-bc8e8b5cc66e" />

<img width="2311" height="972" alt="pr23_deploy" src="https://github.com/user-attachments/assets/98107d50-aaed-433a-af3e-3af0687468bc" />

####
Список нод кластера etcd
####
```sh
vm-etcd1  10.129.0.23
vm-etcd2  10.129.0.17
vm-etcd3  10.129.0.11
```

###
Настройка etcd
###
####
Ниже показан ход настроек для ноды etcd1, для оставшихся 2 нод настройки идентичные
####
#####
Обновляем списки пакетов
#####
```sh
asvpg@vm-etcd1:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Get:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease [126 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Packages [1154 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main Translation-en [278 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Components [180 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Packages [1680 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/universe Translation-en [334 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Components [388 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/restricted amd64 Packages [1367 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/restricted Translation-en [308 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/multiverse Translation-en [12.3 kB]
Get:14 http://mirror.yandex.ru/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:15 http://mirror.yandex.ru/ubuntu noble-backports/main amd64 Components [5772 B]
Get:16 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Packages [31.0 kB]
Get:17 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Components [12.6 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [897 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [198 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [46.3 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1199 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [239 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [76.3 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [1273 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [290 kB]
Get:26 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [10.6 kB]
Fetched 10.4 MB in 2s (6546 kB/s)

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
11 packages can be upgraded. Run 'apt list --upgradable' to see them.
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ hostname
vm-etcd1
asvpg@vm-etcd1:~$
```
#####
Устанавливаем инструменты сети (для отладки) и синхронизируем время
#####
```sh
asvpg@vm-etcd1:~$ sudo apt install -y net-tools chrony dnsutils
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  tzdata tzdata-legacy
The following packages will be REMOVED:
  systemd-timesyncd
The following NEW packages will be installed:
  chrony dnsutils net-tools tzdata-legacy
The following packages will be upgraded:
  tzdata
1 upgraded, 4 newly installed, 1 to remove and 10 not upgraded.
Need to get 899 kB of archives.
After this operation, 2466 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 tzdata all 2026c-0ubuntu0.24.04.1 [280 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 tzdata-legacy all 2026c-0ubuntu0.24.04.1 [95.3 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 chrony amd64 4.5-1ubuntu4.2 [316 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 dnsutils all 1:9.18.39-0ubuntu0.24.04.5 [3686 B]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 net-tools amd64 2.10-0.1ubuntu4.4 [204 kB]
Fetched 899 kB in 0s (27.4 MB/s)
Preconfiguring packages ...
(Reading database ... 106573 files and directories currently installed.)
Removing systemd-timesyncd (255.4-1ubuntu8.16) ...
(Reading database ... 106557 files and directories currently installed.)
Preparing to unpack .../tzdata_2026c-0ubuntu0.24.04.1_all.deb ...
Unpacking tzdata (2026c-0ubuntu0.24.04.1) over (2026b-0ubuntu0.24.04.1) ...
Selecting previously unselected package tzdata-legacy.
Preparing to unpack .../tzdata-legacy_2026c-0ubuntu0.24.04.1_all.deb ...
Unpacking tzdata-legacy (2026c-0ubuntu0.24.04.1) ...
Selecting previously unselected package chrony.
Preparing to unpack .../chrony_4.5-1ubuntu4.2_amd64.deb ...
Unpacking chrony (4.5-1ubuntu4.2) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../dnsutils_1%3a9.18.39-0ubuntu0.24.04.5_all.deb ...
Unpacking dnsutils (1:9.18.39-0ubuntu0.24.04.5) ...
Selecting previously unselected package net-tools.
Preparing to unpack .../net-tools_2.10-0.1ubuntu4.4_amd64.deb ...
Unpacking net-tools (2.10-0.1ubuntu4.4) ...
Setting up net-tools (2.10-0.1ubuntu4.4) ...
Setting up dnsutils (1:9.18.39-0ubuntu0.24.04.5) ...
Setting up tzdata (2026c-0ubuntu0.24.04.1) ...

Current default time zone: 'Etc/UTC'
Local time is now:      Mon Aug  3 18:15:31 UTC 2026.
Universal Time is now:  Mon Aug  3 18:15:31 UTC 2026.
Run 'dpkg-reconfigure tzdata' if you wish to change it.

Setting up tzdata-legacy (2026c-0ubuntu0.24.04.1) ...
Setting up chrony (4.5-1ubuntu4.2) ...

Creating config file /etc/chrony/chrony.conf with new version

Creating config file /etc/chrony/chrony.keys with new version
dpkg-statoverride: warning: --update given but /var/log/chrony does not exist
Created symlink /etc/systemd/system/chronyd.service → /usr/lib/systemd/system/chrony.service.
Created symlink /etc/systemd/system/multi-user.target.wants/chrony.service → /usr/lib/systemd/system/chrony.service.
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-etcd1:~$
```

#####
Установка самого etcd (на всех трех ВМ)
#####
```sh
asvpg@vm-etcd1:~$ sudo apt install -y etcd-server etcd-client
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  etcd-client etcd-server
0 upgraded, 2 newly installed, 0 to remove and 10 not upgraded.
Need to get 14.5 MB of archives.
After this operation, 43.0 MB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 etcd-server amd64 3.4.30-1ubuntu0.24.04.3 [9184 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 etcd-client amd64 3.4.30-1ubuntu0.24.04.3 [5295 kB]
Fetched 14.5 MB in 0s (50.5 MB/s)
Selecting previously unselected package etcd-server.
(Reading database ... 107398 files and directories currently installed.)
Preparing to unpack .../etcd-server_3.4.30-1ubuntu0.24.04.3_amd64.deb ...
Unpacking etcd-server (3.4.30-1ubuntu0.24.04.3) ...
Selecting previously unselected package etcd-client.
Preparing to unpack .../etcd-client_3.4.30-1ubuntu0.24.04.3_amd64.deb ...
Unpacking etcd-client (3.4.30-1ubuntu0.24.04.3) ...
Setting up etcd-client (3.4.30-1ubuntu0.24.04.3) ...
Setting up etcd-server (3.4.30-1ubuntu0.24.04.3) ...
info: Selecting UID from range 100 to 999 ...

info: Selecting GID from range 100 to 999 ...
info: Adding system user `etcd' (UID 111) ...
info: Adding new group `etcd' (GID 114) ...
info: Adding new user `etcd' (UID 111) with group `etcd' ...
info: Creating home directory `/var/lib/etcd/' ...
Created symlink /etc/systemd/system/etcd2.service → /usr/lib/systemd/system/etcd.service.
Created symlink /etc/systemd/system/multi-user.target.wants/etcd.service → /usr/lib/systemd/system/etcd.service.
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-etcd1:~$
```

#####
Проверяем, что сервис установился и запустился
#####
```sh
asvpg@vm-etcd1:~$ hostname; ps -aef | grep etcd | grep -v grep
vm-etcd1
etcd        2505       1  0 18:20 ?        00:00:00 /usr/bin/etcd
asvpg@vm-etcd1:~$

asvpg@vm-etcd2:~$ hostname; ps -aef | grep etcd | grep -v grep
vm-etcd2
etcd        2456       1  0 18:38 ?        00:00:00 /usr/bin/etcd
asvpg@vm-etcd2:~$

asvpg@vm-etcd3:~$ hostname; ps -aef | grep etcd | grep -v grep
vm-etcd3
etcd        3496       1  0 18:43 ?        00:00:00 /usr/bin/etcd
asvpg@vm-etcd3:~$
```

#####
Останавливаем сервис etcd (чтобы не завис)
#####
```sh
asvpg@vm-etcd1:~$ sudo systemctl stop etcd
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ sudo systemctl status etcd
○ etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: inactive (dead) since Mon 2026-08-03 18:27:46 UTC; 40s ago
   Duration: 7min 12.386s
       Docs: https://etcd.io/docs
             man:etcd
    Process: 2505 ExecStart=/usr/bin/etcd $DAEMON_ARGS (code=killed, signal=TERM)
   Main PID: 2505 (code=killed, signal=TERM)
        CPU: 1.594s

Aug 03 18:20:33 vm-etcd1 etcd[2505]: set the initial cluster version to 3.4
Aug 03 18:20:33 vm-etcd1 etcd[2505]: enabled capabilities for version 3.4
Aug 03 18:27:46 vm-etcd1 etcd[2505]: received terminated signal, shutting down...
Aug 03 18:27:46 vm-etcd1 etcd[2505]: stopping insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of closed network con>
Aug 03 18:27:46 vm-etcd1 etcd[2505]: stopped insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of closed network conn>
Aug 03 18:27:46 vm-etcd1 etcd[2505]: skipped leadership transfer for single voting member cluster
Aug 03 18:27:46 vm-etcd1 systemd[1]: Stopping etcd.service - etcd - highly-available key value store...
Aug 03 18:27:46 vm-etcd1 systemd[1]: etcd.service: Deactivated successfully.
Aug 03 18:27:46 vm-etcd1 systemd[1]: Stopped etcd.service - etcd - highly-available key value store.
Aug 03 18:27:46 vm-etcd1 systemd[1]: etcd.service: Consumed 1.594s CPU time.
asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ sudo systemctl stop etcd
asvpg@vm-etcd2:~$ sudo systemctl status etcd
○ etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: inactive (dead) since Mon 2026-08-03 18:39:50 UTC; 10s ago
   Duration: 1min 14.728s
       Docs: https://etcd.io/docs
             man:etcd
    Process: 2456 ExecStart=/usr/bin/etcd $DAEMON_ARGS (code=killed, signal=TERM)
   Main PID: 2456 (code=killed, signal=TERM)
        CPU: 271ms

Aug 03 18:38:36 vm-etcd2 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 03 18:38:36 vm-etcd2 etcd[2456]: set the initial cluster version to 3.4
Aug 03 18:38:36 vm-etcd2 etcd[2456]: enabled capabilities for version 3.4
Aug 03 18:39:50 vm-etcd2 etcd[2456]: received terminated signal, shutting down...
Aug 03 18:39:50 vm-etcd2 systemd[1]: Stopping etcd.service - etcd - highly-available key value store...
Aug 03 18:39:50 vm-etcd2 etcd[2456]: stopping insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of clos>
Aug 03 18:39:50 vm-etcd2 etcd[2456]: stopped insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of close>
Aug 03 18:39:50 vm-etcd2 etcd[2456]: skipped leadership transfer for single voting member cluster
Aug 03 18:39:50 vm-etcd2 systemd[1]: etcd.service: Deactivated successfully.
Aug 03 18:39:50 vm-etcd2 systemd[1]: Stopped etcd.service - etcd - highly-available key value store.
asvpg@vm-etcd2:~$


asvpg@vm-etcd3:~$ sudo systemctl stop etcd
asvpg@vm-etcd3:~$ sudo systemctl status etcd
○ etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: inactive (dead) since Mon 2026-08-03 18:44:06 UTC; 7s ago
   Duration: 33.932s
       Docs: https://etcd.io/docs
             man:etcd
    Process: 3496 ExecStart=/usr/bin/etcd $DAEMON_ARGS (code=killed, signal=TERM)
   Main PID: 3496 (code=killed, signal=TERM)
        CPU: 131ms

Aug 03 18:43:32 vm-etcd3 etcd[3496]: set the initial cluster version to 3.4
Aug 03 18:43:32 vm-etcd3 etcd[3496]: enabled capabilities for version 3.4
Aug 03 18:43:32 vm-etcd3 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 03 18:44:06 vm-etcd3 systemd[1]: Stopping etcd.service - etcd - highly-available key value store...
Aug 03 18:44:06 vm-etcd3 etcd[3496]: received terminated signal, shutting down...
Aug 03 18:44:06 vm-etcd3 etcd[3496]: stopping insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of clos>
Aug 03 18:44:06 vm-etcd3 etcd[3496]: stopped insecure grpc server due to error: accept tcp 127.0.0.1:2379: use of close>
Aug 03 18:44:06 vm-etcd3 etcd[3496]: skipped leadership transfer for single voting member cluster
Aug 03 18:44:06 vm-etcd3 systemd[1]: etcd.service: Deactivated successfully.
Aug 03 18:44:06 vm-etcd3 systemd[1]: Stopped etcd.service - etcd - highly-available key value store.
asvpg@vm-etcd3:~$
```

#####
В Ubuntu с версии 24 команда hostname может выдавать FQDN (полное имя), а не короткое имя.
Если передать FQDN, то не заработает. В нашем случае сразу выдает короткое имя.
Если делаем руками - проблем не будет, если через скрипты с использованием $hostname, нужно проверить.
Проверяем конфиг, пока пустой:
#####
```sh
asvpg@vm-etcd1:~$ hostname
vm-etcd1
asvpg@vm-etcd1:~$ hostname -s
vm-etcd1
asvpg@vm-etcd1:~$ sudo tail /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ hostname
vm-etcd2
asvpg@vm-etcd2:~$ sudo tail /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
asvpg@vm-etcd2:~$


asvpg@vm-etcd3:~$ hostname
vm-etcd3
asvpg@vm-etcd3:~$ sudo tail /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
asvpg@vm-etcd3:~$
```

#####
Настраиваем конфиг etcd на каждой ноде
#####
```sh
--
asvpg@vm-etcd1:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd1:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd1"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.23:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.23:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.23:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.23:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd1:~$



--
asvpg@vm-etcd2:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd2:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd2"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.17:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.17:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.17:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.17:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd2:~$

--
asvpg@vm-etcd3:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd3:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd3"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.11:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.11:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.11:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd3:~$
```

#####
Стартуем каждую ноду кластера etcd и проверяем статус
#####
```sh
asvpg@vm-etcd1:~$ sudo systemctl start etcd
asvpg@vm-etcd1:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-03 19:24:35 UTC; 23s ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 3433 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 12.1M (peak: 12.6M)
        CPU: 497ms
     CGroup: /system.slice/etcd.service
             └─3433 /usr/bin/etcd

Aug 03 19:24:47 vm-etcd1 etcd[3433]: peer e6178adcaa90b752 became active
Aug 03 19:24:47 vm-etcd1 etcd[3433]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message writer)
Aug 03 19:24:47 vm-etcd1 etcd[3433]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2 writer)
Aug 03 19:24:47 vm-etcd1 etcd[3433]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2 reader)
Aug 03 19:24:47 vm-etcd1 etcd[3433]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message reader)
Aug 03 19:24:47 vm-etcd1 etcd[3433]: updating the cluster version from 3.0 to 3.4
Aug 03 19:24:47 vm-etcd1 etcd[3433]: updated the cluster version from 3.0 to 3.4
Aug 03 19:24:47 vm-etcd1 etcd[3433]: enabled capabilities for version 3.4
Aug 03 19:24:48 vm-etcd1 etcd[3433]: health check for peer e6178adcaa90b752 could not connect: dial tcp 10.129.0.11:2380: connect: co>
Aug 03 19:24:48 vm-etcd1 etcd[3433]: health check for peer e6178adcaa90b752 could not connect: dial tcp 10.129.0.11:2380: connect: co>
asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ sudo systemctl start etcd
asvpg@vm-etcd2:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-03 19:24:35 UTC; 1min 4s ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 2941 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 10.1M (peak: 10.6M)
        CPU: 560ms
     CGroup: /system.slice/etcd.service
             └─2941 /usr/bin/etcd

Aug 03 19:24:40 vm-etcd2 etcd[2941]: health check for peer e6178adcaa90b752 could not connect: dial tcp 10.129.0.11:238>
Aug 03 19:24:45 vm-etcd2 etcd[2941]: health check for peer e6178adcaa90b752 could not connect: dial tcp 10.129.0.11:238>
Aug 03 19:24:45 vm-etcd2 etcd[2941]: health check for peer e6178adcaa90b752 could not connect: dial tcp 10.129.0.11:238>
Aug 03 19:24:47 vm-etcd2 etcd[2941]: peer e6178adcaa90b752 became active
Aug 03 19:24:47 vm-etcd2 etcd[2941]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v>
Aug 03 19:24:47 vm-etcd2 etcd[2941]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message >
Aug 03 19:24:47 vm-etcd2 etcd[2941]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v>
Aug 03 19:24:47 vm-etcd2 etcd[2941]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message >
Aug 03 19:24:47 vm-etcd2 etcd[2941]: updated the cluster version from 3.0 to 3.4
Aug 03 19:24:47 vm-etcd2 etcd[2941]: enabled capabilities for version 3.4
asvpg@vm-etcd2:~$


asvpg@vm-etcd3:~$ sudo systemctl start etcd
asvpg@vm-etcd3:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-03 19:24:47 UTC; 1min 21s ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 3929 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 9.7M (peak: 10.2M)
        CPU: 676ms
     CGroup: /system.slice/etcd.service
             └─3929 /usr/bin/etcd

Aug 03 19:24:47 vm-etcd3 etcd[3929]: serving insecure client requests on 10.129.0.11:2379, this is strongly discouraged!
Aug 03 19:24:47 vm-etcd3 etcd[3929]: published {Name:vm-etcd3 ClientURLs:[http://10.129.0.11:2379]} to cluster f30f6076>
Aug 03 19:24:47 vm-etcd3 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 03 19:24:47 vm-etcd3 etcd[3929]: established a TCP streaming connection with peer fb67a1ba60c76013 (stream MsgApp v>
Aug 03 19:24:47 vm-etcd3 etcd[3929]: established a TCP streaming connection with peer fb67a1ba60c76013 (stream Message >
Aug 03 19:24:47 vm-etcd3 etcd[3929]: e6178adcaa90b752 initialized peer connection; fast-forwarding 8 ticks (election ti>
Aug 03 19:24:47 vm-etcd3 etcd[3929]: established a TCP streaming connection with peer 9b8dccf1b7e23a6e (stream MsgApp v>
Aug 03 19:24:47 vm-etcd3 etcd[3929]: established a TCP streaming connection with peer 9b8dccf1b7e23a6e (stream Message >
Aug 03 19:24:47 vm-etcd3 etcd[3929]: updated the cluster version from 3.0 to 3.4
Aug 03 19:24:47 vm-etcd3 etcd[3929]: enabled capabilities for version 3.4
asvpg@vm-etcd3:~$

--Начиная с Ubuntu v24 используется API v3:
asvpg@vm-etcd1:~$ etcdctl endpoint health --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
{"level":"warn","ts":"2026-08-03T19:29:28.452203Z","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc000007880/vm-etcd1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: last connection error: connection error: desc = \"transport: Error while dialing dial tcp 127.0.1.1:2379: connect: connection refused\""}
http://vm-etcd2:2379 is healthy: successfully committed proposal: took = 3.943415ms
http://vm-etcd3:2379 is healthy: successfully committed proposal: took = 4.060009ms
http://vm-etcd1:2379 is unhealthy: failed to commit proposal: context deadline exceeded
Error: unhealthy cluster
asvpg@vm-etcd1:~$
```

####
Видим ошибку unhealthy cluster.
Причина: клиент (etcdctl) пытается разрешить имя vm-etcd1.
Система смотрит в файл /etc/hosts и находит там запись (создается облачным провайдером или установщиком ОС автоматически):
127.0.1.1 vm-etcd1.
Клиент идет по адресу 127.0.1.1. Это адрес локальной петли (loopback), но не тот же самый, что 127.0.0.1 (localhost). На порту 2379 по адресу 127.0.1.1 никто не слушает, поэтому соединение сбрасывается.
При этом узлы vm-etcd2 и vm-etcd3 здоровы, т.к. их имена либо резолвятся через DNS Яндекса во внешние IP, или обращение происходит к ним напрямую, минуя локальный конфликт имен.
Сам сервер etcd на первой ноде работает (иначе кластер бы вообще не собрался), просто клиент обращается 'не туда'.

Посмотрим содержимое файла /etc/hosts:
####
```sh
asvpg@vm-etcd1:~$ cat /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
127.0.1.1 vm-etcd1.ru-central1.internal vm-etcd1
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd1:~$
```

Важно!: в Яндекс Облаке файл /etc/hosts управляется автоматически (об этом говорит заголовок про manage_etc_hosts). 
Нужно отключить автоматическое управление этим файлом, чтобы можно было прописать статические IP-адреса вручную.
В файле /etc/cloud/cloud.cfg настройки нет, выполняем через шаблон, в котором комментируем строку 127.0.1.1 {{fqdn}} {{hostname}}
и прописываем статические IP для каждой ноды кластера etcd, меняем конфиг на каждой из нод кластера etcd:
####
```sh
asvpg@vm-etcd1:~$ cat /etc/cloud/templates/hosts.debian.tmpl
## template:jinja
{#
This file (/etc/cloud/templates/hosts.debian.tmpl) is only utilized
if enabled in cloud-config.  Specifically, in order to enable it
you need to add the following to config:
   manage_etc_hosts: True
-#}
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
{# The value '{{hostname}}' will be replaced with the local-hostname -#}
127.0.1.1 {{fqdn}} {{hostname}}
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd1:~$


asvpg@vm-etcd1:~$ sudo nano /etc/cloud/templates/hosts.debian.tmpl
asvpg@vm-etcd1:~$ cat /etc/cloud/templates/hosts.debian.tmpl
## template:jinja
{#
This file (/etc/cloud/templates/hosts.debian.tmpl) is only utilized
if enabled in cloud-config.  Specifically, in order to enable it
you need to add the following to config:
   manage_etc_hosts: True
-#}
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
{# The value '{{hostname}}' will be replaced with the local-hostname -#}
#127.0.1.1 {{fqdn}} {{hostname}}

10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ sudo nano /etc/cloud/templates/hosts.debian.tmpl
asvpg@vm-etcd2:~$ cat /etc/cloud/templates/hosts.debian.tmpl
## template:jinja
{#
This file (/etc/cloud/templates/hosts.debian.tmpl) is only utilized
if enabled in cloud-config.  Specifically, in order to enable it
you need to add the following to config:
   manage_etc_hosts: True
-#}
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
{# The value '{{hostname}}' will be replaced with the local-hostname -#}
#127.0.1.1 {{fqdn}} {{hostname}}
10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd2:~$



asvpg@vm-etcd3:~$ sudo nano /etc/cloud/templates/hosts.debian.tmpl
asvpg@vm-etcd3:~$ cat /etc/cloud/templates/hosts.debian.tmpl
## template:jinja
{#
This file (/etc/cloud/templates/hosts.debian.tmpl) is only utilized
if enabled in cloud-config.  Specifically, in order to enable it
you need to add the following to config:
   manage_etc_hosts: True
-#}
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
{# The value '{{hostname}}' will be replaced with the local-hostname -#}
#127.0.1.1 {{fqdn}} {{hostname}}
10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd3:~$
```

####
Выполняем рестарт каждой из 3 ВМ и проверяем настройку в /etc/hosts:
####
```sh
asvpg@vm-etcd1:~$ cat /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
#127.0.1.1 vm-etcd1.ru-central1.internal vm-etcd1

10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ cat /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
#127.0.1.1 vm-etcd2.ru-central1.internal vm-etcd2
10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd2:~$



asvpg@vm-etcd3:~$ cat /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
#127.0.1.1 vm-etcd3.ru-central1.internal vm-etcd3
10.129.0.23 vm-etcd1 vm-etcd1.ru-central1.internal
10.129.0.17 vm-etcd2 vm-etcd2.ru-central1.internal
10.129.0.11 vm-etcd3 vm-etcd3.ru-central1.internal
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

asvpg@vm-etcd3:~$
```

###
Далее проверяем статус службы etcd и кластера:
###
```sh
asvpg@vm-etcd1:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-06 19:32:22 UTC; 6min ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 755 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 33.1M (peak: 33.6M)
        CPU: 3.315s
     CGroup: /system.slice/etcd.service
             └─755 /usr/bin/etcd

Aug 06 19:32:22 vm-etcd1 etcd[755]: ready to serve client requests
Aug 06 19:32:22 vm-etcd1 etcd[755]: serving insecure client requests on 127.0.0.1:2379, this is strongly discouraged!
Aug 06 19:32:22 vm-etcd1 etcd[755]: serving insecure client requests on 10.129.0.23:2379, this is strongly discouraged!
Aug 06 19:32:22 vm-etcd1 etcd[755]: published {Name:vm-etcd1 ClientURLs:[http://10.129.0.23:2379]} to cluster f30f6076f>
Aug 06 19:32:22 vm-etcd1 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 06 19:32:23 vm-etcd1 etcd[755]: peer e6178adcaa90b752 became active
Aug 06 19:32:23 vm-etcd1 etcd[755]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message w>
Aug 06 19:32:23 vm-etcd1 etcd[755]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2>
Aug 06 19:32:23 vm-etcd1 etcd[755]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2>
Aug 06 19:32:23 vm-etcd1 etcd[755]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message r>
asvpg@vm-etcd1:~$

svpg@vm-etcd2:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-06 19:32:22 UTC; 7min ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 758 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 50.9M (peak: 51.4M)
        CPU: 4.418s
     CGroup: /system.slice/etcd.service
             └─758 /usr/bin/etcd

Aug 06 19:32:22 vm-etcd2 etcd[758]: serving insecure client requests on 10.129.0.17:2379, this is strongly discouraged!
Aug 06 19:32:22 vm-etcd2 etcd[758]: failed to reach the peerURL(http://10.129.0.11:2380) of member e6178adcaa90b752 (Ge>
Aug 06 19:32:22 vm-etcd2 etcd[758]: cannot get the version of member e6178adcaa90b752 (Get "http://10.129.0.11:2380/ver>
Aug 06 19:32:22 vm-etcd2 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 06 19:32:24 vm-etcd2 etcd[758]: peer e6178adcaa90b752 became active
Aug 06 19:32:24 vm-etcd2 etcd[758]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message r>
Aug 06 19:32:24 vm-etcd2 etcd[758]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2>
Aug 06 19:32:24 vm-etcd2 etcd[758]: established a TCP streaming connection with peer e6178adcaa90b752 (stream MsgApp v2>
Aug 06 19:32:24 vm-etcd2 etcd[758]: established a TCP streaming connection with peer e6178adcaa90b752 (stream Message w>
Aug 06 19:32:24 vm-etcd2 etcd[758]: 9b8dccf1b7e23a6e initialized peer connection; fast-forwarding 8 ticks (election tic>
asvpg@vm-etcd2:~$

asvpg@vm-etcd3:~$ sudo systemctl status etcd.service
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-06 19:32:23 UTC; 8min ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 753 (etcd)
      Tasks: 8 (limit: 2313)
     Memory: 48.9M (peak: 49.4M)
        CPU: 4.198s
     CGroup: /system.slice/etcd.service
             └─753 /usr/bin/etcd

Aug 06 19:32:23 vm-etcd3 etcd[753]: raft2026/08/06 19:32:23 INFO: raft.node: e6178adcaa90b752 elected leader 9b8dccf1b7e23a6e at term>
Aug 06 19:32:23 vm-etcd3 etcd[753]: published {Name:vm-etcd3 ClientURLs:[http://10.129.0.11:2379]} to cluster f30f6076f7a70326
Aug 06 19:32:23 vm-etcd3 etcd[753]: ready to serve client requests
Aug 06 19:32:23 vm-etcd3 etcd[753]: ready to serve client requests
Aug 06 19:32:23 vm-etcd3 etcd[753]: serving insecure client requests on 127.0.0.1:2379, this is strongly discouraged!
Aug 06 19:32:23 vm-etcd3 etcd[753]: serving insecure client requests on 10.129.0.11:2379, this is strongly discouraged!
Aug 06 19:32:23 vm-etcd3 systemd[1]: Started etcd.service - etcd - highly-available key value store.
Aug 06 19:32:23 vm-etcd3 etcd[753]: established a TCP streaming connection with peer fb67a1ba60c76013 (stream MsgApp v2 writer)
Aug 06 19:32:23 vm-etcd3 etcd[753]: established a TCP streaming connection with peer fb67a1ba60c76013 (stream Message writer)
Aug 06 19:32:23 vm-etcd3 etcd[753]: e6178adcaa90b752 initialized peer connection; fast-forwarding 8 ticks (election ticks 10) with 2 >
asvpg@vm-etcd3:~$

asvpg@vm-etcd1:~$ etcdctl endpoint health --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
http://vm-etcd2:2379 is healthy: successfully committed proposal: took = 2.51286ms
http://vm-etcd3:2379 is healthy: successfully committed proposal: took = 2.916398ms
http://vm-etcd1:2379 is healthy: successfully committed proposal: took = 2.847273ms
asvpg@vm-etcd1:~$

asvpg@vm-etcd2:~$ etcdctl endpoint health --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
http://vm-etcd2:2379 is healthy: successfully committed proposal: took = 1.323118ms
http://vm-etcd1:2379 is healthy: successfully committed proposal: took = 5.579086ms
http://vm-etcd3:2379 is healthy: successfully committed proposal: took = 5.669986ms
asvpg@vm-etcd2:~$

asvpg@vm-etcd3:~$ etcdctl endpoint health --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
http://vm-etcd2:2379 is healthy: successfully committed proposal: took = 6.963632ms
http://vm-etcd3:2379 is healthy: successfully committed proposal: took = 11.870338ms
http://vm-etcd1:2379 is healthy: successfully committed proposal: took = 7.032845ms
asvpg@vm-etcd3:~$
```

####
С помощью опции table можно посмотреть информацию в более подробном формате с размером БД, кто лидер итд:
####
```sh
asvpg@vm-etcd1:~$ etcdctl endpoint status -w table --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://vm-etcd1:2379 | fb67a1ba60c76013 |  3.4.30 |   16 kB |     false |      false |        59 |         36 |                 36 |        |
| http://vm-etcd2:2379 | 9b8dccf1b7e23a6e |  3.4.30 |   20 kB |      true |      false |        59 |         36 |                 36 |        |
| http://vm-etcd3:2379 | e6178adcaa90b752 |  3.4.30 |   20 kB |     false |      false |        59 |         36 |                 36 |        |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
asvpg@vm-etcd1:~$
```
####
Несмотря на указанную вторую ноду как лидер, писать и читать можно с любой имеющейся ноды кластера!
Если у ноды статус LEARNER, то она не участвует в выборах лидера, нода изучает\проигрывает логи, т.к. на настоящий момент у нее нет всего объема информации. Если видим этот статус - ничего страшного, через какое-то время нода перейдет в нормальный статус.

После сборки кластера есть рекомендация поменять состояние\статус кластера с NEW на EXISTING:
####
```sh
svpg@vm-etcd1:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd1:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd1"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.23:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.23:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.23:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.23:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd1:~$


asvpg@vm-etcd2:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd2:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd2"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.17:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.17:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.17:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.17:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd2:~$


asvpg@vm-etcd3:~$ sudo nano /etc/default/etcd
asvpg@vm-etcd3:~$ cat /etc/default/etcd
## etcd(1) daemon options
## See "/usr/share/doc/etcd-server/op-guide/configuration.md.gz"
## for available options.
##
## Use environment to override, for example: ETCD_NAME=default
ETCD_NAME="vm-etcd3"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.129.0.11:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.129.0.11:2379"
ETCD_LISTEN_PEER_URLS="http://10.129.0.11:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
ETCD_INITIAL_CLUSTER="vm-etcd1=http://10.129.0.23:2380,vm-etcd2=http://10.129.0.17:2380,vm-etcd3=http://10.129.0.11:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_DATA_DIR="/var/lib/etcd"
asvpg@vm-etcd3:~$
```

####
Хранилище является формата ключ-значение, можно положить в БД, а потом получить. По умолчанию возвращается всегда последнее значение,
но при указании номера ревизию можно просмотреть нужное значение. Система сохраняет всю историю изменений:
####
```sh
asvpg@vm-etcd1:~$ etcdctl put scott tiger
OK
asvpg@vm-etcd1:~$ etcdctl put scott tiger1
OK
asvpg@vm-etcd1:~$ etcdctl get scott
scott
tiger1
asvpg@vm-etcd1:~$ etcdctl put scott tiger2
OK
asvpg@vm-etcd1:~$ etcdctl get scott
scott
tiger2
asvpg@vm-etcd1:~$ etcdctl get --prefix scott
scott
tiger2
asvpg@vm-etcd1:~$ etcdctl get --prefix scott --rev=2
scott
tiger
asvpg@vm-etcd1:~$
```

####
Есть проблематика разрастания размера БД, вследствие чего место рано или поздно закончится.
Отсюда встает вопрос про компактизацию. Согласно документации она выполняется вручную, можно прописать, до какой ревизии можно удалить историю:
####
```sh
asvpg@vm-etcd1:~$ etcdctl get --prefix scott --rev=2
scott
tiger
asvpg@vm-etcd1:~$ etcdctl get --prefix scott --rev=1
{"level":"warn","ts":"2026-08-12T19:45:15.986268Z","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0001076c0/127.0.0.1:2379","attempt":0,"error":"rpc error: code = OutOfRange desc = etcdserver: mvcc: required revision has been compacted"}
Error: etcdserver: mvcc: required revision has been compacted
asvpg@vm-etcd1:~$
```
####
Здесь видим, что вторая ревизия доступна, а первая уже нет.

Для автоматизации компактизации необходимо задать политику удержания (retention policy).
Для начала необходимо проверить, включена ли она\выполнялась ли? есть 3 способа.
####
```sh
--1. Проверить в конфиге, но т.к. не настраивали - ничего не увидим
asvpg@vm-etcd1:~$ cat /etc/default/etcd | grep COMPACTION
asvpg@vm-etcd1:~$
--2. Посмотреть в логах - не получилось, надо понять причину
asvpg@vm-etcd1:~$ journalctl -u etcd | grep -i "compact"
Hint: You are currently not seeing messages from other users and the system.
      Users in groups 'adm', 'systemd-journal' can see all messages.
      Pass -q to turn off this notice.
--3. ЧЕрез метрики - видим информацию:
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ curl -s http://127.0.0.1:2379/metrics | grep etcd_debugging_mvcc_compact_revision
# HELP etcd_debugging_mvcc_compact_revision The revision of the last compaction in store.
# TYPE etcd_debugging_mvcc_compact_revision gauge
etcd_debugging_mvcc_compact_revision 2
asvpg@vm-etcd1:~$
```

####
По умолчанию, максимальный размер БД составляет 2 ГБ. Если явно включена квота, то проверяется через следующую команду:
####
```sh
asvpg@vm-etcd1:~$ ps aux | grep etcd | grep quota
asvpg@vm-etcd1:~$
```
####
В нашем случае квота не задана.
Параметры очистки (компактизации) можно задать через следующие параметры:
####
```sh
ETCD_AUTO_COMPACTION_RETENTION="8h"
ETCD_AUTO_COMPACTION_MODE="periodic"
```
####
Т.е. компактизация будет выполняться с периодичностью через каждые 8 часов.
####
####
Если в БД большие объемы, то необходимо настроить ее дефрагментацию - очистить место от старых и уже удаленных данных.

####
--Есть 2 способа выполнения - локальный, на конкретной ноде:
```sh
asvpg@vm-etcd1:~$ etcdctl endpoint status -w table --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://vm-etcd1:2379 | fb67a1ba60c76013 |  3.4.30 |   16 kB |     false |      false |        59 |         40 |                 40 |        |
| http://vm-etcd2:2379 | 9b8dccf1b7e23a6e |  3.4.30 |   20 kB |      true |      false |        59 |         40 |                 40 |        |
| http://vm-etcd3:2379 | e6178adcaa90b752 |  3.4.30 |   20 kB |     false |      false |        59 |         40 |                 40 |        |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
asvpg@vm-etcd1:~$ etcdctl defrag
Finished defragmenting etcd member[127.0.0.1:2379]
asvpg@vm-etcd1:~$ etcdctl endpoint status -w table --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://vm-etcd1:2379 | fb67a1ba60c76013 |  3.4.30 |   20 kB |     false |      false |        59 |         40 |                 40 |        |
| http://vm-etcd2:2379 | 9b8dccf1b7e23a6e |  3.4.30 |   20 kB |      true |      false |        59 |         40 |                 40 |        |
| http://vm-etcd3:2379 | e6178adcaa90b752 |  3.4.30 |   20 kB |     false |      false |        59 |         40 |                 40 |        |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

--На уровне кластера одной командой. Если бд большая, то возможны зависания, блокировки, перевыборы лидера, поэтому есть рекомендация делать вручную на каждой ноде кластера
asvpg@vm-etcd1:~$ etcdctl defrag --cluster
Finished defragmenting etcd member[http://10.129.0.17:2379]
Finished defragmenting etcd member[http://10.129.0.11:2379]
Finished defragmenting etcd member[http://10.129.0.23:2379]
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ etcdctl endpoint status -w table --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://vm-etcd1:2379 | fb67a1ba60c76013 |  3.4.30 |   20 kB |     false |      false |        59 |         40 |                 40 |        |
| http://vm-etcd2:2379 | 9b8dccf1b7e23a6e |  3.4.30 |   20 kB |      true |      false |        59 |         40 |                 40 |        |
| http://vm-etcd3:2379 | e6178adcaa90b752 |  3.4.30 |   20 kB |     false |      false |        59 |         40 |                 40 |        |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
asvpg@vm-etcd1:~$
```

####
Также важным вопросом является отказоустойчивость.
При 3 нодах кворум - 2. Т.е. при отказе одной ноды кластер продолжит работать.
Важно помнить, что удаление нод возможно только при живом кластере! Самый простой вариант - поднять любой etcd, добавить новую ноду, а старую отключить. Из 2 нод кластер - не отказоустойчив! Если нода 'отвалилась', но при этом физически жива, то стартуем ее, кластер должен собраться и синхронизироваться. Если нода физически не жива, тогда ее необходимо исключить из кластера и добавить новую ноду и при остановленном сервисе прописать изменения в конфиге.

####
```sh
asvpg@vm-etcd1:~$ etcdctl member list
9b8dccf1b7e23a6e, started, vm-etcd2, http://10.129.0.17:2380, http://10.129.0.17:2379, false
e6178adcaa90b752, started, vm-etcd3, http://10.129.0.11:2380, http://10.129.0.11:2379, false
fb67a1ba60c76013, started, vm-etcd1, http://10.129.0.23:2380, http://10.129.0.23:2379, false
asvpg@vm-etcd1:~$
```
####
После таких изменений требуется рестарт сервиса etcd с последующей проверкой работоспособности:
####
```sh
asvpg@vm-etcd1:~$ sudo systemctl restart etcd
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ etcdctl endpoint health --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
http://vm-etcd1:2379 is healthy: successfully committed proposal: took = 2.686182ms
http://vm-etcd3:2379 is healthy: successfully committed proposal: took = 2.951778ms
http://vm-etcd2:2379 is healthy: successfully committed proposal: took = 2.601351ms
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ etcdctl endpoint status -w table --endpoints="http://vm-etcd1:2379,http://vm-etcd2:2379,http://vm-etcd3:2379"
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://vm-etcd1:2379 | fb67a1ba60c76013 |  3.4.30 |   20 kB |      true |      false |        61 |         45 |                 45 |        |
| http://vm-etcd2:2379 | 9b8dccf1b7e23a6e |  3.4.30 |   20 kB |     false |      false |        61 |         45 |                 45 |        |
| http://vm-etcd3:2379 | e6178adcaa90b752 |  3.4.30 |   20 kB |     false |      false |        61 |         45 |                 45 |        |
+----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
asvpg@vm-etcd1:~$
asvpg@vm-etcd1:~$ etcdctl member list
9b8dccf1b7e23a6e, started, vm-etcd2, http://10.129.0.17:2380, http://10.129.0.17:2379, false
e6178adcaa90b752, started, vm-etcd3, http://10.129.0.11:2380, http://10.129.0.11:2379, false
fb67a1ba60c76013, started, vm-etcd1, http://10.129.0.23:2380, http://10.129.0.23:2379, false
asvpg@vm-etcd1:~$
```
####
На этом настройку кластера etcd считаем завершенной, переходим к настройке кластера PostgreSQL
####



###
2. Настройка кластера PostgreSQL
###
####
Задача собрать кластер PostgreSQL из 3 нод (один мастер и две реплики). В качестве источника ВМ используется Yandex Cloud. Настройки по созданию ВМ аналогичны настройкам ВМ кластера etcd, детально создание ВМ для кластера PostgreSQL не рассматривается.
ВМ создаем в другом ЦОДе (ru-central1-d), с теми же системными характеристиками, что и для кластера etcd.

Созданы 3 ВМ со следующими названиями и внутренними IP:
####
```sh
vm-pg1  10.130.0.13
vm-pg2  10.130.0.28
vm-pg3  10.130.0.33
```

####
Далее установим необходимое ПО для дальнейшей настройки кластера PostgreSQL.
Пошаговое описание будет дано для одной ВМ, на остальных всё делалось по аналогии:
####
```sh
--обновляем локальные списки пакетов (package lists) из всех настроенных репозиториев.
asvpg@vm-pg1:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Get:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease [126 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease [126 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Packages [1191 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main Translation-en [282 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Components [180 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Packages [1683 kB]
Get:8 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/universe Translation-en [335 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Components [388 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/restricted amd64 Packages [1424 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/restricted Translation-en [323 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/multiverse Translation-en [12.6 kB]
Get:14 http://mirror.yandex.ru/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:15 http://mirror.yandex.ru/ubuntu noble-backports/main amd64 Components [5776 B]
Get:16 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Components [12.6 kB]
Get:17 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [934 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [202 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [46.4 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1201 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [240 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [76.2 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [1329 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [304 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [10.9 kB]
Fetched 10.6 MB in 2s (4777 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
asvpg@vm-pg1:~$

--устанавливаем последние доступные версии всех установленных в системе пакетов. Флаг -y автоматически отвечает «да» на все вопросы установщика (например, подтверждение дискового пространства), а флаг -q включает тихий режим, убирая лишние строки вывода. Это гарантирует, что базовая ОС актуальна и совместима с новыми библиотеками PostgreSQL.

asvpg@vm-pg1:~$ sudo apt upgrade -y -q
Reading package lists...
Building dependency tree...
Reading state information...
Calculating upgrade...
The following upgrades have been deferred due to phasing:
  krb5-locales libgssapi-krb5-2 libk5crypto3 libkrb5-3 libkrb5support0
The following packages have been kept back:
  google-compute-engine-oslogin
The following packages will be upgraded:
  libnss-systemd libpam-systemd libsystemd-shared libsystemd0 libudev1 systemd systemd-dev systemd-resolved
  systemd-sysv systemd-timesyncd udev
11 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
11 standard LTS security updates
Need to get 8882 kB of archives.
After this operation, 18.4 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libnss-systemd amd64 255.4-1ubuntu8.17 [159 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-dev all 255.4-1ubuntu8.17 [107 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-timesyncd amd64 255.4-1ubuntu8.17 [35.3 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-resolved amd64 255.4-1ubuntu8.17 [297 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd-shared amd64 255.4-1ubuntu8.17 [2077 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd0 amd64 255.4-1ubuntu8.17 [432 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-sysv amd64 255.4-1ubuntu8.17 [11.9 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libpam-systemd amd64 255.4-1ubuntu8.17 [235 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd amd64 255.4-1ubuntu8.17 [3475 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 udev amd64 255.4-1ubuntu8.17 [1875 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libudev1 amd64 255.4-1ubuntu8.17 [178 kB]
Fetched 8882 kB in 0s (95.2 MB/s)
(Reading database ... 106575 files and directories currently installed.)
Preparing to unpack .../0-libnss-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libnss-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../1-systemd-dev_255.4-1ubuntu8.17_all.deb ...
Unpacking systemd-dev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../2-systemd-timesyncd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-timesyncd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../3-systemd-resolved_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-resolved (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../4-libsystemd-shared_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd-shared:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../5-libsystemd0_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd0:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libsystemd0:amd64 (255.4-1ubuntu8.17) ...
(Reading database ... 106575 files and directories currently installed.)
Preparing to unpack .../systemd-sysv_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-sysv (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libpam-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libpam-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../udev_255.4-1ubuntu8.17_amd64.deb ...
Unpacking udev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libudev1_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libudev1:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libudev1:amd64 (255.4-1ubuntu8.17) ...
Setting up systemd-dev (255.4-1ubuntu8.17) ...
Setting up libsystemd-shared:amd64 (255.4-1ubuntu8.17) ...
Setting up systemd (255.4-1ubuntu8.17) ...
Setting up systemd-timesyncd (255.4-1ubuntu8.17) ...
Setting up udev (255.4-1ubuntu8.17) ...
Setting up systemd-resolved (255.4-1ubuntu8.17) ...
Setting up systemd-sysv (255.4-1ubuntu8.17) ...
Setting up libnss-systemd:amd64 (255.4-1ubuntu8.17) ...
Setting up libpam-systemd:amd64 (255.4-1ubuntu8.17) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for initramfs-tools (0.142ubuntu25.8) ...
update-initramfs: Generating /boot/initrd.img-6.8.0-137-generic
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart multipathd.service packagekit.service polkit.service rsyslog.service ssh.service udisks2.service

Service restarts being deferred:
 systemctl restart ModemManager.service
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: apt[1661], sshd[1080]
 asvpg @ user manager service: systemd[1090]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg1:~$


--Добавление официального репозитория PostgreSQL 17 (принято решение поставить уже проверенное временем ядро СУБД).
Версии СУБД в стандартных репозиториях Linux-дистрибутивов часто сильно отстают от актуальных. Для получения свежей стабильной ветки (в данном случае — 17-й) официальный сайт Postgres рекомендует подключить собственный репозиторий APT.

Эта команда формирует строку конфигурации репозитория и записывает её в отдельный файл внутри директории /etc/apt/sources.list.d/. 

$(lsb_release -cs) — подстановка короткого названия вашей операционной системы (например, ubuntu24.04 итд). Скрипт динамически определяет нужную папку с пакетами на сервере разработчиков.
tee позволяет записать вывод echo в файл от имени суперпользователя (sudo), так как обычные пользователи не имеют прав изменять системные конфиги.

asvpg@vm-pg1:~$ echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
deb http://apt.postgresql.org/pub/repos/apt noble-pgdg main
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$ cd /etc/apt/sources.list.d/
asvpg@vm-pg1:/etc/apt/sources.list.d$ ls -altr
total 16
drwxr-xr-x 8 root root 4096 Jun  5  2025 ..
-rw-r--r-- 1 root root 2987 Aug 13 18:55 ubuntu.sources
-rw-r--r-- 1 root root   60 Aug 13 19:13 pgdg.list
drwxr-xr-x 2 root root 4096 Aug 13 19:13 .
asvpg@vm-pg1:/etc/apt/sources.list.d$ cat pgdg.list
deb http://apt.postgresql.org/pub/repos/apt noble-pgdg main
asvpg@vm-pg1:/etc/apt/sources.list.d$


--Импорт ключа подписи репозитория.
Система управления пакетами должна убедиться, что пакеты приходят именно от официального сообщества PostgreSQL, а не были подменены злоумышленником.

wget --quiet -O - скачивает публичный PGP-ключ по ссылке. Флаг --quiet скрывает лог загрузки, а -O - выводит содержимое файла прямо в консоль (стандартный поток вывода).
Символ трубы | передает этот ключ на вход следующей команде.
sudo apt-key add - принимает ключ со входа (на что указывает дефис -) и добавляет его в доверенное хранилище ключей текущего сервера. После этого система будет доверять пакетам из добавленного выше репозитория.

Утилита apt-key объявлена устаревшей во многих современных дистрибутивах. Более правильным подходом является сохранение ключа в формате .gpg в директорию /usr/share/keyrings/ и указание параметра signed-by в файле репозитория. Однако классический метод через apt-key add - всё ещё работает.

asvpg@vm-pg1:/etc/apt/sources.list.d$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
asvpg@vm-pg1:/etc/apt/sources.list.d$



--Синхронизация списков пакетов.
Заново обновляет списки пакетов, но теперь учитывает ранее добавленный репозиторий pgdg. Если пропустить эту команду, на следующем шаге система попытается установить PostgreSQL из стандартного репозитория Яндекса (где версия почти наверняка будет старее 17-й) или выдаст ошибку, что пакета не существует.

asvpg@vm-pg1:/etc/apt/sources.list.d$ sudo apt-get update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Hit:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease
Get:4 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease [189 kB]
Hit:5 http://security.ubuntu.com/ubuntu noble-security InRelease
Get:6 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 Packages [790 kB]
Fetched 979 kB in 1s (1277 kB/s)
Reading package lists... Done
W: http://apt.postgresql.org/pub/repos/apt/dists/noble-pgdg/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
asvpg@vm-pg1:/etc/apt/sources.list.d$



--Установка СУБД PostgreSQL 17.
Загружает и устанавливает метапакет postgresql-17. Вместе с ним будут установлены:

Основной сервер базы данных;
Утилиты клиента (psql);
Дополнительные contrib-модули;
Пользовательская учетная запись postgres в операционной системе, от имени которой запускаются процессы БД.

Менеджер пакетов автоматически создаст кластер базы данных с настройками по умолчанию, выполнит первичную инициализацию каталога данных (по умолчанию /var/lib/postgresql/17/main) и запустит фоновый процесс сервера. СУБД начнет слушать порт 5432 исключительно на локальном интерфейсе (localhost).

asvpg@vm-pg1:/etc/apt/sources.list.d$ sudo apt -y install postgresql-17
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm19 libpq5 libtypes-serialiser-perl postgresql-client-17
  postgresql-client-common postgresql-common ssl-cert
Suggested packages:
  libpq-oauth postgresql-doc-17
The following NEW packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm19 libpq5 libtypes-serialiser-perl postgresql-17
  postgresql-client-17 postgresql-client-common postgresql-common ssl-cert
0 upgraded, 11 newly installed, 0 to remove and 6 not upgraded.
Need to get 48.4 MB of archives.
After this operation, 201 MB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 libjson-perl all 4.10000-1 [81.9 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 ssl-cert all 1.1.2ubuntu1 [17.8 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 libcommon-sense-perl amd64 3.75-3build3 [20.4 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble/main amd64 libtypes-serialiser-perl all 1.01-1 [11.6 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libjson-xs-perl amd64 4.040-0ubuntu0.24.04.1 [83.7 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libllvm19 amd64 1:19.1.1-1ubuntu1~24.04.2 [28.7 MB]
Get:7 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-client-common all 293.pgdg24.04+1 [48.5 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-common all 293.pgdg24.04+1 [113 kB]
Get:9 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 libpq5 amd64 18.6-1.pgdg24.04+2 [264 kB]
Get:10 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-client-17 amd64 17.11-1.pgdg24.04+2 [2053 kB]
Get:11 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-17 amd64 17.11-1.pgdg24.04+2 [16.9 MB]
Fetched 48.4 MB in 1s (77.7 MB/s)
Preconfiguring packages ...
Selecting previously unselected package libjson-perl.
(Reading database ... 106575 files and directories currently installed.)
Preparing to unpack .../00-libjson-perl_4.10000-1_all.deb ...
Unpacking libjson-perl (4.10000-1) ...
Selecting previously unselected package postgresql-client-common.
Preparing to unpack .../01-postgresql-client-common_293.pgdg24.04+1_all.deb ...
Unpacking postgresql-client-common (293.pgdg24.04+1) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../02-ssl-cert_1.1.2ubuntu1_all.deb ...
Unpacking ssl-cert (1.1.2ubuntu1) ...
Selecting previously unselected package postgresql-common.
Preparing to unpack .../03-postgresql-common_293.pgdg24.04+1_all.deb ...
Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
Unpacking postgresql-common (293.pgdg24.04+1) ...
Selecting previously unselected package libcommon-sense-perl:amd64.
Preparing to unpack .../04-libcommon-sense-perl_3.75-3build3_amd64.deb ...
Unpacking libcommon-sense-perl:amd64 (3.75-3build3) ...
Selecting previously unselected package libtypes-serialiser-perl.
Preparing to unpack .../05-libtypes-serialiser-perl_1.01-1_all.deb ...
Unpacking libtypes-serialiser-perl (1.01-1) ...
Selecting previously unselected package libjson-xs-perl.
Preparing to unpack .../06-libjson-xs-perl_4.040-0ubuntu0.24.04.1_amd64.deb ...
Unpacking libjson-xs-perl (4.040-0ubuntu0.24.04.1) ...
Selecting previously unselected package libllvm19:amd64.
Preparing to unpack .../07-libllvm19_1%3a19.1.1-1ubuntu1~24.04.2_amd64.deb ...
Unpacking libllvm19:amd64 (1:19.1.1-1ubuntu1~24.04.2) ...
Selecting previously unselected package libpq5:amd64.
Preparing to unpack .../08-libpq5_18.6-1.pgdg24.04+2_amd64.deb ...
Unpacking libpq5:amd64 (18.6-1.pgdg24.04+2) ...
Selecting previously unselected package postgresql-client-17.
Preparing to unpack .../09-postgresql-client-17_17.11-1.pgdg24.04+2_amd64.deb ...
Unpacking postgresql-client-17 (17.11-1.pgdg24.04+2) ...
Selecting previously unselected package postgresql-17.
Preparing to unpack .../10-postgresql-17_17.11-1.pgdg24.04+2_amd64.deb ...
Unpacking postgresql-17 (17.11-1.pgdg24.04+2) ...
Setting up postgresql-client-common (293.pgdg24.04+1) ...
Setting up libllvm19:amd64 (1:19.1.1-1ubuntu1~24.04.2) ...
Setting up libpq5:amd64 (18.6-1.pgdg24.04+2) ...
Setting up libcommon-sense-perl:amd64 (3.75-3build3) ...
Setting up ssl-cert (1.1.2ubuntu1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/ssl-cert.service → /usr/lib/systemd/system/ssl-cert.service.
Setting up libtypes-serialiser-perl (1.01-1) ...
Setting up libjson-perl (4.10000-1) ...
Setting up libjson-xs-perl (4.040-0ubuntu0.24.04.1) ...
Setting up postgresql-client-17 (17.11-1.pgdg24.04+2) ...
update-alternatives: using /usr/share/postgresql/17/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
Setting up postgresql-common (293.pgdg24.04+1) ...

Creating config file /etc/postgresql-common/createcluster.conf with new version
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
Removing obsolete dictionary files:
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
Setting up postgresql-17 (17.11-1.pgdg24.04+2) ...
Creating new PostgreSQL cluster 17/main ...
/usr/lib/postgresql/17/bin/initdb -D /var/lib/postgresql/17/main --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "C.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/17/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default "max_connections" ... 100
selecting default "shared_buffers" ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1080]
 asvpg @ user manager service: systemd[1090]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg1:/etc/apt/sources.list.d$
```

####
Далее проверяем, что кластер PostgreSQL создан и стартован на каждой ноде (по умолчанию используется порт 5432):
####
```sh
asvpg@vm-pg1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg1:~$


asvpg@vm-pg2:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg2:~$


asvpg@vm-pg3:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg3:~$
```

####
На первой ноде (vm-pg1) установим тестовую БД на 6 млн строк:
####
```sh
asvpg@vm-pg1:~$ sudo su postgres
postgres@vm-pg1:/home/asvpg$ cd ~ && wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
--2026-08-13 19:47:48--  https://storage.googleapis.com/thaibus/thai_small.tar.gz
Resolving storage.googleapis.com (storage.googleapis.com)... 74.125.131.207, 64.233.164.207, 64.233.165.207, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|74.125.131.207|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 84252589 (80M) [application/x-gzip]
Saving to: ‘thai_small.tar.gz’

thai_small.tar.gz             100%[=================================================>]  80.35M  23.7MB/s    in 3.7s

2026-08-13 19:47:52 (21.7 MB/s) - ‘thai_small.tar.gz’ saved [84252589/84252589]

SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
CREATE DATABASE
ALTER DATABASE
You are now connected to database "thai" as user "postgres".
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
CREATE SCHEMA
ALTER SCHEMA
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 60
COPY 10
COPY 111
COPY 109
COPY 144000
COPY 1440
COPY 200
COPY 3
COPY 5185505
 setval
--------
      1
(1 row)

 setval
--------
     60
(1 row)

 setval
--------
      1
(1 row)

 setval
--------
 144000
(1 row)

 setval
--------
   1440
(1 row)

 setval
--------
    200
(1 row)

 setval
--------
      1
(1 row)

 setval
---------
 5185505
(1 row)

ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
postgres@vm-pg1:~$
```

####
Далее создаем реплики. Для этого используем отдельного пользователя repl_user с атрибутом replication. Выполнение настроек под системным пользователем postgres считается неправильным.
####
```sh
postgres@vm-pg1:~$ psql -c "create user repl_user with replication login password 'repl_user';"
CREATE ROLE
postgres@vm-pg1:~$
```

####
Останавливаем 2 и 3 ноды БД
####
```sh
asvpg@vm-pg2:~$ sudo pg_ctlcluster 17 main stop
asvpg@vm-pg2:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 down   postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo pg_ctlcluster 17 main stop
asvpg@vm-pg3:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 down   postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg3:~$
```
####
Удаляем каталог с данными
####
```sh
postgres@vm-pg2:~$ ls -altr /var/lib/postgresql/17/main
total 88
drwxr-xr-x  3 postgres postgres 4096 Aug 13 19:36 ..
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_twophase
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_tblspc
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_snapshots
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_serial
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_replslot
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_notify
drwx------  4 postgres postgres 4096 Aug 13 19:36 pg_multixact
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_dynshmem
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_commit_ts
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_stat_tmp
-rw-------  1 postgres postgres    3 Aug 13 19:36 PG_VERSION
-rw-------  1 postgres postgres   88 Aug 13 19:36 postgresql.auto.conf
drwx------  4 postgres postgres 4096 Aug 13 19:36 pg_wal
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_xact
drwx------  2 postgres postgres 4096 Aug 13 19:36 pg_subtrans
drwx------  5 postgres postgres 4096 Aug 13 19:36 base
-rw-------  1 postgres postgres  130 Aug 15 13:28 postmaster.opts
drwx------  2 postgres postgres 4096 Aug 15 13:28 global
drwx------  4 postgres postgres 4096 Aug 15 13:41 pg_logical
drwx------  2 postgres postgres 4096 Aug 15 13:41 pg_stat
drwx------ 19 postgres postgres 4096 Aug 15 13:41 .
postgres@vm-pg2:~$ rm -rf /var/lib/postgresql/17/main
postgres@vm-pg2:~$ ls -altr /var/lib/postgresql/17/main
ls: cannot access '/var/lib/postgresql/17/main': No such file or directory
postgres@vm-pg2:~$


postgres@vm-pg3:~$ ls -altr /var/lib/postgresql/17/main
total 88
drwxr-xr-x  3 postgres postgres 4096 Aug 13 19:25 ..
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_twophase
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_tblspc
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_stat_tmp
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_snapshots
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_serial
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_replslot
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_notify
drwx------  4 postgres postgres 4096 Aug 13 19:25 pg_multixact
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_dynshmem
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_commit_ts
-rw-------  1 postgres postgres    3 Aug 13 19:25 PG_VERSION
-rw-------  1 postgres postgres   88 Aug 13 19:25 postgresql.auto.conf
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_xact
drwx------  4 postgres postgres 4096 Aug 13 19:25 pg_wal
drwx------  2 postgres postgres 4096 Aug 13 19:25 pg_subtrans
drwx------  5 postgres postgres 4096 Aug 13 19:25 base
-rw-------  1 postgres postgres  130 Aug 15 13:28 postmaster.opts
drwx------  2 postgres postgres 4096 Aug 15 13:28 global
drwx------  4 postgres postgres 4096 Aug 15 13:42 pg_logical
drwx------  2 postgres postgres 4096 Aug 15 13:42 pg_stat
drwx------ 19 postgres postgres 4096 Aug 15 13:42 .
postgres@vm-pg3:~$
postgres@vm-pg3:~$ rm -rf /var/lib/postgresql/17/main
postgres@vm-pg3:~$
postgres@vm-pg3:~$ ls -altr /var/lib/postgresql/17/main
ls: cannot access '/var/lib/postgresql/17/main': No such file or directory
postgres@vm-pg3:~$
```
####
Далее настраиваем сетевой доступ к БД через конфиг pg_hba (файервол). Также настраиваем листенер (по умолчанию не разрешает доступ).
На первой ноде разрешим сетевой доступ для бэкапа. Указываем всегда только внутренние IP адрес (открываем только внутреннюю сеть), не внешние!
Для листенера прописываем localhost (иначе не сможем подключиться с локального хоста) и внутренний адрес ноды. * не ставим, иначе внешние IP также будут доступны.
####
```sh
postgres@vm-pg1:~$ nano /etc/postgresql/17/main/pg_hba.conf
postgres@vm-pg1:~$ cat /etc/postgresql/17/main/pg_hba.conf
# PostgreSQL Client Authentication Configuration File
# ===================================================
...
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             10.0.0.0/8            scram-sha-256  --!!!
host    replication     all             ::1/128                 scram-sha-256
postgres@vm-pg1:~$

postgres@vm-pg1:~$ nano /etc/postgresql/17/main/postgresql.conf
postgres@vm-pg1:~$ cat /etc/postgresql/17/main/postgresql.conf
# -----------------------------
# PostgreSQL configuration file
# -----------------------------
#
# This file consists of lines of the form:
#
#   name = value
#
...
#------------------------------------------------------------------------------
# FILE LOCATIONS
#------------------------------------------------------------------------------

# The default values of these variables are driven from the -D command-line
# option or PGDATA environment variable, represented here as ConfigDir.

data_directory = '/var/lib/postgresql/17/main'          # use data in another directory
                                        # (change requires restart)
hba_file = '/etc/postgresql/17/main/pg_hba.conf'        # host-based authentication file
                                        # (change requires restart)
ident_file = '/etc/postgresql/17/main/pg_ident.conf'    # ident configuration file
                                        # (change requires restart)

# If external_pid_file is not explicitly set, no extra PID file is written.
external_pid_file = '/var/run/postgresql/17-main.pid'                   # write an extra PID file
                                        # (change requires restart)


#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = 'localhost, 10.130.0.13'             # what IP address(es) to listen on; --!!!
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)
```
####
Изменения в ph_hba требуют перезагрузки конфигурации, а листенер - рестарт кластера БД (через стоп и старт)
####
```sh
postgres@vm-pg1:~$ pg_ctlcluster stop 17 main && pg_ctlcluster start 17 main
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@17-main
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@17-main
postgres@vm-pg1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
postgres@vm-pg1:~$

postgres@vm-pg1:~$ psql
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

postgres=# show listen_addresses;
    listen_addresses
------------------------
 localhost, 10.130.0.13
(1 row)

postgres=#
```

####
Первая нода PG готова к снятию реплики.
На второй и третьем нодах выполняем подключение к первой ноде и для снятия бэкапа через утилиту pg_basebackup без создания слота.
Команду необходимо выполнять обязательно под пользователем postgres, а не через sudo -u, иначе будут присвоены некорректные права root,
после чего будут ошибки в работе.
Указываем пароль ранее созданного пользователя repl_user. Система может ожидать прихода checkpoint. Для ускорения можно выполнить его вручную рна первой ноде. Время выполнения физического бэкапа намного меньше, чем заливка через дамп (copy).
####
```sh
postgres@vm-pg2:~$ pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
17  main    5432 down   <unknown> /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
postgres@vm-pg2:~$
postgres@vm-pg2:~$ pg_basebackup -R -h 10.130.0.13 -D /var/lib/postgresql/17/main -P -U repl_user
Password:
626636/626636 kB (100%), 1/1 tablespace
postgres@vm-pg2:~$
postgres@vm-pg2:~$ ls -altr /var/lib/postgresql/17/main
total 272
drwxr-xr-x  3 postgres postgres   4096 Aug 15 14:16 ..
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_xact
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_stat
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_notify
-rw-------  1 postgres postgres    227 Aug 15 14:16 backup_label
drwx------  4 postgres postgres   4096 Aug 15 14:16 pg_wal
drwx------  4 postgres postgres   4096 Aug 15 14:16 pg_logical
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_commit_ts
-rw-------  1 postgres postgres      3 Aug 15 14:16 PG_VERSION
drwx------  4 postgres postgres   4096 Aug 15 14:16 pg_multixact
-rw-------  1 postgres postgres    415 Aug 15 14:16 postgresql.auto.conf
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_tblspc
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_subtrans
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_stat_tmp
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_serial
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_replslot
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_dynshmem
drwx------  6 postgres postgres   4096 Aug 15 14:16 base
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_twophase
drwx------  2 postgres postgres   4096 Aug 15 14:16 pg_snapshots
drwx------  2 postgres postgres   4096 Aug 15 14:16 global
-rw-------  1 postgres postgres      0 Aug 15 14:16 standby.signal
-rw-------  1 postgres postgres 187039 Aug 15 14:16 backup_manifest
drwx------ 19 postgres postgres   4096 Aug 15 14:17 .
postgres@vm-pg2:~$

postgres@vm-pg3:~$ pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
17  main    5432 down   <unknown> /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
postgres@vm-pg3:~$
postgres@vm-pg3:~$ pg_basebackup -R -h 10.130.0.13 -D /var/lib/postgresql/17/main -P -U repl_user
Password:
626636/626636 kB (100%), 1/1 tablespace
postgres@vm-pg3:~$
postgres@vm-pg3:~$ ls -altr /var/lib/postgresql/17/main
total 272
drwxr-xr-x  3 postgres postgres   4096 Aug 15 14:23 ..
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_stat
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_notify
-rw-------  1 postgres postgres    227 Aug 15 14:23 backup_label
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_xact
drwx------  4 postgres postgres   4096 Aug 15 14:23 pg_wal
drwx------  4 postgres postgres   4096 Aug 15 14:23 pg_logical
-rw-------  1 postgres postgres    415 Aug 15 14:23 postgresql.auto.conf
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_tblspc
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_subtrans
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_stat_tmp
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_serial
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_replslot
drwx------  4 postgres postgres   4096 Aug 15 14:23 pg_multixact
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_dynshmem
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_commit_ts
-rw-------  1 postgres postgres      3 Aug 15 14:23 PG_VERSION
drwx------  6 postgres postgres   4096 Aug 15 14:23 base
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_twophase
drwx------  2 postgres postgres   4096 Aug 15 14:23 pg_snapshots
-rw-------  1 postgres postgres      0 Aug 15 14:23 standby.signal
drwx------  2 postgres postgres   4096 Aug 15 14:23 global
-rw-------  1 postgres postgres 187039 Aug 15 14:23 backup_manifest
drwx------ 19 postgres postgres   4096 Aug 15 14:23 .
postgres@vm-pg3:~$
```

####
Выполняем старт PG на второй и третьей нодах
####
```sh
postgres@vm-pg2:~$ pg_ctlcluster 17 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@17-main
postgres@vm-pg2:~$ pg_lsclusters
Ver Cluster Port Status          Owner    Data directory              Log file
17  main    5432 online,recovery postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
postgres@vm-pg2:~$

postgres@vm-pg3:~$ pg_ctlcluster 17 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@17-main
postgres@vm-pg3:~$ pg_lsclusters
Ver Cluster Port Status          Owner    Data directory              Log file
17  main    5432 online,recovery postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
postgres@vm-pg3:~$
```

####
Настройка кластера завершена: 1 мастер и 2 реплики. Физическая репликация настроена.
####
```sh
postgres@vm-pg1:~$ psql
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

postgres=#

postgres@vm-pg2:~$ psql
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)

postgres=#

postgres@vm-pg3:~$ psql
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)

postgres=#
```

###
3. Настройка Patroni для автоматического переключения
###

####
Устанавливаем patroni из пакетов на каждой из трех нод. Если требуется конкретная версия, то можно поставить из python, с использованием pip
####
```sh
asvpg@vm-pg1:~$ sudo apt install patroni -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2 python3-wcwidth
Suggested packages:
  postgresql etcd-server | consul | zookeeperd vip-manager haproxy patroni-doc python3-trio python3-aioquic python3-h2
  python3-httpx python3-httpcore etcd python-psycopg2-doc
The following NEW packages will be installed:
  patroni python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2
  python3-wcwidth
0 upgraded, 8 newly installed, 0 to remove and 6 not upgraded.
Need to get 877 kB of archives.
After this operation, 5469 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-wcwidth all 0.2.5+dfsg1-1.1ubuntu1 [22.5 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-prettytable all 3.6.0-2 [32.8 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-psutil amd64 5.9.8-2build2 [195 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-dnspython all 2.6.1-1ubuntu1 [163 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-etcd all 0.4.5-4 [31.9 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-cdiff all 1.0-1.1 [16.4 kB]
Get:7 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 python3-psycopg2 amd64 2.9.10-1.pgdg24.04+1 [123 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 patroni all 4.1.5-1.pgdg24.04+1 [291 kB]
Fetched 877 kB in 0s (2859 kB/s)
Selecting previously unselected package python3-wcwidth.
(Reading database ... 108856 files and directories currently installed.)
Preparing to unpack .../0-python3-wcwidth_0.2.5+dfsg1-1.1ubuntu1_all.deb ...
Unpacking python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Selecting previously unselected package python3-prettytable.
Preparing to unpack .../1-python3-prettytable_3.6.0-2_all.deb ...
Unpacking python3-prettytable (3.6.0-2) ...
Selecting previously unselected package python3-psutil.
Preparing to unpack .../2-python3-psutil_5.9.8-2build2_amd64.deb ...
Unpacking python3-psutil (5.9.8-2build2) ...
Selecting previously unselected package python3-psycopg2.
Preparing to unpack .../3-python3-psycopg2_2.9.10-1.pgdg24.04+1_amd64.deb ...
Unpacking python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Selecting previously unselected package python3-dnspython.
Preparing to unpack .../4-python3-dnspython_2.6.1-1ubuntu1_all.deb ...
Unpacking python3-dnspython (2.6.1-1ubuntu1) ...
Selecting previously unselected package python3-etcd.
Preparing to unpack .../5-python3-etcd_0.4.5-4_all.deb ...
Unpacking python3-etcd (0.4.5-4) ...
Selecting previously unselected package python3-cdiff.
Preparing to unpack .../6-python3-cdiff_1.0-1.1_all.deb ...
Unpacking python3-cdiff (1.0-1.1) ...
Selecting previously unselected package patroni.
Preparing to unpack .../7-patroni_4.1.5-1.pgdg24.04+1_all.deb ...
Unpacking patroni (4.1.5-1.pgdg24.04+1) ...
Setting up python3-cdiff (1.0-1.1) ...
Setting up python3-psutil (5.9.8-2build2) ...
Setting up python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Setting up python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Setting up python3-dnspython (2.6.1-1ubuntu1) ...
Setting up python3-prettytable (3.6.0-2) ...
Setting up python3-etcd (0.4.5-4) ...
Setting up patroni (4.1.5-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /usr/lib/systemd/system/patroni.service.
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo apt install patroni -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2 python3-wcwidth
Suggested packages:
  postgresql etcd-server | consul | zookeeperd vip-manager haproxy patroni-doc python3-trio python3-aioquic python3-h2
  python3-httpx python3-httpcore etcd python-psycopg2-doc
The following NEW packages will be installed:
  patroni python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2
  python3-wcwidth
0 upgraded, 8 newly installed, 0 to remove and 1 not upgraded.
Need to get 877 kB of archives.
After this operation, 5469 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-wcwidth all 0.2.5+dfsg1-1.1ubuntu1 [22.5 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-prettytable all 3.6.0-2 [32.8 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-psutil amd64 5.9.8-2build2 [195 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-dnspython all 2.6.1-1ubuntu1 [163 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-etcd all 0.4.5-4 [31.9 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-cdiff all 1.0-1.1 [16.4 kB]
Get:7 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 python3-psycopg2 amd64 2.9.10-1.pgdg24.04+1 [123 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 patroni all 4.1.5-1.pgdg24.04+1 [291 kB]
Fetched 877 kB in 0s (4262 kB/s)
Selecting previously unselected package python3-wcwidth.
(Reading database ... 108856 files and directories currently installed.)
Preparing to unpack .../0-python3-wcwidth_0.2.5+dfsg1-1.1ubuntu1_all.deb ...
Unpacking python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Selecting previously unselected package python3-prettytable.
Preparing to unpack .../1-python3-prettytable_3.6.0-2_all.deb ...
Unpacking python3-prettytable (3.6.0-2) ...
Selecting previously unselected package python3-psutil.
Preparing to unpack .../2-python3-psutil_5.9.8-2build2_amd64.deb ...
Unpacking python3-psutil (5.9.8-2build2) ...
Selecting previously unselected package python3-psycopg2.
Preparing to unpack .../3-python3-psycopg2_2.9.10-1.pgdg24.04+1_amd64.deb ...
Unpacking python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Selecting previously unselected package python3-dnspython.
Preparing to unpack .../4-python3-dnspython_2.6.1-1ubuntu1_all.deb ...
Unpacking python3-dnspython (2.6.1-1ubuntu1) ...
Selecting previously unselected package python3-etcd.
Preparing to unpack .../5-python3-etcd_0.4.5-4_all.deb ...
Unpacking python3-etcd (0.4.5-4) ...
Selecting previously unselected package python3-cdiff.
Preparing to unpack .../6-python3-cdiff_1.0-1.1_all.deb ...
Unpacking python3-cdiff (1.0-1.1) ...
Selecting previously unselected package patroni.
Preparing to unpack .../7-patroni_4.1.5-1.pgdg24.04+1_all.deb ...
Unpacking patroni (4.1.5-1.pgdg24.04+1) ...
Setting up python3-cdiff (1.0-1.1) ...
Setting up python3-psutil (5.9.8-2build2) ...
Setting up python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Setting up python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Setting up python3-dnspython (2.6.1-1ubuntu1) ...
Setting up python3-prettytable (3.6.0-2) ...
Setting up python3-etcd (0.4.5-4) ...
Setting up patroni (4.1.5-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /usr/lib/systemd/system/patroni.service.
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo apt install patroni -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2 python3-wcwidth
Suggested packages:
  postgresql etcd-server | consul | zookeeperd vip-manager haproxy patroni-doc python3-trio python3-aioquic python3-h2
  python3-httpx python3-httpcore etcd python-psycopg2-doc
The following NEW packages will be installed:
  patroni python3-cdiff python3-dnspython python3-etcd python3-prettytable python3-psutil python3-psycopg2
  python3-wcwidth
0 upgraded, 8 newly installed, 0 to remove and 6 not upgraded.
Need to get 877 kB of archives.
After this operation, 5469 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-wcwidth all 0.2.5+dfsg1-1.1ubuntu1 [22.5 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-prettytable all 3.6.0-2 [32.8 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-psutil amd64 5.9.8-2build2 [195 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble/main amd64 python3-dnspython all 2.6.1-1ubuntu1 [163 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-etcd all 0.4.5-4 [31.9 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble/universe amd64 python3-cdiff all 1.0-1.1 [16.4 kB]
Get:7 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 python3-psycopg2 amd64 2.9.10-1.pgdg24.04+1 [123 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 patroni all 4.1.5-1.pgdg24.04+1 [291 kB]
Fetched 877 kB in 0s (3583 kB/s)
Selecting previously unselected package python3-wcwidth.
(Reading database ... 108856 files and directories currently installed.)
Preparing to unpack .../0-python3-wcwidth_0.2.5+dfsg1-1.1ubuntu1_all.deb ...
Unpacking python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Selecting previously unselected package python3-prettytable.
Preparing to unpack .../1-python3-prettytable_3.6.0-2_all.deb ...
Unpacking python3-prettytable (3.6.0-2) ...
Selecting previously unselected package python3-psutil.
Preparing to unpack .../2-python3-psutil_5.9.8-2build2_amd64.deb ...
Unpacking python3-psutil (5.9.8-2build2) ...
Selecting previously unselected package python3-psycopg2.
Preparing to unpack .../3-python3-psycopg2_2.9.10-1.pgdg24.04+1_amd64.deb ...
Unpacking python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Selecting previously unselected package python3-dnspython.
Preparing to unpack .../4-python3-dnspython_2.6.1-1ubuntu1_all.deb ...
Unpacking python3-dnspython (2.6.1-1ubuntu1) ...
Selecting previously unselected package python3-etcd.
Preparing to unpack .../5-python3-etcd_0.4.5-4_all.deb ...
Unpacking python3-etcd (0.4.5-4) ...
Selecting previously unselected package python3-cdiff.
Preparing to unpack .../6-python3-cdiff_1.0-1.1_all.deb ...
Unpacking python3-cdiff (1.0-1.1) ...
Selecting previously unselected package patroni.
Preparing to unpack .../7-patroni_4.1.5-1.pgdg24.04+1_all.deb ...
Unpacking patroni (4.1.5-1.pgdg24.04+1) ...
Setting up python3-cdiff (1.0-1.1) ...
Setting up python3-psutil (5.9.8-2build2) ...
Setting up python3-wcwidth (0.2.5+dfsg1-1.1ubuntu1) ...
Setting up python3-psycopg2 (2.9.10-1.pgdg24.04+1) ...
Setting up python3-dnspython (2.6.1-1ubuntu1) ...
Setting up python3-prettytable (3.6.0-2) ...
Setting up python3-etcd (0.4.5-4) ...
Setting up patroni (4.1.5-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /usr/lib/systemd/system/patroni.service.
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg3:~$
```

####
Проверяем, что сервис установлен и стартован
####
```sh
asvpg@vm-pg1:~$ sudo systemctl status patroni
○ patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: inactive (dead)
  Condition: start condition unmet at Sat 2026-08-15 14:35:50 UTC; 3min 40s ago
             └─ ConditionPathExists=/etc/patroni/config.yml was not met

Aug 15 14:35:50 vm-pg1 systemd[1]: patroni.service - Runners to orchestrate a high-availability PostgreSQL was skipped >
asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo systemctl status patroni
○ patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: inactive (dead)
  Condition: start condition unmet at Sat 2026-08-15 14:37:19 UTC; 3min 4s ago
             └─ ConditionPathExists=/etc/patroni/config.yml was not met

Aug 15 14:37:19 vm-pg2 systemd[1]: patroni.service - Runners to orchestrate a high-availability PostgreSQL was skipped >
asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo systemctl status patroni
○ patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: inactive (dead)
  Condition: start condition unmet at Sat 2026-08-15 14:38:29 UTC; 2min 16s ago
             └─ ConditionPathExists=/etc/patroni/config.yml was not met

Aug 15 14:38:29 vm-pg3 systemd[1]: patroni.service - Runners to orchestrate a high-availability PostgreSQL was skipped >
asvpg@vm-pg3:~$
```

####
Далее необходимо настроить конфиг Patroni на каждой из 3 нод, в котором указываем 3 версию API у ETCD (etcd3), в строке etcd указываем FQDN (полные имена) нод кластера etcd, т.к. они находятся физически в другом ЦОДе. Если указать через короткие имена, то сам кластер etcd будет работать, но кластер Patroni с ним работать не сможет.

В postgresql в строке листенера (listen) можно указывать 2 адрес (127.0.0.1 и внутренний IP), а в patroni нельзя (только один).

Секция bootstrap отработает, когда будет создания на пустом кластере (автоматически создаются пользователи, настраивается pg_hba итд).
Если создается на уже имеющемся кластере (наш вариант), то все настройки остаются за нами.

Очень важно правильно указать значение параметра bin_dir, т.к. Patroni не знает, где находятся бинарники PG.

Есть 3 типа пользователей: для репликации (repl_user), суперпользователь (postgres), пользователь для pg_rewind.

####
```sh
root@vm-pg1:/home/asvpg# cd /etc/patroni/
root@vm-pg1:/etc/patroni# ls -altr
total 20
-rw-r--r--   1 root root  101 Aug 12 21:15 dcs.yml
-rw-r--r--   1 root root 5341 Aug 12 21:15 config.yml.in
drwxr-xr-x 109 root root 4096 Aug 15 14:35 ..
drwxr-xr-x   2 root root 4096 Aug 15 14:35 .
root@vm-pg1:/etc/patroni# touch config.yml
root@vm-pg1:/etc/patroni# chmod 600 config.yml
root@vm-pg1:/etc/patroni# ls -altr
total 20
-rw-r--r--   1 root root  101 Aug 12 21:15 dcs.yml
-rw-r--r--   1 root root 5341 Aug 12 21:15 config.yml.in
drwxr-xr-x 109 root root 4096 Aug 15 14:35 ..
-rw-------   1 root root    0 Aug 15 15:17 config.yml
drwxr-xr-x   2 root root 4096 Aug 15 15:17 .
root@vm-pg1:/etc/patroni#

root@vm-pg1:/etc/patroni# nano config.yml
root@vm-pg1:/etc/patroni#
root@vm-pg1:/etc/patroni# cat config.yml
scope: patroni
name: vm-pg1  # hostname -s
restapi:
  listen: 10.130.0.13:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.13:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.13:5432
  connect_address: 10.130.0.13:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
root@vm-pg1:/etc/patroni#


root@vm-pg2:/etc/patroni# nano config.yml
root@vm-pg2:/etc/patroni# cat config.yml
scope: patroni
name: vm-pg2  # hostname -s
restapi:
  listen: 10.130.0.28:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.28:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.28:5432
  connect_address: 10.130.0.28:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
root@vm-pg2:/etc/patroni#


root@vm-pg3:/etc/patroni# nano config.yml
root@vm-pg3:/etc/patroni# cat config.yml
scope: patroni
name: vm-pg3  # hostname -s
restapi:
  listen: 10.130.0.33:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.33:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.33:5432
  connect_address: 10.130.0.33:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
root@vm-pg3:/etc/patroni#
```

####
У суперпользователя postgres меняем пароль, а также создаем пользователей rewind_user, т.к. их нет (сделаем позже)
####
```sh
postgres@vm-pg1:~$ psql
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

postgres=# \du+
                                    List of roles
 Role name |                         Attributes                         | Description
-----------+------------------------------------------------------------+-------------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS |
 repl_user | Replication                                                |

postgres=#
postgres=# alter user postgres with password 'postgres';
ALTER ROLE
postgres=#
```

####
Останавливаем экземпляры PostgeSQL, т.к. ими будет управлять Patroni и стартуем Patroni. Получаем ошибку.
####
```sh
asvpg@vm-pg1:~$ sudo pg_ctlcluster 17 main stop
asvpg@vm-pg1:~$ sudo pg_ctlcluster 17 main status
pg_ctl: no server running
asvpg@vm-pg1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 down   postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$ sudo systemctl start patroni
Job for patroni.service failed because the control process exited with error code.
See "systemctl status patroni.service" and "journalctl -xeu patroni.service" for details.
asvpg@vm-pg1:~$

asvpg@vm-pg1:~$ systemctl status patroni.service
× patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Sat 2026-08-15 15:47:28 UTC; 1min 4s ago
    Process: 3896 ExecStart=/usr/bin/patroni /etc/patroni/config.yml (code=exited, status=1/FAILURE)
   Main PID: 3896 (code=exited, status=1/FAILURE)
        CPU: 1.226s
asvpg@vm-pg1:~$

root@vm-pg1:/etc/patroni# sudo journalctl -f -u patroni
Aug 15 15:47:28 vm-pg1 patroni[3896]: PermissionError: [Errno 13] Permission denied: '/etc/patroni/config.yml'
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Main process exited, code=exited, status=1/FAILURE
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Failed with result 'exit-code'.
Aug 15 15:47:28 vm-pg1 systemd[1]: Failed to start patroni.service - Runners to orchestrate a high-availability PostgreSQL.
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Consumed 1.226s CPU time.
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Scheduled restart job, restart counter is at 5.
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Start request repeated too quickly.
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Failed with result 'exit-code'.
Aug 15 15:47:28 vm-pg1 systemd[1]: Failed to start patroni.service - Runners to orchestrate a high-availability PostgreSQL.
Aug 15 15:47:28 vm-pg1 systemd[1]: patroni.service: Consumed 1.226s CPU time.
```

####
Некорректные права, делаем владельцем конфига пользователя postgres
####
```sh
root@vm-pg1:/etc/patroni# chown postgres:postgres /etc/patroni/config.yml
root@vm-pg1:/etc/patroni#
root@vm-pg1:/etc/patroni# ls -altr
total 24
-rw-r--r--   1 root     root      101 Aug 12 21:15 dcs.yml
-rw-r--r--   1 root     root     5341 Aug 12 21:15 config.yml.in
drwxr-xr-x 109 root     root     4096 Aug 15 14:35 ..
-rw-------   1 postgres postgres 1281 Aug 15 15:52 config.yml
drwxr-xr-x   2 root     root     4096 Aug 15 15:52 .
root@vm-pg1:/etc/patroni#
```

####
Стартуем сервис заново, проверяем и видим, что сам сервис запущен, но фактически не работает:
####
```sh
asvpg@vm-pg1:~$ sudo systemctl start patroni
asvpg@vm-pg1:~$ systemctl status patroni.service
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-08-15 15:57:28 UTC; 14s ago
   Main PID: 3954 (patroni)
      Tasks: 17 (limit: 2313)
     Memory: 30.5M (peak: 32.1M)
        CPU: 511ms
     CGroup: /system.slice/patroni.service
             └─3954 /usr/bin/python3 /usr/bin/patroni /etc/patroni/config.yml
asvpg@vm-pg1:~$ sudo systemctl status patroni
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-08-15 15:57:28 UTC; 31s ago
   Main PID: 3954 (patroni)
      Tasks: 17 (limit: 2313)
     Memory: 30.5M (peak: 32.3M)
        CPU: 555ms
     CGroup: /system.slice/patroni.service
             └─3954 /usr/bin/python3 /usr/bin/patroni /etc/patroni/config.yml

Aug 15 15:57:59 vm-pg1 patroni[3954]:     ret = self.start(timeout=timeout, block_callbacks=change_role, role=role) or >
Aug 15 15:57:59 vm-pg1 patroni[3954]:           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 15:57:59 vm-pg1 patroni[3954]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/__init__.py", line 793,>
Aug 15 15:57:59 vm-pg1 patroni[3954]:     self.config.write_postgresql_conf(configuration)
Aug 15 15:57:59 vm-pg1 patroni[3954]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/config.py", line 556, i>
Aug 15 15:57:59 vm-pg1 patroni[3954]:     os.rename(self._postgresql_conf, self._postgresql_base_conf)
Aug 15 15:57:59 vm-pg1 patroni[3954]: FileNotFoundError: [Errno 2] No such file or directory: '/var/lib/postgresql/17/m>
Aug 15 15:57:59 vm-pg1 patroni[3996]: localhost:5432 - no response
Aug 15 15:57:59 vm-pg1 patroni[3954]: 2026-08-15 15:57:59,309 INFO: Lock owner: None; I am vm-pg1
Aug 15 15:57:59 vm-pg1 patroni[3954]: 2026-08-15 15:57:59,363 INFO: failed to start postgres
asvpg@vm-pg1:~$
```

####
Patroni не смог запустить PostgreSQL, не смог захватить блокировку. Причина - в конфиге Patroni необходимо прописать параметр config_dir, т.к. с новых версий ищет конфиг в директории с данными
####
```sh
postgres@vm-pg1:~$ nano /etc/patroni/config.yml
postgres@vm-pg1:~$ cat /etc/patroni/config.yml
scope: patroni
name: vm-pg1  # hostname -s
restapi:
  listen: 10.130.0.13:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.13:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.13:5432
  connect_address: 10.130.0.13:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  config_dir: /etc/postgresql/17/main
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
postgres@vm-pg1:~$

postgres@vm-pg2:~$ nano /etc/patroni/config.yml
postgres@vm-pg2:~$ cat /etc/patroni/config.yml
scope: patroni
name: vm-pg2  # hostname -s
restapi:
  listen: 10.130.0.28:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.28:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.28:5432
  connect_address: 10.130.0.28:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  config_dir: /etc/postgresql/17/main
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
postgres@vm-pg2:~$

postgres@vm-pg3:~$ nano /etc/patroni/config.yml
postgres@vm-pg3:~$ cat /etc/patroni/config.yml
scope: patroni
name: vm-pg3  # hostname -s
restapi:
  listen: 10.130.0.33:8008 #hostname -I | tr -d " "
  connect_address: 10.130.0.33:8008 #hostname -I | tr -d " "
etcd3:
  hosts: vm-etcd1.ru-central1.internal:2379,vm-etcd2.ru-central1.internal:2379,vm-etcd3.ru-central1.internal:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
  initdb:
  - encoding: UTF8
  - data-checksums
  pg_hba:
  - host replication repl_user 10.0.0.0/8 md5
  - host all all 10.0.0.0/8 md5
  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.130.0.33:5432
  connect_address: 10.130.0.33:5432
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  config_dir: /etc/postgresql/17/main
  log_directory: /var/log/postgresql
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: repl_user
      password: repl_user
    superuser:
      username: postgres
      password: postgres
    rewind:
      username: rewind_user
      password: rewind_user
  parameters:
    unix_socket_directories: '.'
tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
postgres@vm-pg3:~$
```

####
Пробуем стартовать сервис patroni еще раз
####
```sh
asvpg@vm-pg1:~$ sudo systemctl stop patroni
asvpg@vm-pg1:~$ sudo systemctl start patroni
asvpg@vm-pg1:~$ sudo systemctl status patroni
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/patroni.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-08-15 16:07:06 UTC; 4s ago
   Main PID: 4323 (patroni)
      Tasks: 25 (limit: 2313)
     Memory: 70.2M (peak: 70.3M)
        CPU: 1.019s
     CGroup: /system.slice/patroni.service
             ├─4323 /usr/bin/python3 /usr/bin/patroni /etc/patroni/config.yml
             ├─4349 /usr/lib/postgresql/17/bin/postgres -D /var/lib/postgresql/17/main --config-file=/etc/postgresql/17>
             ├─4351 "postgres: patroni: checkpointer "
             ├─4352 "postgres: patroni: background writer "
             ├─4357 "postgres: patroni: postgres postgres 127.0.0.1(57112) idle"
             ├─4364 "postgres: patroni: walwriter "
             ├─4365 "postgres: patroni: autovacuum launcher "
             ├─4366 "postgres: patroni: logical replication launcher "
             └─4369 "postgres: patroni: walsender repl_user 10.130.0.33(38880) streaming 0/22000E60"

Aug 15 16:07:06 vm-pg1 patroni[4323]: 2026-08-15 16:07:06,813 INFO: promoted self to leader by acquiring session lock
Aug 15 16:07:06 vm-pg1 patroni[4362]: server promoting
Aug 15 16:07:06 vm-pg1 patroni[4353]: 2026-08-15 16:07:06.815 UTC [4353] LOG:  received promote request
Aug 15 16:07:06 vm-pg1 patroni[4353]: 2026-08-15 16:07:06.815 UTC [4353] LOG:  redo is not required
Aug 15 16:07:06 vm-pg1 patroni[4353]: 2026-08-15 16:07:06.831 UTC [4353] LOG:  selected new timeline ID: 2
Aug 15 16:07:06 vm-pg1 patroni[4353]: 2026-08-15 16:07:06.967 UTC [4353] LOG:  archive recovery complete
Aug 15 16:07:07 vm-pg1 patroni[4351]: 2026-08-15 16:07:07.077 UTC [4351] LOG:  checkpoint starting: force
Aug 15 16:07:07 vm-pg1 patroni[4349]: 2026-08-15 16:07:07.081 UTC [4349] LOG:  database system is ready to accept conne>
Aug 15 16:07:07 vm-pg1 patroni[4351]: 2026-08-15 16:07:07.120 UTC [4351] LOG:  checkpoint complete: wrote 3 buffers (0.>
Aug 15 16:07:08 vm-pg1 patroni[4323]: 2026-08-15 16:07:08,105 INFO: no action. I am (vm-pg1), the leader with the lock
asvpg@vm-pg1:~$
```

####
Лидерство захвачено, БД стартовала, однако остались проблемы с подключением к БД через UNIX сокет:
####
```sh
postgres@vm-pg1:~$ patroni /etc/patroni/config.yml
2026-08-15 18:27:28,165 INFO: Using default value thread_stack_size = 524288
2026-08-15 18:27:28,175 INFO: Patroni global thread_pool_size = 5
2026-08-15 18:27:28,328 INFO: Selected new etcd server http://10.129.0.23:2379
2026-08-15 18:27:28,354 WARNING: I am the leader but not owner of the lease
2026-08-15 18:27:28,355 CRITICAL: Can't start; there is already a node named 'vm-pg1' running
postgres@vm-pg1:~$
postgres@vm-pg1:~$ psql -h localhost
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

postgres=# exit
postgres@vm-pg1:~$ psql
psql: error: could not translate host name "." to address: No address associated with hostname
postgres@vm-pg1:~$
```

####
Добавляем запись для репликации в pg_hba
####
```sh
asvpg@vm-pg1:~$ sudo nano /etc/postgresql/17/main/pg_hba.conf
asvpg@vm-pg1:~$ cat /etc/postgresql/17/main/pg_hba.conf
cat: /etc/postgresql/17/main/pg_hba.conf: Permission denied
asvpg@vm-pg1:~$ sudo cat /etc/postgresql/17/main/pg_hba.conf
# PostgreSQL Client Authentication Configuration File
# ===================================================
...
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             10.0.0.0/8            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication     all             127.0.0.1/32            scram-sha-256
asvpg@vm-pg1:~$
```

####
Стартуем сервис Patroni и проверяем лог
####
```sh
asvpg@vm-pg1:~$ sudo systemctl start patroni
asvpg@vm-pg1:~$ sudo bash
root@vm-pg1:/home/asvpg# journalctl -u patroni.service -n 50 -f
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Maximum columns in an index: 32
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Maximum size of a TOAST chunk: 1996
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Size of a large-object chunk: 2048
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Date/time type storage: 64-bit integers
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Float8 argument passing: by value
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Data page checksum version: 0
Aug 15 18:37:36 vm-pg1 patroni[6827]:   Mock authentication nonce: 64ff4c4c751a10f364360e73a19655d6dd41d667752e8e896f7f2c21a3a0f3da
Aug 15 18:37:36 vm-pg1 patroni[6827]: 2026-08-15 18:37:36,673 INFO: Lock owner: None; I am vm-pg1
Aug 15 18:37:36 vm-pg1 patroni[6827]: 2026-08-15 18:37:36,673 INFO: starting as a secondary
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.077 UTC [6855] LOG:  starting PostgreSQL 17.11 (Ubuntu 17.11-1.pgdg24.04+2) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0, 64-bit
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.077 UTC [6855] LOG:  listening on IPv4 address "127.0.0.1", port 5432
Aug 15 18:37:37 vm-pg1 patroni[6827]: 2026-08-15 18:37:37,077 INFO: postmaster pid=6855
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.080 UTC [6855] LOG:  listening on IPv4 address "10.130.0.13", port 5432
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.085 UTC [6855] LOG:  listening on Unix socket "./.s.PGSQL.5432"
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.096 UTC [6860] LOG:  database system was shut down at 2026-08-15 18:34:10 UTC
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.096 UTC [6860] WARNING:  specified neither "primary_conninfo" nor "restore_command"
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.096 UTC [6860] HINT:  The database server will regularly poll the pg_wal subdirectory to check for files placed there.
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.096 UTC [6860] LOG:  entering standby mode
Aug 15 18:37:37 vm-pg1 patroni[6861]: 2026-08-15 18:37:37.104 UTC [6861] postgres@postgres FATAL:  the database system is starting up
Aug 15 18:37:37 vm-pg1 patroni[6856]: localhost:5432 - rejecting connections
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.106 UTC [6860] LOG:  consistent recovery state reached at 0/22000F10
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.106 UTC [6860] LOG:  invalid record length at 0/22000F10: expected at least 24, got 0
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.106 UTC [6860] LOG:  waiting for WAL to become available at 0/22000F28
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.107 UTC [6855] LOG:  database system is ready to accept read-only connections
Aug 15 18:37:37 vm-pg1 patroni[6862]: localhost:5432 - accepting connections
Aug 15 18:37:37 vm-pg1 patroni[6827]: 2026-08-15 18:37:37,127 INFO: establishing a new patroni heartbeat connection to postgres
Aug 15 18:37:37 vm-pg1 patroni[6867]: 2026-08-15 18:37:37.317 UTC [6867] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:37 vm-pg1 patroni[6867]: 2026-08-15 18:37:37.317 UTC [6867] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:37 vm-pg1 patroni[6827]: 2026-08-15 18:37:37,352 INFO: promoted self to leader by acquiring session lock
Aug 15 18:37:37 vm-pg1 patroni[6868]: server promoting
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.354 UTC [6860] LOG:  received promote request
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.354 UTC [6860] LOG:  redo is not required
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.366 UTC [6860] LOG:  selected new timeline ID: 3
Aug 15 18:37:37 vm-pg1 patroni[6860]: 2026-08-15 18:37:37.486 UTC [6860] LOG:  archive recovery complete
Aug 15 18:37:37 vm-pg1 patroni[6858]: 2026-08-15 18:37:37.528 UTC [6858] LOG:  checkpoint starting: force
Aug 15 18:37:37 vm-pg1 patroni[6855]: 2026-08-15 18:37:37.534 UTC [6855] LOG:  database system is ready to accept connections
Aug 15 18:37:37 vm-pg1 patroni[6858]: 2026-08-15 18:37:37.687 UTC [6858] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.009 s, sync=0.007 s, total=0.160 s; sync files=2, longest=0.006 s, average=0.004 s; distance=0 kB, estimate=0 kB; lsn=0/22000FA0, redo lsn=0/22000F48
Aug 15 18:37:38 vm-pg1 patroni[6827]: 2026-08-15 18:37:38,582 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 18:37:42 vm-pg1 patroni[6888]: 2026-08-15 18:37:42.321 UTC [6888] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:42 vm-pg1 patroni[6888]: 2026-08-15 18:37:42.321 UTC [6888] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:42 vm-pg1 patroni[6892]: 2026-08-15 18:37:42.341 UTC [6892] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:42 vm-pg1 patroni[6892]: 2026-08-15 18:37:42.341 UTC [6892] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:47 vm-pg1 patroni[6894]: 2026-08-15 18:37:47.320 UTC [6894] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:47 vm-pg1 patroni[6894]: 2026-08-15 18:37:47.320 UTC [6894] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:48 vm-pg1 patroni[6827]: 2026-08-15 18:37:48,413 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 18:37:52 vm-pg1 patroni[6895]: 2026-08-15 18:37:52.322 UTC [6895] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:52 vm-pg1 patroni[6895]: 2026-08-15 18:37:52.322 UTC [6895] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:57 vm-pg1 patroni[6896]: 2026-08-15 18:37:57.326 UTC [6896] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:37:57 vm-pg1 patroni[6896]: 2026-08-15 18:37:57.326 UTC [6896] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:37:58 vm-pg1 patroni[6827]: 2026-08-15 18:37:58,524 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 18:38:02 vm-pg1 patroni[6898]: 2026-08-15 18:38:02.334 UTC [6898] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:38:02 vm-pg1 patroni[6898]: 2026-08-15 18:38:02.334 UTC [6898] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:38:07 vm-pg1 patroni[6899]: 2026-08-15 18:38:07.331 UTC [6899] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:38:07 vm-pg1 patroni[6899]: 2026-08-15 18:38:07.331 UTC [6899] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:38:08 vm-pg1 patroni[6827]: 2026-08-15 18:38:08,413 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 18:38:12 vm-pg1 patroni[6901]: 2026-08-15 18:38:12.338 UTC [6901] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:38:12 vm-pg1 patroni[6901]: 2026-08-15 18:38:12.338 UTC [6901] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:38:17 vm-pg1 patroni[6902]: 2026-08-15 18:38:17.337 UTC [6902] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:38:17 vm-pg1 patroni[6902]: 2026-08-15 18:38:17.337 UTC [6902] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
Aug 15 18:38:18 vm-pg1 patroni[6827]: 2026-08-15 18:38:18,413 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 18:38:22 vm-pg1 patroni[6903]: 2026-08-15 18:38:22.340 UTC [6903] repl_user@[unknown] ERROR:  requested WAL segment 000000010000000000000020 has already been removed
Aug 15 18:38:22 vm-pg1 patroni[6903]: 2026-08-15 18:38:22.340 UTC [6903] repl_user@[unknown] STATEMENT:  START_REPLICATION 0/20000000 TIMELINE 1
root@vm-pg1:/home/asvpg#
```

####
Пока Patroni работает только на первой ноде. Перед тем как добавить в Patroni вторую и третью ноды необходимо создать пользователя rewind_user и добавить соответствующую строку в pg_hba
####
```sh
asvpg@vm-pg1:~$ sudo bash
root@vm-pg1:/home/asvpg# su - postgres
postgres@vm-pg1:~$ psql
psql: error: could not translate host name "." to address: No address associated with hostname
postgres@vm-pg1:~$ psql -h localhost
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

postgres=# create user rewind_user with password 'rewind_user';
CREATE ROLE
postgres=#

postgres@vm-pg1:~$ nano /etc/postgresql/17/main/pg_hba.conf
postgres@vm-pg1:~$ cat /etc/postgresql/17/main/pg_hba.conf
# PostgreSQL Client Authentication Configuration File
# ===================================================
#
# Refer to the "Client Authentication" section in the PostgreSQL
# documentation for a complete description of this file.  A short
# synopsis follows.
...
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
host    all         rewind_user         10.0.0.0/8              scram-sha-256  --!!!
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             10.0.0.0/8              scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication     all             127.0.0.1/32            scram-sha-256
postgres@vm-pg1:~$
postgres@vm-pg1:~$
```

####
Были проблемы со второй нодой (не находил исторический WAL, видимо он был удален).
Решил удалить вторую ноду из кластера Patroni и пересоздать его заново
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | running   |    |     unknown |     |    unknown |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/22002F18 |   0 | 0/22002F18 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo systemctl stop patroni
asvpg@vm-pg2:~$ ps aux | grep postgres
asvpg       8608  0.0  0.1   7080  2236 pts/0    S+   19:35   0:00 grep --color=auto postgres
asvpg@vm-pg2:~$ sudo bash
root@vm-pg2:/home/asvpg# su - postgres
postgres@vm-pg2:~$ rm -rf /var/lib/postgresql/17/main/*
postgres@vm-pg2:~$

asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml remove vm-pg2
+ Cluster: vm-pg2 (uninitialized) --+-------------+-----+------------+-----+
| Member | Host | Role | State | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+------+------+-------+----+-------------+-----+------------+-----+
+--------+------+------+-------+----+-------------+-----+------------+-----+
Please confirm the cluster name to remove: vm-pg2
You are about to remove all information in DCS for vm-pg2, please type: "Yes I am aware": Yes I am aware
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/22002F18 |   0 | 0/22002F18 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+

asvpg@vm-pg2:~$ sudo systemctl start patroni
asvpg@vm-pg2:~$ sudo journalctl -f -u patroni.service
Aug 15 19:38:53 vm-pg2 patroni[8648]: 2026-08-15 19:38:53,227 INFO: Selected new etcd server http://10.129.0.23:2379
Aug 15 19:38:53 vm-pg2 patroni[8648]: 2026-08-15 19:38:53,260 INFO: No PostgreSQL configuration items changed, nothing to reload.
Aug 15 19:38:53 vm-pg2 patroni[8648]: 2026-08-15 19:38:53,261 INFO: REST API thread_pool_size = 5
Aug 15 19:38:53 vm-pg2 systemd[1]: Started patroni.service - Runners to orchestrate a high-availability PostgreSQL.
Aug 15 19:38:53 vm-pg2 patroni[8648]: 2026-08-15 19:38:53,320 INFO: Lock owner: vm-pg1; I am vm-pg2
Aug 15 19:38:53 vm-pg2 patroni[8648]: 2026-08-15 19:38:53,377 INFO: trying to bootstrap from leader 'vm-pg1'
Aug 15 19:38:53 vm-pg2 patroni[8666]: WARNING:  skipping special file "./.s.PGSQL.5432"
Aug 15 19:38:53 vm-pg2 patroni[8666]: WARNING:  skipping special file "./.s.PGSQL.5432"
Aug 15 19:38:59 vm-pg2 patroni[8648]: 2026-08-15 19:38:59,014 INFO: Lock owner: vm-pg1; I am vm-pg2
Aug 15 19:38:59 vm-pg2 patroni[8648]: 2026-08-15 19:38:59,130 INFO: bootstrap from leader 'vm-pg1' in progress
Aug 15 19:39:09 vm-pg2 patroni[8648]: 2026-08-15 19:39:09,014 INFO: Lock owner: vm-pg1; I am vm-pg2
Aug 15 19:39:09 vm-pg2 patroni[8648]: 2026-08-15 19:39:09,069 INFO: bootstrap from leader 'vm-pg1' in progress
asvpg@vm-pg2:~$

asvpg@vm-pg2:~$ sudo journalctl -f -u patroni.service
Aug 15 19:39:36 vm-pg2 patroni[8694]: 2026-08-15 19:39:36.239 UTC [8694] LOG:  consistent recovery state reached at 0/23000158
Aug 15 19:39:36 vm-pg2 patroni[8689]: 2026-08-15 19:39:36.239 UTC [8689] LOG:  database system is ready to accept read-only connections
Aug 15 19:39:36 vm-pg2 patroni[8698]: 2026-08-15 19:39:36.264 UTC [8698] LOG:  started streaming WAL from primary at 0/24000000 on timeline 3
Aug 15 19:39:37 vm-pg2 patroni[8699]: localhost:5432 - accepting connections
Aug 15 19:39:37 vm-pg2 patroni[8648]: 2026-08-15 19:39:37,254 INFO: Lock owner: vm-pg1; I am vm-pg2
Aug 15 19:39:37 vm-pg2 patroni[8648]: 2026-08-15 19:39:37,254 INFO: establishing a new patroni heartbeat connection to postgres
Aug 15 19:39:37 vm-pg2 patroni[8648]: 2026-08-15 19:39:37,443 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 15 19:39:39 vm-pg2 patroni[8648]: 2026-08-15 19:39:39,075 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 15 19:39:49 vm-pg2 patroni[8648]: 2026-08-15 19:39:49,573 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 15 19:39:59 vm-pg2 patroni[8648]: 2026-08-15 19:39:59,572 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 15 19:40:09 vm-pg2 patroni[8648]: 2026-08-15 19:40:09,573 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)

asvpg@vm-pg3:~$ sudo journalctl -f -u patroni.service
Aug 15 19:43:39 vm-pg3 patroni[4013]: 2026-08-15 19:43:39,572 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:43:49 vm-pg3 patroni[4013]: 2026-08-15 19:43:49,572 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:43:59 vm-pg3 patroni[4013]: 2026-08-15 19:43:59,126 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:43:59 vm-pg3 patroni[4043]: 2026-08-15 19:43:59.344 UTC [4043] LOG:  restartpoint starting: time
Aug 15 19:43:59 vm-pg3 patroni[4043]: 2026-08-15 19:43:59.362 UTC [4043] LOG:  restartpoint complete: wrote 0 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.001 s, sync=0.001 s, total=0.019 s; sync files=0, longest=0.000 s, average=0.000 s; distance=16384 kB, estimate=16384 kB; lsn=0/240000B8, redo lsn=0/24000060
Aug 15 19:43:59 vm-pg3 patroni[4043]: 2026-08-15 19:43:59.362 UTC [4043] LOG:  recovery restart point at 0/24000060
Aug 15 19:44:09 vm-pg3 patroni[4013]: 2026-08-15 19:44:09,132 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:44:19 vm-pg3 patroni[4013]: 2026-08-15 19:44:19,572 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:44:29 vm-pg3 patroni[4013]: 2026-08-15 19:44:29,571 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)
Aug 15 19:44:39 vm-pg3 patroni[4013]: 2026-08-15 19:44:39,571 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg1)

asvpg@vm-pg1:~$ sudo journalctl -f -u patroni.service
Aug 15 19:43:53 vm-pg1 patroni[8441]: 2026-08-15 19:43:53.649 UTC [8441] LOG:  checkpoint complete: wrote 0 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.001 s, sync=0.001 s, total=0.019 s; sync files=0, longest=0.000 s, average=0.000 s; distance=16384 kB, estimate=16384 kB; lsn=0/240000B8, redo lsn=0/24000060
Aug 15 19:43:59 vm-pg1 patroni[6827]: 2026-08-15 19:43:59,069 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:09 vm-pg1 patroni[6827]: 2026-08-15 19:44:09,070 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:18 vm-pg1 patroni[6827]: 2026-08-15 19:44:18,958 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:29 vm-pg1 patroni[6827]: 2026-08-15 19:44:29,012 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:38 vm-pg1 patroni[6827]: 2026-08-15 19:44:38,958 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:48 vm-pg1 patroni[6827]: 2026-08-15 19:44:48,958 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:44:58 vm-pg1 patroni[6827]: 2026-08-15 19:44:58,959 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:45:08 vm-pg1 patroni[6827]: 2026-08-15 19:45:08,958 INFO: no action. I am (vm-pg1), the leader with the lock
Aug 15 19:45:18 vm-pg1 patroni[6827]: 2026-08-15 19:45:18,958 INFO: no action. I am (vm-pg1), the leader with the lock

asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming |  3 |  0/24000060 |   0 | 0/24000060 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/24000060 |   0 | 0/24000060 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$
```

####
Управлять Patroni можно с любом ноды, как такового мастера у него нет.
Листенер пропишется автоматически (nano /etc/postgresql/17/main/postgresql.conf), т.е. Patroni заменил базовый файл конфигурации.
Важно: в случае развертывания на существующем кластере необходимо поправить pg_hba на всех нодах кластера, иначе при swithcover/failover 
к новым лидерам не получится подключиться!
####

####
Для проверки корректности заполнения конфига Patroni (отступы итп) есть команда валидации
####
```sh
asvpg@vm-pg2:~$ sudo patroni --validate-config /etc/patroni/config.yml
postgresql.listen 127.0.0.1, 10.130.0.28:5432 didn't pass validation: [Errno -2] Name or service not known
asvpg@vm-pg2:~$
```

####
Есть возможность очистить конфиг DCS
####
```sh
sudo -u postgres patroni /etc/patroni/config.yml --clean
```

####
Есть возможность собрать с другим DCS
####
```sh
sudo -u postgres patronictl -c /etc/patroni/config.yml remove patroni (patroni - имя в конфиге в параметре scope)
```

####
Просмотр конфигурации Patroni
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml show-config
loop_wait: 10
maximum_lag_on_failover: 1048576
postgresql:
  use_pg_rewind: true
retry_timeout: 10
ttl: 30

asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo patronictl -c /etc/patroni/config.yml show-config
loop_wait: 10
maximum_lag_on_failover: 1048576
postgresql:
  use_pg_rewind: true
retry_timeout: 10
ttl: 30

asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo patronictl -c /etc/patroni/config.yml show-config
loop_wait: 10
maximum_lag_on_failover: 1048576
postgresql:
  use_pg_rewind: true
retry_timeout: 10
ttl: 30

asvpg@vm-pg3:~$
```

####
Проверяем отсутствие зависших\неактивных слотов репликации
####
```sh
asvpg@vm-pg1:~$ sudo -u postgres psql -h localhost -c "SELECT slot_name, slot_type, active, wal_status FROM pg_replication_slots;"
Password for user postgres:
 slot_name | slot_type | active | wal_status
-----------+-----------+--------+------------
 vm_pg3    | physical  | t      | reserved
 vm_pg2    | physical  | t      | reserved
(2 rows)

asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo -u postgres psql -h localhost -c "SELECT slot_name, slot_type, active, wal_status FROM pg_replication_slots;"
Password for user postgres:
 slot_name | slot_type | active | wal_status
-----------+-----------+--------+------------
 vm_pg3    | physical  | f      | reserved
 vm_pg1    | physical  | f      | reserved
(2 rows)

asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo -u postgres psql -h localhost -c "SELECT slot_name, slot_type, active, wal_status FROM pg_replication_slots;"
Password for user postgres:
 slot_name | slot_type | active | wal_status
-----------+-----------+--------+------------
 vm_pg1    | physical  | f      | reserved
 vm_pg2    | physical  | f      | reserved
(2 rows)

asvpg@vm-pg3:~$
```

####
Есть возможность выполнить рестарт ноды кластера Patroni
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml restart patroni vm-pg2
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
When should the restart take place (e.g. 2026-08-15T21:17)  [now]:
Are you sure you want to restart members vm-pg2? [y/N]: y
Restart if the PostgreSQL version is less than provided (e.g. 9.5.2)  []:
Success: restart on member vm-pg2
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$
```

####
Можно выполнить плановое переключение
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml switchover
Current cluster topology
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  3 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  3 |  0/24000168 |   0 | 0/24000168 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
Primary [vm-pg1]:
Candidate ['vm-pg2', 'vm-pg3'] []: vm-pg2
When should the switchover take place (e.g. 2026-08-15T21:20 )  [now]:
Are you sure you want to switchover cluster patroni, demoting current leader vm-pg1? [y/N]: y
2026-08-15 20:20:18.63812 Successfully switched over to "vm-pg2"
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | stopped |    |     unknown |     |    unknown |     |
| vm-pg2 | 10.130.0.28 | Leader  | running |  3 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | running |  3 |  0/240002B0 |   0 | 0/240002B0 |   0 |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | running |  3 |  0/24000000 |   0 | 0/240002B0 |   0 |
| vm-pg2 | 10.130.0.28 | Leader  | running |  4 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | running |  3 |  0/240002B0 |   0 | 0/240002B0 |   0 |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$
```

####
При смене лидера происходит инкремент TL. Если это не произошло на всех нодах кластера, значит скорее всего есть ошибки в настройке pg_hba. Смотрим логи и исправляем
####
```sh
asvpg@vm-pg1:~$ sudo journalctl -u patroni.service -n 50 -f
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3.12/contextlib.py", line 137, in __enter__
Aug 15 20:23:09 vm-pg1 patroni[6827]:     return next(self.gen)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/__init__.py", line 1114, in get_replication_connection_cursor
Aug 15 20:23:09 vm-pg1 patroni[6827]:     with get_connection_cursor(**conn_kwargs) as cur:
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3.12/contextlib.py", line 137, in __enter__
Aug 15 20:23:09 vm-pg1 patroni[6827]:     return next(self.gen)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/connection.py", line 164, in get_connection_cursor
Aug 15 20:23:09 vm-pg1 patroni[6827]:     conn = psycopg.connect(**conn_kwargs)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/psycopg.py", line 150, in connect
Aug 15 20:23:09 vm-pg1 patroni[6827]:     ret = _connect(*args, **kwargs)
Aug 15 20:23:09 vm-pg1 patroni[6827]:           ^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/psycopg2/__init__.py", line 122, in connect
Aug 15 20:23:09 vm-pg1 patroni[6827]:     conn = _connect(dsn, connection_factory=connection_factory, **kwasync)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]: psycopg2.OperationalError: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", SSL encryption
Aug 15 20:23:09 vm-pg1 patroni[6827]: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", no encryption
Aug 15 20:23:09 vm-pg1 patroni[6827]: 2026-08-15 20:23:09,864 INFO: Local timeline=3 lsn=0/240002B0
Aug 15 20:23:09 vm-pg1 patroni[6827]: 2026-08-15 20:23:09,892 ERROR: Exception when working with primary via replication connection
Aug 15 20:23:09 vm-pg1 patroni[6827]: Traceback (most recent call last):
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/rewind.py", line 253, in _check_timeline_and_lsn
Aug 15 20:23:09 vm-pg1 patroni[6827]:     with self._postgresql.get_replication_connection_cursor(**leader.conn_kwargs()) as cur:
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3.12/contextlib.py", line 137, in __enter__
Aug 15 20:23:09 vm-pg1 patroni[6827]:     return next(self.gen)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/__init__.py", line 1114, in get_replication_connection_cursor
Aug 15 20:23:09 vm-pg1 patroni[6827]:     with get_connection_cursor(**conn_kwargs) as cur:
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3.12/contextlib.py", line 137, in __enter__
Aug 15 20:23:09 vm-pg1 patroni[6827]:     return next(self.gen)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/postgresql/connection.py", line 164, in get_connection_cursor
Aug 15 20:23:09 vm-pg1 patroni[6827]:     conn = psycopg.connect(**conn_kwargs)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/patroni/psycopg.py", line 150, in connect
Aug 15 20:23:09 vm-pg1 patroni[6827]:     ret = _connect(*args, **kwargs)
Aug 15 20:23:09 vm-pg1 patroni[6827]:           ^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]:   File "/usr/lib/python3/dist-packages/psycopg2/__init__.py", line 122, in connect
Aug 15 20:23:09 vm-pg1 patroni[6827]:     conn = _connect(dsn, connection_factory=connection_factory, **kwasync)
Aug 15 20:23:09 vm-pg1 patroni[6827]:            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aug 15 20:23:09 vm-pg1 patroni[6827]: psycopg2.OperationalError: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", SSL encryption
Aug 15 20:23:09 vm-pg1 patroni[6827]: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", no encryption
Aug 15 20:23:09 vm-pg1 patroni[6827]: 2026-08-15 20:23:09,960 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 15 20:23:10 vm-pg1 patroni[9148]: 2026-08-15 20:23:10.626 UTC [9148] FATAL:  could not connect to the primary server: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", SSL encryption
Aug 15 20:23:10 vm-pg1 patroni[9148]:         connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", no encryption
Aug 15 20:23:10 vm-pg1 patroni[9010]: 2026-08-15 20:23:10.627 UTC [9010] LOG:  waiting for WAL to become available at 0/240002C8
Aug 15 20:23:15 vm-pg1 patroni[9149]: 2026-08-15 20:23:15.631 UTC [9149] FATAL:  could not connect to the primary server: connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", SSL encryption
Aug 15 20:23:15 vm-pg1 patroni[9149]:         connection to server at "10.130.0.28", port 5432 failed: FATAL:  no pg_hba.conf entry for replication connection from host "10.130.0.13", user "repl_user", no encryption
Aug 15 20:23:15 vm-pg1 patroni[9010]: 2026-08-15 20:23:15.632 UTC [9010] LOG:  waiting for WAL to become available at 0/240002C8


asvpg@vm-pg1:~$ sudo nano /etc/postgresql/17/main/pg_hba.conf
asvpg@vm-pg1:~$ sudo cat /etc/postgresql/17/main/pg_hba.conf
# PostgreSQL Client Authentication Configuration File
# ===================================================
...
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
host    all         rewind_user         10.0.0.0/8              scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             10.0.0.0/8              scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     repl_user       10.0.0.0/8              scram-sha-256
asvpg@vm-pg1:~$

--на каждой ноде в конфиге pg_hba добавляем следующую строку:
host    replication     repl_user       10.0.0.0/8              scram-sha-256

--ниже представлены все записи в конфиге pg_hba
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
host    all         rewind_user         10.0.0.0/8              scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
host    replication     repl_user       10.0.0.0/8              scram-sha-256
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication     all             10.0.0.0/8              scram-sha-256

asvpg@vm-pg1:~$ sudo nano /etc/postgresql/17/main/pg_hba.conf
asvpg@vm-pg1:~$ sudo pg_ctlcluster 17 main reload

asvpg@vm-pg2:~$ sudo nano /etc/postgresql/17/main/pg_hba.conf
asvpg@vm-pg2:~$ sudo pg_ctlcluster 17 main reload

asvpg@vm-pg3:~$ sudo nano /etc/postgresql/17/main/pg_hba.conf
asvpg@vm-pg3:~$ sudo pg_ctlcluster 17 main reload

--Теперь TL поменялись, ошибки ушли
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming |  4 |  0/240003F0 |   0 | 0/240003F0 |   0 |
| vm-pg2 | 10.130.0.28 | Leader  | running   |  4 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  4 |  0/240003F0 |   0 | 0/240003F0 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

--делаем обратный switchover для проверки
asvpg@vm-pg2:~$ sudo patronictl -c /etc/patroni/config.yml switchover
Current cluster topology
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming |  4 |  0/240003F0 |   0 | 0/240003F0 |   0 |
| vm-pg2 | 10.130.0.28 | Leader  | running   |  4 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  4 |  0/240003F0 |   0 | 0/240003F0 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
Primary [vm-pg2]:
Candidate ['vm-pg1', 'vm-pg3'] []: vm-pg1
When should the switchover take place (e.g. 2026-08-15T21:48 )  [now]:
Are you sure you want to switchover cluster patroni, demoting current leader vm-pg2? [y/N]: y
2026-08-15 20:48:09.38332 Successfully switched over to "vm-pg1"
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running |  4 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | stopped |    |     unknown |     |    unknown |     |
| vm-pg3 | 10.130.0.33 | Replica | running |  4 |  0/24000538 |   0 | 0/24000538 |   0 |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg2:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   |  5 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming |  5 |  0/24000678 |   0 | 0/24000678 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming |  5 |  0/24000678 |   0 | 0/24000678 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg2:~$
```

####
Управлять можно не только через утилиту patronictl, но и через REST API.
По localhost в новых версиях Patroni не работает, только на внутреннем IP
####
```sh
asvpg@vm-pg1:~$ curl -s http://localhost:8008/patroni | jq .
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$ curl -s http://10.130.0.13:8008/patroni | jq .
{
  "state": "running",
  "postmaster_start_time": "2026-08-15 19:17:48.871339+00:00",
  "role": "primary",
  "server_version": 170011,
  "xlog": {
    "location": 603980136
  },
  "timeline": 3,
  "replication": [
    {
      "usename": "repl_user",
      "application_name": "vm-pg3",
      "client_addr": "10.130.0.33",
      "state": "streaming",
      "sync_state": "async",
      "sync_priority": 0
    },
    {
      "usename": "repl_user",
      "application_name": "vm-pg2",
      "client_addr": "10.130.0.28",
      "state": "streaming",
      "sync_state": "async",
      "sync_priority": 0
    }
  ],
  "dcs_last_seen": 1786824828,
  "database_system_identifier": "7673599298398442043",
  "patroni": {
    "version": "4.1.5",
    "scope": "patroni",
    "name": "vm-pg1"
  }
}
asvpg@vm-pg1:~$
```

###
4. Настройка pgbouncer
###

####
Устанавливается как сервис ОС на тех же серверах, где функционирует PostgreSQL. Является пулером соединений и обычно располагается между прикладными подами (например, Hikari) и СУБД.
####
```sh
--подготовка к установке - обновляем локальные списки пакетов APT. Это необходимо, чтобы система знала о самых свежих версиях ПО в репозиториях Яндекса.
asvpg@vm-pg2:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Get:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease [126 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease [126 kB]
Hit:4 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
Get:5 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Packages [1191 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Components [180 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Packages [1683 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Components [388 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:11 http://mirror.yandex.ru/ubuntu noble-backports/main amd64 Components [5740 B]
Get:12 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Components [12.6 kB]
Get:13 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [46.4 kB]
Get:14 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [76.3 kB]
Fetched 3963 kB in 1s (3871 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
W: http://apt.postgresql.org/pub/repos/apt/dists/noble-pgdg/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
asvpg@vm-pg2:~$

--на первой ноде есть проблема с подключением:
asvpg@vm-pg1:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Hit:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:5 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
Ign:5 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
Ign:5 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
Err:5 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
  Cannot initiate the connection to apt.postgresql.org:80 (2a04:4e42:6b::820). - connect (101: Network is unreachable) Could not connect to apt.postgresql.org:80 (151.101.247.52), connection timed out
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
6 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: Failed to fetch http://apt.postgresql.org/pub/repos/apt/dists/noble-pgdg/InRelease  Cannot initiate the connection to apt.postgresql.org:80 (2a04:4e42:6b::820). - connect (101: Network is unreachable) Could not connect to apt.postgresql.org:80 (151.101.247.52), connection timed out
W: Some index files failed to download. They have been ignored, or old ones used instead.
asvpg@vm-pg1:~$

--ранее, в рамках настройки PostgreSQL на второй ноде была аналогичная ошибка, выполняли принудительное использование IPv4 для менеджера пакетов
asvpg@vm-pg1:~$ sudo cat /etc/apt/apt.conf.d/99force-ipv4
cat: /etc/apt/apt.conf.d/99force-ipv4: No such file or directory
asvpg@vm-pg1:~$ echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4
Acquire::ForceIPv4 "true";
asvpg@vm-pg1:~$ sudo cat /etc/apt/apt.conf.d/99force-ipv4
Acquire::ForceIPv4 "true";
asvpg@vm-pg1:~$

asvpg@vm-pg3:~$ sudo cat /etc/apt/apt.conf.d/99force-ipv4
cat: /etc/apt/apt.conf.d/99force-ipv4: No such file or directory
asvpg@vm-pg3:~$ echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4
Acquire::ForceIPv4 "true";
asvpg@vm-pg3:~$ sudo cat /etc/apt/apt.conf.d/99force-ipv4
Acquire::ForceIPv4 "true";
asvpg@vm-pg3:~$

--настройки не помогли, выполнил рестарт ВМ в Яндекс Облаке, после этого всё получилось
asvpg@vm-pg1:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Hit:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease
Hit:4 http://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease
Hit:5 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
6 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: http://apt.postgresql.org/pub/repos/apt/dists/noble-pgdg/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
asvpg@vm-pg1:~$

--устанавливаем последние обновления безопасности и библиотек для всех установленных пакетов
asvpg@vm-pg1:~$ sudo apt upgrade -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  google-compute-engine-oslogin
The following packages will be upgraded:
  krb5-locales libgssapi-krb5-2 libk5crypto3 libkrb5-3 libkrb5support0
5 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
Need to get 622 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 krb5-locales all 1.20.1-6ubuntu2.8 [15.1 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libgssapi-krb5-2 amd64 1.20.1-6ubuntu2.8 [143 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5-3 amd64 1.20.1-6ubuntu2.8 [348 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5support0 amd64 1.20.1-6ubuntu2.8 [34.7 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libk5crypto3 amd64 1.20.1-6ubuntu2.8 [81.9 kB]
Fetched 622 kB in 0s (27.0 MB/s)
(Reading database ... 109305 files and directories currently installed.)
Preparing to unpack .../krb5-locales_1.20.1-6ubuntu2.8_all.deb ...
Unpacking krb5-locales (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libgssapi-krb5-2_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5-3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5support0_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libk5crypto3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Setting up krb5-locales (1.20.1-6ubuntu2.8) ...
Setting up libkrb5support0:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libk5crypto3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libkrb5-3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart patroni.service ssh.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #2: sshd[1295]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg1:~$

--для второй и третьей нод обновлений почему то не нашлось:
asvpg@vm-pg2:~$ sudo apt upgrade -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  google-compute-engine-oslogin
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo apt upgrade -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  google-compute-engine-oslogin
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
asvpg@vm-pg3:~$

--устанавливаем пулер pgbouncer
asvpg@vm-pg1:~$ sudo apt install -y pgbouncer
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcares2 libevent-2.1-7t64
The following NEW packages will be installed:
  libcares2 libevent-2.1-7t64 pgbouncer
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 453 kB of archives.
After this operation, 1152 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 libcares2 amd64 1.27.0-1.0ubuntu1 [73.7 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 libevent-2.1-7t64 amd64 2.1.12-stable-9ubuntu2 [145 kB]
Get:3 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1 [235 kB]
Fetched 453 kB in 0s (2035 kB/s)
Selecting previously unselected package libcares2:amd64.
(Reading database ... 109305 files and directories currently installed.)
Preparing to unpack .../libcares2_1.27.0-1.0ubuntu1_amd64.deb ...
Unpacking libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Selecting previously unselected package libevent-2.1-7t64:amd64.
Preparing to unpack .../libevent-2.1-7t64_2.1.12-stable-9ubuntu2_amd64.deb ...
Unpacking libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Selecting previously unselected package pgbouncer.
Preparing to unpack .../pgbouncer_1.25.2-1.pgdg24.04+1_amd64.deb ...
Unpacking pgbouncer (1.25.2-1.pgdg24.04+1) ...
Setting up libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Setting up libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Setting up pgbouncer (1.25.2-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/pgbouncer.service → /usr/lib/systemd/system/pgbouncer.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #2: sshd[1295]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg1:~$

--на второй ноде есть проблема с подключением к репозиторию
asvpg@vm-pg2:~$ sudo apt install -y pgbouncer
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcares2 libevent-2.1-7t64
The following NEW packages will be installed:
  libcares2 libevent-2.1-7t64 pgbouncer
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 235 kB/453 kB of archives.
After this operation, 1152 kB of additional disk space will be used.
Ign:1 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1
Ign:1 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1
Ign:1 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1
Err:1 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1
  Could not connect to apt.postgresql.org:80 (199.232.175.52), connection timed out
E: Failed to fetch http://apt.postgresql.org/pub/repos/apt/pool/main/p/pgbouncer/pgbouncer_1.25.2-1.pgdg24.04%2b1_amd64.deb  Could not connect to apt.postgresql.org:80 (199.232.175.52), connection timed out
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
asvpg@vm-pg2:~$

--DNS работает (мы видим IP-адрес), но TCP-соединение по порту 80 обрывается с таймаутом.
--Есть подозрения на нестабильную работу сетевой инфраструктуры Яндекс Облака.
--Что то с одной из следующих настроек:
Cloud NAT: Подсеть этой машины отвязалась от шлюза.
Security Group (Egress): Правило разрешающее исходящий трафик слетело или имеет низкий приоритет.
Cloud Router / Route Table: Пропал маршрут по умолчанию для подсети.
--Выполнял detach\attach публичного IP-адреса - не помогло.
--В итоге помог перезапуск службы DNA-резолвера.
--Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.130... - сообщение говорит о том, что служба разрешения имен видела сетевые ошибки при общении с внутренним DNS-сервером Яндекса. Из-за этого она перешла в «деградированный режим» и перестала корректно обрабатывать запросы от сложных приложений вроде APT, хотя базовый пинг мог проходить.

Простой перезапуск службы (restart) сбросил кэш и заставил её заново договориться с облачным DNS-протоколом на нормальных скоростях.
svpg@vm-pg2:~$ sudo systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 08:02:11 UTC; 2min 32s ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 442 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 2313)
     Memory: 7.5M (peak: 8.0M)
        CPU: 73ms
     CGroup: /system.slice/systemd-resolved.service
             └─442 /usr/lib/systemd/systemd-resolved

Aug 16 08:02:11 vm-pg2 systemd[1]: Starting systemd-resolved.service - Network Name Resolution...
Aug 16 08:02:11 vm-pg2 systemd-resolved[442]: Positive Trust Anchors:
Aug 16 08:02:11 vm-pg2 systemd-resolved[442]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc68345710423>
Aug 16 08:02:11 vm-pg2 systemd-resolved[442]: Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17.>
Aug 16 08:02:11 vm-pg2 systemd-resolved[442]: Using system hostname 'vm-pg2'.
Aug 16 08:02:11 vm-pg2 systemd[1]: Started systemd-resolved.service - Network Name Resolution.
Aug 16 08:02:16 vm-pg2 systemd-resolved[442]: Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.130>
Aug 16 08:02:22 vm-pg2 systemd-resolved[442]: Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.130>
Aug 16 08:02:59 vm-pg2 systemd-resolved[442]: Clock change detected. Flushing caches.

asvpg@vm-pg2:~$ sudo systemctl restart systemd-resolved
asvpg@vm-pg2:~$ sudo systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 08:05:17 UTC; 3s ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 1394 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 2313)
     Memory: 2.5M (peak: 2.5M)
        CPU: 63ms
     CGroup: /system.slice/systemd-resolved.service
             └─1394 /usr/lib/systemd/systemd-resolved

Aug 16 08:05:17 vm-pg2 systemd[1]: Starting systemd-resolved.service - Network Name Resolution...
Aug 16 08:05:17 vm-pg2 systemd-resolved[1394]: Positive Trust Anchors:
Aug 16 08:05:17 vm-pg2 systemd-resolved[1394]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc6834571042>
Aug 16 08:05:17 vm-pg2 systemd-resolved[1394]: Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17>
Aug 16 08:05:17 vm-pg2 systemd-resolved[1394]: Using system hostname 'vm-pg2'.
Aug 16 08:05:17 vm-pg2 systemd[1]: Started systemd-resolved.service - Network Name Resolution.

asvpg@vm-pg2:~$ sudo apt install -y pgbouncer
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcares2 libevent-2.1-7t64
The following NEW packages will be installed:
  libcares2 libevent-2.1-7t64 pgbouncer
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 235 kB/453 kB of archives.
After this operation, 1152 kB of additional disk space will be used.
Get:1 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1 [235 kB]
Fetched 235 kB in 0s (1383 kB/s)
Selecting previously unselected package libcares2:amd64.
(Reading database ... 109305 files and directories currently installed.)
Preparing to unpack .../libcares2_1.27.0-1.0ubuntu1_amd64.deb ...
Unpacking libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Selecting previously unselected package libevent-2.1-7t64:amd64.
Preparing to unpack .../libevent-2.1-7t64_2.1.12-stable-9ubuntu2_amd64.deb ...
Unpacking libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Selecting previously unselected package pgbouncer.
Preparing to unpack .../pgbouncer_1.25.2-1.pgdg24.04+1_amd64.deb ...
Unpacking pgbouncer (1.25.2-1.pgdg24.04+1) ...
Setting up libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Setting up libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Setting up pgbouncer (1.25.2-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/pgbouncer.service → /usr/lib/systemd/system/pgbouncer.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg2:~$


asvpg@vm-pg3:~$ sudo apt install -y pgbouncer
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcares2 libevent-2.1-7t64
The following NEW packages will be installed:
  libcares2 libevent-2.1-7t64 pgbouncer
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 453 kB of archives.
After this operation, 1152 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 libcares2 amd64 1.27.0-1.0ubuntu1 [73.7 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble/main amd64 libevent-2.1-7t64 amd64 2.1.12-stable-9ubuntu2 [145 kB]
Get:3 http://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbouncer amd64 1.25.2-1.pgdg24.04+1 [235 kB]
Fetched 453 kB in 0s (3658 kB/s)
Selecting previously unselected package libcares2:amd64.
(Reading database ... 109305 files and directories currently installed.)
Preparing to unpack .../libcares2_1.27.0-1.0ubuntu1_amd64.deb ...
Unpacking libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Selecting previously unselected package libevent-2.1-7t64:amd64.
Preparing to unpack .../libevent-2.1-7t64_2.1.12-stable-9ubuntu2_amd64.deb ...
Unpacking libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Selecting previously unselected package pgbouncer.
Preparing to unpack .../pgbouncer_1.25.2-1.pgdg24.04+1_amd64.deb ...
Unpacking pgbouncer (1.25.2-1.pgdg24.04+1) ...
Setting up libevent-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Setting up libcares2:amd64 (1.27.0-1.0ubuntu1) ...
Setting up pgbouncer (1.25.2-1.pgdg24.04+1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/pgbouncer.service → /usr/lib/systemd/system/pgbouncer.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1255]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-pg3:~$
```

####
Проверяем статус службы:
####
```sh
asvpg@vm-pg1:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 07:38:18 UTC; 31min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 776 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
      Tasks: 3 (limit: 2313)
     Memory: 5.9M (peak: 6.2M)
        CPU: 240ms
     CGroup: /system.slice/pgbouncer.service
             └─776 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 08:00:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:01:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:02:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:03:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:04:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:05:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:06:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:07:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:08:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:09:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >

asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 08:05:33 UTC; 5min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 1593 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
      Tasks: 3 (limit: 2313)
     Memory: 1.8M (peak: 2.3M)
        CPU: 43ms
     CGroup: /system.slice/pgbouncer.service
             └─1593 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 08:05:33 vm-pg2 pgbouncer[1593]: listening on [::1]:6432
Aug 16 08:05:33 vm-pg2 pgbouncer[1593]: listening on 127.0.0.1:6432
Aug 16 08:05:33 vm-pg2 pgbouncer[1593]: listening on unix:/var/run/postgresql/.s.PGSQL.6432
Aug 16 08:05:33 vm-pg2 pgbouncer[1593]: process up: PgBouncer 1.25.2, libevent 2.1.12-stable (epoll), adns: c-ares 1.27>
Aug 16 08:05:33 vm-pg2 systemd[1]: Started pgbouncer.service - connection pooler for PostgreSQL.
Aug 16 08:06:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:07:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:08:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:09:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:10:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>

asvpg@vm-pg2:~$


asvpg@vm-pg3:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 07:38:17 UTC; 33min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 778 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
      Tasks: 3 (limit: 2313)
     Memory: 5.9M (peak: 6.1M)
        CPU: 272ms
     CGroup: /system.slice/pgbouncer.service
             └─778 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 08:02:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:03:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:04:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:05:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:06:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:07:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:08:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:09:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:10:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:11:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >

asvpg@vm-pg3:~$
```

####
Настраиваем конфиги pgbouncer, предварительно останавливая сервис
####
```sh
asvpg@vm-pg1:~$ sudo systemctl stop pgbouncer
asvpg@vm-pg1:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:08 UTC; 1s ago
   Duration: 34min 49.811s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 776 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 776 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
        CPU: 262ms

Aug 16 08:08:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:09:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:10:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:11:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:12:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: server connections dropped, exiting
Aug 16 08:13:08 vm-pg1 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.

asvpg@vm-pg1:~$


--
postgres@vm-pg3:~$ cd /etc/pgbouncer/
postgres@vm-pg3:/etc/pgbouncer$ ls -altr
total 20
-rw-r-----   1 postgres postgres     0 May  9 12:09 userlist.txt
-rw-r-----   1 postgres postgres 11228 May  9 12:09 pgbouncer.ini
drwxr-xr-x   2 root     root      4096 Aug 16 07:31 .
drwxr-xr-x 110 root     root      4096 Aug 16 07:31 ..
postgres@vm-pg3:/etc/pgbouncer$

root@vm-pg3:/etc/pgbouncer# cat pgbouncer.ini
[databases]
thai = host=127.0.0.1 port=5432 dbname=thai
[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = 10.130.0.33
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
admin_users = admindb
root@vm-pg3:/etc/pgbouncer#

root@vm-pg1:/home/asvpg# cd /etc/pgbouncer/
root@vm-pg1:/etc/pgbouncer# ls -altr
total 20
-rw-r-----   1 postgres postgres     0 May  9 12:09 userlist.txt
-rw-r-----   1 postgres postgres 11228 May  9 12:09 pgbouncer.ini
drwxr-xr-x   2 root     root      4096 Aug 16 07:31 .
drwxr-xr-x 110 root     root      4096 Aug 16 07:31 ..
root@vm-pg1:/etc/pgbouncer# cp pgbouncer.ini pgbouncer.ini.default
root@vm-pg1:/etc/pgbouncer# rm pgbouncer.ini
root@vm-pg1:/etc/pgbouncer# touch pgbouncer.ini
root@vm-pg1:/etc/pgbouncer# chown postgres:postgres pgbouncer.ini
root@vm-pg1:/etc/pgbouncer# chmod 640 pgbouncer.ini
root@vm-pg1:/etc/pgbouncer# ls -altr
total 20
-rw-r-----   1 postgres postgres     0 May  9 12:09 userlist.txt
drwxr-xr-x 110 root     root      4096 Aug 16 07:31 ..
-rw-r-----   1 root     root     11228 Aug 16 08:39 pgbouncer.ini.default
-rw-r-----   1 postgres postgres     0 Aug 16 08:39 pgbouncer.ini
drwxr-xr-x   2 root     root      4096 Aug 16 08:39 .
root@vm-pg1:/etc/pgbouncer#

root@vm-pg1:/etc/pgbouncer# cat pgbouncer.ini
[databases]
thai = host=127.0.0.1 port=5432 dbname=thai
[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = 10.130.0.13
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
admin_users = admindb
root@vm-pg1:/etc/pgbouncer#

root@vm-pg2:/etc/pgbouncer# cat pgbouncer.ini
[databases]
thai = host=127.0.0.1 port=5432 dbname=thai
[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = 10.130.0.28
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
admin_users = admindb
root@vm-pg2:/etc/pgbouncer#

--создаем пользователя admindb - будет администратором pgbouncer
asvpg@vm-pg3:~$ sudo -u postgres psql -h localhost
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

postgres=# create user admindb with password 'admindb';
CREATE ROLE
postgres=#

--просмотрим соль\хеш созданных пользователей
postgres=# select usename,passwd from pg_shadow;
   usename   |                                                                passwd                                                             
-------------+--------------------------------------------------------------------------------------------------------------------------------------
 repl_user   | SCRAM-SHA-256$4096:DvCV1uQ8ot382XA0v/pwKA==$ynl+tF9ICJBNMlChe9OuhTv7gDjJudazcbOAkLn0zXc=:sNera5KzYSH4MeXxh77TOPVAiKfEyS/47oHPHI905h4=
 postgres    | SCRAM-SHA-256$4096:lw/7mLaZFdn0cSzYo2cNmQ==$FiW/u1WDfn4BzAMMNKezNkuRewIGLKwxssoDDKlXlAo=:ayX5tzUNWHZdyTSmyX3/zV1/VEi+j97e1KapH5KoVic=
 rewind_user | SCRAM-SHA-256$4096:2q526mRy39kyGJexXLpefg==$IdxNiSd0yp7dMelhNxgxWyFrgyVG9BmXtwwyDEp7E7w=:xSmAM8d/qZIOKB4/vfOvYNca+hnUwm51hL/kv1xQ9gY=
 admindb     | SCRAM-SHA-256$4096:UOxKB4r22j9WGjS+p/uCpQ==$awO0lcLCDGxUn9JX1TIqPJbLZ3pvIfk+Y3uxm8ruZ+s=:bIhKT3V99UHDxpkKaTH+ByfWpTZE5qkNMxYhv52bB3A=
(4 rows)

--на каждой из 3 нод в файл userlist.txt укажем хеш для 2 пользователей- admindb, postgres (пароль указываем через кавычки, иначе спец символы будут экранироваться и пароль будет обрезан)
root@vm-pg3:/etc/pgbouncer# nano userlist.txt
root@vm-pg3:/etc/pgbouncer# cat userlist.txt
"admindb" "SCRAM-SHA-256$4096:UOxKB4r22j9WGjS+p/uCpQ==$awO0lcLCDGxUn9JX1TIqPJbLZ3pvIfk+Y3uxm8ruZ+s=:bIhKT3V99UHDxpkKaTH+ByfWpTZE5qkNMxYhv52bB3A="
"postgres" "SCRAM-SHA-256$4096:lw/7mLaZFdn0cSzYo2cNmQ==$FiW/u1WDfn4BzAMMNKezNkuRewIGLKwxssoDDKlXlAo=:ayX5tzUNWHZdyTSmyX3/zV1/VEi+j97e1KapH5KoVic="
root@vm-pg3:/etc/pgbouncer#
```

####
Запускать pgbouncer можно в качестве сервиса или в качестве демона. Второе не рекомендуется из-за сложности управления, поэтому выбираем управление через сервис.
####
```sh
asvpg@vm-pg3:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:42 UTC; 42min ago
   Duration: 35min 24.782s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 778 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 778 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 μs, query 0 μs, wait 0>
        CPU: 290ms

Aug 16 08:09:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
Aug 16 08:10:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
Aug 16 08:11:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
Aug 16 08:12:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
Aug 16 08:13:18 vm-pg3 pgbouncer[778]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
Aug 16 08:13:42 vm-pg3 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:42 vm-pg3 pgbouncer[778]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:42 vm-pg3 pgbouncer[778]: server connections dropped, exiting
Aug 16 08:13:42 vm-pg3 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:42 vm-pg3 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg3:~$
asvpg@vm-pg3:~$ sudo systemctl enable pgbouncer
Synchronizing state of pgbouncer.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable pgbouncer
asvpg@vm-pg3:~$ sudo systemctl start pgbouncer
asvpg@vm-pg3:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 09:12:12 UTC; 5s ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 2190 (pgbouncer)
      Tasks: 3 (limit: 2313)
     Memory: 1.8M (peak: 2.1M)
        CPU: 5ms
     CGroup: /system.slice/pgbouncer.service
             └─2190 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 09:12:12 vm-pg3 systemd[1]: Starting pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 09:12:12 vm-pg3 pgbouncer[2190]: kernel file descriptor limit: 1024 (hard: 524288); max_client_conn: 100, max expected fd use: 172
Aug 16 09:12:12 vm-pg3 pgbouncer[2190]: listening on 10.130.0.33:6432
Aug 16 09:12:12 vm-pg3 pgbouncer[2190]: listening on unix:/tmp/.s.PGSQL.6432
Aug 16 09:12:12 vm-pg3 pgbouncer[2190]: process up: PgBouncer 1.25.2, libevent 2.1.12-stable (epoll), adns: c-ares 1.27.0, tls: OpenSSL 3.0.13 3>
Aug 16 09:12:12 vm-pg3 systemd[1]: Started pgbouncer.service - connection pooler for PostgreSQL.
lines 1-18/18 (END)
asvpg@vm-pg3:~$

asvpg@vm-pg1:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:08 UTC; 43min ago
   Duration: 34min 49.811s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 776 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 776 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
        CPU: 262ms

Aug 16 08:08:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:09:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:10:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:11:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:12:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: server connections dropped, exiting
Aug 16 08:13:08 vm-pg1 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:08 UTC; 59min ago
   Duration: 34min 49.811s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 776 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 776 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
        CPU: 262ms

Aug 16 08:08:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:09:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:10:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:11:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:12:18 vm-pg1 pgbouncer[776]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, >
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:08 vm-pg1 pgbouncer[776]: server connections dropped, exiting
Aug 16 08:13:08 vm-pg1 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:08 vm-pg1 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg1:~$ sudo systemctl enable pgbouncer
Synchronizing state of pgbouncer.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable pgbouncer
asvpg@vm-pg1:~$
asvpg@vm-pg1:~$ sudo systemctl start pgbouncer
asvpg@vm-pg1:~$ sudo systemctl enable pgbouncer
Synchronizing state of pgbouncer.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable pgbouncer
asvpg@vm-pg1:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 09:13:16 UTC; 8s ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 1773 (pgbouncer)
      Tasks: 3 (limit: 2313)
     Memory: 1.8M (peak: 2.3M)
        CPU: 9ms
     CGroup: /system.slice/pgbouncer.service
             └─1773 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 09:13:16 vm-pg1 systemd[1]: Starting pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 09:13:16 vm-pg1 pgbouncer[1773]: kernel file descriptor limit: 1024 (hard: 524288); max_client_conn: 100, max ex>
Aug 16 09:13:16 vm-pg1 pgbouncer[1773]: listening on 10.130.0.13:6432
Aug 16 09:13:16 vm-pg1 pgbouncer[1773]: listening on unix:/tmp/.s.PGSQL.6432
Aug 16 09:13:16 vm-pg1 pgbouncer[1773]: process up: PgBouncer 1.25.2, libevent 2.1.12-stable (epoll), adns: c-ares 1.27>
Aug 16 09:13:16 vm-pg1 systemd[1]: Started pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg1:~$

asvpg@vm-pg2:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:39 UTC; 42min ago
   Duration: 8min 5.067s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 1593 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 1593 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
        CPU: 62ms

Aug 16 08:09:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:10:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:11:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:12:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:13:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:13:38 vm-pg2 pgbouncer[1593]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:38 vm-pg2 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:39 vm-pg2 pgbouncer[1593]: server connections dropped, exiting
Aug 16 08:13:39 vm-pg2 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:39 vm-pg2 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg2:~$
asvpg@vm-pg2:~$
asvpg@vm-pg2:~$ sudo systemctl status pgbouncer
○ pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-08-16 08:13:39 UTC; 1h 0min ago
   Duration: 8min 5.067s
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
    Process: 1593 ExecStart=/usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini (code=exited, status=0/SUCCESS)
   Main PID: 1593 (code=exited, status=0/SUCCESS)
     Status: "stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact>
        CPU: 62ms

Aug 16 08:09:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:10:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:11:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:12:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:13:33 vm-pg2 pgbouncer[1593]: stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s,>
Aug 16 08:13:38 vm-pg2 pgbouncer[1593]: got SIGINT, shutting down, waiting for all servers connections to be released
Aug 16 08:13:38 vm-pg2 systemd[1]: Stopping pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 08:13:39 vm-pg2 pgbouncer[1593]: server connections dropped, exiting
Aug 16 08:13:39 vm-pg2 systemd[1]: pgbouncer.service: Deactivated successfully.
Aug 16 08:13:39 vm-pg2 systemd[1]: Stopped pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg2:~$ sudo systemctl enable pgbouncer
Synchronizing state of pgbouncer.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable pgbouncer
asvpg@vm-pg2:~$
asvpg@vm-pg2:~$ sudo systemctl start pgbouncer
asvpg@vm-pg2:~$
asvpg@vm-pg2:~$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-16 09:14:00 UTC; 3s ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 2307 (pgbouncer)
      Tasks: 3 (limit: 2313)
     Memory: 1.8M (peak: 2.2M)
        CPU: 7ms
     CGroup: /system.slice/pgbouncer.service
             └─2307 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Aug 16 09:14:00 vm-pg2 systemd[1]: Starting pgbouncer.service - connection pooler for PostgreSQL...
Aug 16 09:14:00 vm-pg2 pgbouncer[2307]: kernel file descriptor limit: 1024 (hard: 524288); max_client_conn: 100, max ex>
Aug 16 09:14:00 vm-pg2 pgbouncer[2307]: listening on 10.130.0.28:6432
Aug 16 09:14:00 vm-pg2 pgbouncer[2307]: listening on unix:/tmp/.s.PGSQL.6432
Aug 16 09:14:00 vm-pg2 pgbouncer[2307]: process up: PgBouncer 1.25.2, libevent 2.1.12-stable (epoll), adns: c-ares 1.27>
Aug 16 09:14:00 vm-pg2 systemd[1]: Started pgbouncer.service - connection pooler for PostgreSQL.
asvpg@vm-pg2:~$
```

####
Проверяем правильность настроек через подключение
####
```sh
--напрямую
asvpg@vm-pg3:~$ sudo -u postgres psql -p 5432 -d thai -h localhost
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=# exit
asvpg@vm-pg3:~$

--через pgbouncer
asvpg@vm-pg3:~$ sudo -u postgres psql -p 6432 -d thai -h 10.130.0.33
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=# exit
asvpg@vm-pg3:~$
```

####
Если не хочется следить за списком пользователей, которые могут использовать pgbouncer, можно в конфиге добавить опцию auth_query с запросом на вывод списка пользователей из pg_shadow
####
```sh
auth_query = SELECT usename, passwd FROM pg_shadow WHERE usename=$1
```

####
Логи изучаем здесь
####
```sh
asvpg@vm-pg3:~$ sudo tail /var/log/postgresql/pgbouncer.log
2026-08-16 09:20:05.545 UTC [2190] LOG C-0x6252920d18b0: thai/postgres@10.130.0.13:39604 login attempt: db=thai user=postgres tls=no replication=no
2026-08-16 09:20:07.621 UTC [2190] LOG C-0x6252920d18b0: thai/postgres@10.130.0.13:39612 login attempt: db=thai user=postgres tls=no replication=no
2026-08-16 09:20:12.872 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 2 B/s, xact 1689 us, query 1689 us, wait 202 us
2026-08-16 09:20:13.788 UTC [2190] LOG C-0x6252920d18b0: thai/postgres@10.130.0.13:39612 closing because: client close request (age=6s)
2026-08-16 09:21:12.871 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
2026-08-16 09:22:12.870 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
2026-08-16 09:23:12.872 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
2026-08-16 09:24:12.870 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
2026-08-16 09:25:12.871 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
2026-08-16 09:26:12.871 UTC [2190] LOG stats: 0 xacts/s, 0 queries/s, 0 client parses/s, 0 server parses/s, 0 binds/s, in 0 B/s, out 0 B/s, xact 0 us, query 0 us, wait 0 us
asvpg@vm-pg3:~$
```

####
Есть отдельная админ консоль (через SQL*Lite) для подключения пользователя, указанного в конфиге в опции admin_users
####
```sh
asvpg@vm-pg3:~$ sudo -u postgres psql -p 6432 pgbouncer -h 10.130.0.33 -U admindb
Password for user admindb:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2), server 1.25.2/bouncer)
WARNING: psql major version 17, server major version 1.25.
         Some psql features might not work.
Type "help" for help.

pgbouncer=#

pgbouncer=# show clients;
 type |  user   | database  | replication | state |    addr     | port  | local_addr  | local_port |      connect_time       |      request_time       | wait | wait_us | close_needed |      ptr       | link | remote_pid | tls | application_name | prepared_statements | id
------+---------+-----------+-------------+-------+-------------+-------+-------------+------------+-------------------------+-------------------------+------+---------+--------------+----------------+------+------------+-----+------------------+---------------------+----
 C    | admindb | pgbouncer | none        | idle  | 10.130.0.33 | 46080 | 10.130.0.33 |       6432 | 2026-08-16 09:28:58 UTC | 2026-08-16 09:31:14 UTC |    0 |       0 |            0 | 0x6252920d18b0 |      |          0 |     | psql             |                   0 |  7
(1 row)

pgbouncer=# show servers;
 type | user | database | replication | state | addr | port | local_addr | local_port | connect_time | request_time | wait | wait_us | close_needed | ptr | link | remote_pid | tls | application_name | prepared_statements | id
------+------+----------+-------------+-------+------+------+------------+------------+--------------+--------------+------+---------+--------------+-----+------+------------+-----+------------------+---------------------+----
(0 rows)

pgbouncer=# show pools;
 database  |   user    | cl_active | cl_waiting | cl_active_cancel_req | cl_waiting_cancel_req | sv_active | sv_active_cancel | sv_being_canceled | sv_idle | sv_used | sv_tested | sv_login | maxwait | maxwait_us | pool_mode | load_balance_hosts
-----------+-----------+-----------+------------+----------------------+-----------------------+-----------+------------------+-------------------+---------+---------+-----------+----------+---------+------------+-----------+--------------------
 pgbouncer | pgbouncer |         2 |          0 |                    0 |                     0 |         0 |                0 |                 0 |       0 |       0 |         0 |        0 |       0 |          0 | statement |
 thai      | postgres  |         0 |          0 |                    0 |                     0 |         0 |                0 |                 0 |       0 |       0 |         0 |        0 |       0 |          0 | session   |
(2 rows)

pgbouncer=# show stats_totals;
 database  | server_assignment_count | xact_count | query_count | bytes_received | bytes_sent | xact_time | query_time | wait_time | client_parse_count | server_parse_count | bind_count
-----------+-------------------------+------------+-------------+----------------+------------+-----------+------------+-----------+--------------------+--------------------+------------
 pgbouncer |                       0 |          2 |           2 |              0 |          0 |         0 |          0 |         0 |                  0 |                  0 |          0
 thai      |                       2 |          2 |           2 |            998 |       1311 |      5138 |       5138 |      8617 |                  0 |                  0 |          0
(2 rows)

--можно ставить на паузу будущие подключения к БД и возобновлять их
pgbouncer=# pause thai;
PAUSE
pgbouncer=# resume thai;
RESUME
pgbouncer=#
```

###
5. Настройка HAProxy
###
####
В работе с кластером БД встает вопрос: как узнать, где у нас лидер\мастер? Особенно, после смены ролей в кластере (switchover, failover). Для этого используется HAProxy. С 2025 года HAProxy стал платным, можно собирать из исходников, но в целом именно по этой причине (из-за кибербезопасников) многие ушли от использования HAProxy и в строке jdbc подключения к БД указывают server_type (primary\secondary). HAProxy является очень легковесным.
####

####
HAProxy будет работать на 2 ВМ в другой сетевой зоне (ЦОДе)
####
```sh
vm-haproxy1  10.128.0.31
vm-haproxy2  10.128.0.4
```

####
Устанавливаем HAProxy на 2 ноды
####
```sh
asvpg@vm-haproxy1:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Get:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease [126 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Packages [1191 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main Translation-en [282 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Components [180 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Packages [1683 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/universe Translation-en [335 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Components [388 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/restricted amd64 Packages [1424 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/restricted Translation-en [323 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/multiverse Translation-en [12.6 kB]
Get:14 http://mirror.yandex.ru/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:15 http://mirror.yandex.ru/ubuntu noble-backports/main amd64 Components [5740 B]
Get:16 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Components [12.6 kB]
Get:17 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [934 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [202 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [46.4 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1201 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [240 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [76.3 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [1330 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [304 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [10.9 kB]
Fetched 10.6 MB in 2s (5354 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
asvpg@vm-haproxy1:~$ sudo hostnamectl set-hostname haproxynode
asvpg@vm-haproxy1:~$ sudo apt install net-tools
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  net-tools
0 upgraded, 1 newly installed, 0 to remove and 17 not upgraded.
Need to get 204 kB of archives.
After this operation, 811 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 net-tools amd64 2.10-0.1ubuntu4.4 [204 kB]
Fetched 204 kB in 0s (6606 kB/s)
Selecting previously unselected package net-tools.
(Reading database ... 106575 files and directories currently installed.)
Preparing to unpack .../net-tools_2.10-0.1ubuntu4.4_amd64.deb ...
Unpacking net-tools (2.10-0.1ubuntu4.4) ...
Setting up net-tools (2.10-0.1ubuntu4.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy1:~$ sudo apt -y install haproxy
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  liblua5.4-0
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy liblua5.4-0
0 upgraded, 2 newly installed, 0 to remove and 17 not upgraded.
Need to get 2236 kB of archives.
After this operation, 5274 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 haproxy amd64 2.8.16-0ubuntu0.24.04.3 [2070 kB]
Fetched 2236 kB in 0s (8549 kB/s)
Selecting previously unselected package liblua5.4-0:amd64.
(Reading database ... 106623 files and directories currently installed.)
Preparing to unpack .../liblua5.4-0_5.4.6-3build2_amd64.deb ...
Unpacking liblua5.4-0:amd64 (5.4.6-3build2) ...
Selecting previously unselected package haproxy.
Preparing to unpack .../haproxy_2.8.16-0ubuntu0.24.04.3_amd64.deb ...
Unpacking haproxy (2.8.16-0ubuntu0.24.04.3) ...
Setting up liblua5.4-0:amd64 (5.4.6-3build2) ...
Setting up haproxy (2.8.16-0ubuntu0.24.04.3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/haproxy.service → /usr/lib/systemd/system/haproxy.service.
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Processing triggers for rsyslog (8.2312.0-3ubuntu9.3) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo apt update
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Get:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease [126 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease [126 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Packages [1191 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main Translation-en [282 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 Components [180 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Packages [1683 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/universe Translation-en [335 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/universe amd64 Components [388 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/restricted amd64 Packages [1424 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/restricted Translation-en [323 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/multiverse Translation-en [12.6 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:14 http://mirror.yandex.ru/ubuntu noble-backports/main amd64 Components [5740 B]
Get:15 http://mirror.yandex.ru/ubuntu noble-backports/universe amd64 Components [12.6 kB]
Get:16 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:17 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [934 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [202 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [46.4 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1201 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [240 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [76.3 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [1330 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [304 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [10.9 kB]
Fetched 10.6 MB in 31s (336 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
asvpg@vm-haproxy2:~$ sudo hostnamectl set-hostname haproxynode
asvpg@vm-haproxy2:~$ sudo apt install net-tools
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  net-tools
0 upgraded, 1 newly installed, 0 to remove and 17 not upgraded.
Need to get 204 kB of archives.
After this operation, 811 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 net-tools amd64 2.10-0.1ubuntu4.4 [204 kB]
Fetched 204 kB in 0s (5088 kB/s)
Selecting previously unselected package net-tools.
(Reading database ... 106575 files and directories currently installed.)
Preparing to unpack .../net-tools_2.10-0.1ubuntu4.4_amd64.deb ...
Unpacking net-tools (2.10-0.1ubuntu4.4) ...
Setting up net-tools (2.10-0.1ubuntu4.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy2:~$ sudo apt -y install haproxy
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  liblua5.4-0
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy liblua5.4-0
0 upgraded, 2 newly installed, 0 to remove and 17 not upgraded.
Need to get 2236 kB of archives.
After this operation, 5274 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 haproxy amd64 2.8.16-0ubuntu0.24.04.3 [2070 kB]
Fetched 2236 kB in 0s (24.7 MB/s)
Selecting previously unselected package liblua5.4-0:amd64.
(Reading database ... 106623 files and directories currently installed.)
Preparing to unpack .../liblua5.4-0_5.4.6-3build2_amd64.deb ...
Unpacking liblua5.4-0:amd64 (5.4.6-3build2) ...
Selecting previously unselected package haproxy.
Preparing to unpack .../haproxy_2.8.16-0ubuntu0.24.04.3_amd64.deb ...
Unpacking haproxy (2.8.16-0ubuntu0.24.04.3) ...
Setting up liblua5.4-0:amd64 (5.4.6-3build2) ...
Setting up haproxy (2.8.16-0ubuntu0.24.04.3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/haproxy.service → /usr/lib/systemd/system/haproxy.service.
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Processing triggers for rsyslog (8.2312.0-3ubuntu9.3) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy2:~$
```

####
Проверяем пинг с нод HAProxy на ноды PostgreSQL. Работает даже с коротким именем, но лучше использовать FQDN.
####
```sh
asvpg@vm-haproxy1:~$ ping vm-pg1
PING vm-pg1.ru-central1.internal (10.130.0.13) 56(84) bytes of data.
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=1 ttl=61 time=6.81 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=2 ttl=61 time=5.16 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=3 ttl=61 time=5.13 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=4 ttl=61 time=5.23 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=5 ttl=61 time=5.17 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=6 ttl=61 time=5.15 ms
--- vm-pg1.ru-central1.internal ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5007ms
rtt min/avg/max/mdev = 5.132/5.441/6.811/0.613 ms
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ ping vm-pg1.ru-central1.internal
PING vm-pg1.ru-central1.internal (10.130.0.13) 56(84) bytes of data.
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=1 ttl=61 time=5.09 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=2 ttl=61 time=5.14 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=3 ttl=61 time=5.28 ms
--- vm-pg1.ru-central1.internal ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 5.094/5.173/5.282/0.079 ms
asvpg@vm-haproxy1:~$ ping vm-pg2.ru-central1.internal
PING vm-pg2.ru-central1.internal (10.130.0.28) 56(84) bytes of data.
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=1 ttl=61 time=5.67 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=2 ttl=61 time=5.18 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=3 ttl=61 time=5.15 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=4 ttl=61 time=5.35 ms
--- vm-pg2.ru-central1.internal ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 5.148/5.335/5.668/0.206 ms
asvpg@vm-haproxy1:~$ ping vm-pg3.ru-central1.internal
PING vm-pg3.ru-central1.internal (10.130.0.33) 56(84) bytes of data.
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=1 ttl=61 time=5.86 ms
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=2 ttl=61 time=5.39 ms
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=3 ttl=61 time=5.06 ms
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=4 ttl=61 time=5.15 ms
q64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=5 ttl=61 time=5.18 ms
--- vm-pg3.ru-central1.internal ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 5.061/5.327/5.859/0.286 ms
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ ping vm-pg1.ru-central1.internal
PING vm-pg1.ru-central1.internal (10.130.0.13) 56(84) bytes of data.
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=1 ttl=61 time=5.78 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=2 ttl=61 time=5.12 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=3 ttl=61 time=5.16 ms
64 bytes from vm-pg1.ru-central1.internal (10.130.0.13): icmp_seq=4 ttl=61 time=5.11 ms
--- vm-pg1.ru-central1.internal ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 5.114/5.289/5.775/0.280 ms
asvpg@vm-haproxy2:~$ ping vm-pg2.ru-central1.internal
PING vm-pg2.ru-central1.internal (10.130.0.28) 56(84) bytes of data.
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=1 ttl=61 time=5.83 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=2 ttl=61 time=5.11 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=3 ttl=61 time=5.15 ms
64 bytes from vm-pg2.ru-central1.internal (10.130.0.28): icmp_seq=4 ttl=61 time=5.19 ms
--- vm-pg2.ru-central1.internal ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 5.109/5.318/5.827/0.295 ms
asvpg@vm-haproxy2:~$ ping vm-pg3.ru-central1.internal
PING vm-pg3.ru-central1.internal (10.130.0.33) 56(84) bytes of data.
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=1 ttl=61 time=5.81 ms
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=2 ttl=61 time=5.33 ms
64 bytes from vm-pg3.ru-central1.internal (10.130.0.33): icmp_seq=3 ttl=61 time=5.21 ms
--- vm-pg3.ru-central1.internal ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 5.205/5.447/5.812/0.262 ms
asvpg@vm-haproxy2:~$


--Выполним curl и убедимся, что по внутренней сети Patroni доступен
asvpg@vm-haproxy1:~$ curl -v 10.130.0.13:8008/master
*   Trying 10.130.0.13:8008...
* Connected to 10.130.0.13 (10.130.0.13) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.13:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 503 Service Unavailable
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:04:45 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 07:38:24.299546+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637587944, "replayed_location": 637587944, "replayed_timestamp": "2026-08-16 08:43:49.708247+00:00", "paused": false}, "timeline": 11, "replication_state": "streaming", "dcs_last_seen": 1786874684, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg1"}}asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ curl -v 10.130.0.28:8008/master
*   Trying 10.130.0.28:8008...
* Connected to 10.130.0.28 (10.130.0.28) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.28:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 503 Service Unavailable
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:04:54 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 08:02:22.287104+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637587944, "replayed_location": 637587944, "replayed_timestamp": "2026-08-16 08:43:49.708247+00:00", "paused": false}, "timeline": 11, "replication_state": "streaming", "dcs_last_seen": 1786874694, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg2"}}asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ curl -v 10.130.0.33:8008/master
*   Trying 10.130.0.33:8008...
* Connected to 10.130.0.33 (10.130.0.33) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.33:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:05:00 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 07:38:20.379850+00:00", "role": "primary", "server_version": 170011, "xlog": {"location": 637587944}, "timeline": 11, "replication": [{"usename": "repl_user", "application_name": "vm-pg1", "client_addr": "10.130.0.13", "state": "streaming", "sync_state": "async", "sync_priority": 0}, {"usename": "repl_user", "application_name": "vm-pg2", "client_addr": "10.130.0.28", "state": "streaming", "sync_state": "async", "sync_priority": 0}], "dcs_last_seen": 1786874693, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg3"}}asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$


asvpg@vm-haproxy2:~$ curl -v 10.130.0.13:8008/master
*   Trying 10.130.0.13:8008...
* Connected to 10.130.0.13 (10.130.0.13) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.13:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 503 Service Unavailable
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:06:03 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 07:38:24.299546+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637587944, "replayed_location": 637587944, "replayed_timestamp": "2026-08-16 08:43:49.708247+00:00", "paused": false}, "timeline": 11, "replication_state": "streaming", "dcs_last_seen": 1786874754, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg1"}}asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$ curl -v 10.130.0.28:8008/master
*   Trying 10.130.0.28:8008...
* Connected to 10.130.0.28 (10.130.0.28) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.28:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 503 Service Unavailable
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:06:18 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 08:02:22.287104+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637587944, "replayed_location": 637587944, "replayed_timestamp": "2026-08-16 08:43:49.708247+00:00", "paused": false}, "timeline": 11, "replication_state": "streaming", "dcs_last_seen": 1786874774, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg2"}}asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$ curl -v 10.130.0.33:8008/master
*   Trying 10.130.0.33:8008...
* Connected to 10.130.0.33 (10.130.0.33) port 8008
> GET /master HTTP/1.1
> Host: 10.130.0.33:8008
> User-Agent: curl/8.5.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Server: BaseHTTP/0.6 Python/3.12.3
< Date: Sun, 16 Aug 2026 10:06:22 GMT
< Content-Type: application/json
<
* Closing connection
{"state": "running", "postmaster_start_time": "2026-08-16 07:38:20.379850+00:00", "role": "primary", "server_version": 170011, "xlog": {"location": 637587944}, "timeline": 11, "replication": [{"usename": "repl_user", "application_name": "vm-pg1", "client_addr": "10.130.0.13", "state": "streaming", "sync_state": "async", "sync_priority": 0}, {"usename": "repl_user", "application_name": "vm-pg2", "client_addr": "10.130.0.28", "state": "streaming", "sync_state": "async", "sync_priority": 0}], "dcs_last_seen": 1786874773, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg3"}}asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$
```

####
Проверим доступ, установив на нодах HAProxy ПО PostgreSQL (должен быть доступ в другой ЦОД по внутренней сети)
####
```sh
asvpg@vm-haproxy1:~$ sudo apt update && sudo apt upgrade -y && sudo apt install -y postgresql-client-common && sudo apt install postgresql-client -y
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Hit:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  google-compute-engine-oslogin
The following packages will be upgraded:
  krb5-locales libgssapi-krb5-2 libk5crypto3 libkrb5-3 libkrb5support0 libnss-systemd libpam-systemd libsystemd-shared libsystemd0
  libudev1 systemd systemd-dev systemd-resolved systemd-sysv systemd-timesyncd udev
16 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
11 standard LTS security updates
Need to get 9504 kB of archives.
After this operation, 18.4 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libnss-systemd amd64 255.4-1ubuntu8.17 [159 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-dev all 255.4-1ubuntu8.17 [107 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-timesyncd amd64 255.4-1ubuntu8.17 [35.3 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-resolved amd64 255.4-1ubuntu8.17 [297 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd-shared amd64 255.4-1ubuntu8.17 [2077 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd0 amd64 255.4-1ubuntu8.17 [432 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-sysv amd64 255.4-1ubuntu8.17 [11.9 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libpam-systemd amd64 255.4-1ubuntu8.17 [235 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd amd64 255.4-1ubuntu8.17 [3475 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 udev amd64 255.4-1ubuntu8.17 [1875 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libudev1 amd64 255.4-1ubuntu8.17 [178 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 krb5-locales all 1.20.1-6ubuntu2.8 [15.1 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libgssapi-krb5-2 amd64 1.20.1-6ubuntu2.8 [143 kB]
Get:14 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5-3 amd64 1.20.1-6ubuntu2.8 [348 kB]
Get:15 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5support0 amd64 1.20.1-6ubuntu2.8 [34.7 kB]
Get:16 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libk5crypto3 amd64 1.20.1-6ubuntu2.8 [81.9 kB]
Fetched 9504 kB in 0s (87.4 MB/s)
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../0-libnss-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libnss-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../1-systemd-dev_255.4-1ubuntu8.17_all.deb ...
Unpacking systemd-dev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../2-systemd-timesyncd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-timesyncd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../3-systemd-resolved_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-resolved (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../4-libsystemd-shared_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd-shared:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../5-libsystemd0_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd0:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libsystemd0:amd64 (255.4-1ubuntu8.17) ...
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../systemd-sysv_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-sysv (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libpam-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libpam-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../udev_255.4-1ubuntu8.17_amd64.deb ...
Unpacking udev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libudev1_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libudev1:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libudev1:amd64 (255.4-1ubuntu8.17) ...
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../krb5-locales_1.20.1-6ubuntu2.8_all.deb ...
Unpacking krb5-locales (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libgssapi-krb5-2_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5-3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5support0_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libk5crypto3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Setting up systemd-dev (255.4-1ubuntu8.17) ...
Setting up krb5-locales (1.20.1-6ubuntu2.8) ...
Setting up libkrb5support0:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libsystemd-shared:amd64 (255.4-1ubuntu8.17) ...
Setting up libk5crypto3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libkrb5-3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up systemd (255.4-1ubuntu8.17) ...
Setting up systemd-timesyncd (255.4-1ubuntu8.17) ...
Setting up libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) ...
Setting up udev (255.4-1ubuntu8.17) ...
Setting up systemd-resolved (255.4-1ubuntu8.17) ...
Setting up systemd-sysv (255.4-1ubuntu8.17) ...
Setting up libnss-systemd:amd64 (255.4-1ubuntu8.17) ...
Setting up libpam-systemd:amd64 (255.4-1ubuntu8.17) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for initramfs-tools (0.142ubuntu25.8) ...
update-initramfs: Generating /boot/initrd.img-6.8.0-137-generic
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart fwupd.service haproxy.service multipathd.service packagekit.service polkit.service rsyslog.service ssh.service udisks2.service

Service restarts being deferred:
 systemctl restart ModemManager.service
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: apt[2358], sshd[1078]
 asvpg @ user manager service: systemd[1088]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  postgresql-client-common
0 upgraded, 1 newly installed, 0 to remove and 1 not upgraded.
Need to get 36.4 kB of archives.
After this operation, 134 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client-common all 257build1.1 [36.4 kB]
Fetched 36.4 kB in 0s (1280 kB/s)
Selecting previously unselected package postgresql-client-common.
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../postgresql-client-common_257build1.1_all.deb ...
Unpacking postgresql-client-common (257build1.1) ...
Setting up postgresql-client-common (257build1.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1078]
 asvpg @ user manager service: systemd[1088]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libpq5 postgresql-client-16
Suggested packages:
  postgresql-16 postgresql-doc-16
The following NEW packages will be installed:
  libpq5 postgresql-client postgresql-client-16
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 1458 kB of archives.
After this operation, 4830 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libpq5 amd64 16.14-0ubuntu0.24.04.1 [147 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client-16 amd64 16.14-0ubuntu0.24.04.1 [1300 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client all 16+257build1.1 [11.6 kB]
Fetched 1458 kB in 0s (18.2 MB/s)
Selecting previously unselected package libpq5:amd64.
(Reading database ... 106759 files and directories currently installed.)
Preparing to unpack .../libpq5_16.14-0ubuntu0.24.04.1_amd64.deb ...
Unpacking libpq5:amd64 (16.14-0ubuntu0.24.04.1) ...
Selecting previously unselected package postgresql-client-16.
Preparing to unpack .../postgresql-client-16_16.14-0ubuntu0.24.04.1_amd64.deb ...
Unpacking postgresql-client-16 (16.14-0ubuntu0.24.04.1) ...
Selecting previously unselected package postgresql-client.
Preparing to unpack .../postgresql-client_16+257build1.1_all.deb ...
Unpacking postgresql-client (16+257build1.1) ...
Setting up libpq5:amd64 (16.14-0ubuntu0.24.04.1) ...
Setting up postgresql-client-16 (16.14-0ubuntu0.24.04.1) ...
update-alternatives: using /usr/share/postgresql/16/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
Setting up postgresql-client (16+257build1.1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1078]
 asvpg @ user manager service: systemd[1088]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy1:~$


asvpg@vm-haproxy1:~$ psql -p 6432 -d thai -h 10.130.0.33 -U postgres
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#


asvpg@vm-haproxy2:~$ sudo apt update && sudo apt upgrade -y && sudo apt install -y postgresql-client-common && sudo apt install postgresql-client -y
Hit:1 http://mirror.yandex.ru/ubuntu noble InRelease
Hit:2 http://mirror.yandex.ru/ubuntu noble-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  google-compute-engine-oslogin
The following packages will be upgraded:
  krb5-locales libgssapi-krb5-2 libk5crypto3 libkrb5-3 libkrb5support0 libnss-systemd libpam-systemd libsystemd-shared
  libsystemd0 libudev1 systemd systemd-dev systemd-resolved systemd-sysv systemd-timesyncd udev
16 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
11 standard LTS security updates
Need to get 9504 kB of archives.
After this operation, 18.4 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libnss-systemd amd64 255.4-1ubuntu8.17 [159 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-dev all 255.4-1ubuntu8.17 [107 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-timesyncd amd64 255.4-1ubuntu8.17 [35.3 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-resolved amd64 255.4-1ubuntu8.17 [297 kB]
Get:5 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd-shared amd64 255.4-1ubuntu8.17 [2077 kB]
Get:6 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsystemd0 amd64 255.4-1ubuntu8.17 [432 kB]
Get:7 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd-sysv amd64 255.4-1ubuntu8.17 [11.9 kB]
Get:8 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libpam-systemd amd64 255.4-1ubuntu8.17 [235 kB]
Get:9 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 systemd amd64 255.4-1ubuntu8.17 [3475 kB]
Get:10 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 udev amd64 255.4-1ubuntu8.17 [1875 kB]
Get:11 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libudev1 amd64 255.4-1ubuntu8.17 [178 kB]
Get:12 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 krb5-locales all 1.20.1-6ubuntu2.8 [15.1 kB]
Get:13 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libgssapi-krb5-2 amd64 1.20.1-6ubuntu2.8 [143 kB]
Get:14 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5-3 amd64 1.20.1-6ubuntu2.8 [348 kB]
Get:15 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libkrb5support0 amd64 1.20.1-6ubuntu2.8 [34.7 kB]
Get:16 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libk5crypto3 amd64 1.20.1-6ubuntu2.8 [81.9 kB]
Fetched 9504 kB in 0s (95.7 MB/s)
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../0-libnss-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libnss-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../1-systemd-dev_255.4-1ubuntu8.17_all.deb ...
Unpacking systemd-dev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../2-systemd-timesyncd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-timesyncd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../3-systemd-resolved_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-resolved (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../4-libsystemd-shared_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd-shared:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../5-libsystemd0_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libsystemd0:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libsystemd0:amd64 (255.4-1ubuntu8.17) ...
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../systemd-sysv_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd-sysv (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libpam-systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libpam-systemd:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../systemd_255.4-1ubuntu8.17_amd64.deb ...
Unpacking systemd (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../udev_255.4-1ubuntu8.17_amd64.deb ...
Unpacking udev (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Preparing to unpack .../libudev1_255.4-1ubuntu8.17_amd64.deb ...
Unpacking libudev1:amd64 (255.4-1ubuntu8.17) over (255.4-1ubuntu8.16) ...
Setting up libudev1:amd64 (255.4-1ubuntu8.17) ...
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../krb5-locales_1.20.1-6ubuntu2.8_all.deb ...
Unpacking krb5-locales (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libgssapi-krb5-2_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5-3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libkrb5support0_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Preparing to unpack .../libk5crypto3_1.20.1-6ubuntu2.8_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.20.1-6ubuntu2.8) over (1.20.1-6ubuntu2.7) ...
Setting up systemd-dev (255.4-1ubuntu8.17) ...
Setting up krb5-locales (1.20.1-6ubuntu2.8) ...
Setting up libkrb5support0:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libsystemd-shared:amd64 (255.4-1ubuntu8.17) ...
Setting up libk5crypto3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up libkrb5-3:amd64 (1.20.1-6ubuntu2.8) ...
Setting up systemd (255.4-1ubuntu8.17) ...
Setting up systemd-timesyncd (255.4-1ubuntu8.17) ...
Setting up libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8) ...
Setting up udev (255.4-1ubuntu8.17) ...
Setting up systemd-resolved (255.4-1ubuntu8.17) ...
Setting up systemd-sysv (255.4-1ubuntu8.17) ...
Setting up libnss-systemd:amd64 (255.4-1ubuntu8.17) ...
Setting up libpam-systemd:amd64 (255.4-1ubuntu8.17) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for initramfs-tools (0.142ubuntu25.8) ...
update-initramfs: Generating /boot/initrd.img-6.8.0-137-generic
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart fwupd-refresh.service fwupd.service haproxy.service multipathd.service packagekit.service polkit.service rsyslog.service ssh.service udisks2.service
Job for fwupd-refresh.service failed because the control process exited with error code.
See "systemctl status fwupd-refresh.service" and "journalctl -xeu fwupd-refresh.service" for details.

Service restarts being deferred:
 systemctl restart ModemManager.service
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: apt[2378], sshd[1070]
 asvpg @ user manager service: systemd[1080]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  postgresql-client-common
0 upgraded, 1 newly installed, 0 to remove and 1 not upgraded.
Need to get 36.4 kB of archives.
After this operation, 134 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client-common all 257build1.1 [36.4 kB]
Fetched 36.4 kB in 0s (507 kB/s)
Selecting previously unselected package postgresql-client-common.
(Reading database ... 106723 files and directories currently installed.)
Preparing to unpack .../postgresql-client-common_257build1.1_all.deb ...
Unpacking postgresql-client-common (257build1.1) ...
Setting up postgresql-client-common (257build1.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1070]
 asvpg @ user manager service: systemd[1080]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libpq5 postgresql-client-16
Suggested packages:
  postgresql-16 postgresql-doc-16
The following NEW packages will be installed:
  libpq5 postgresql-client postgresql-client-16
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 1458 kB of archives.
After this operation, 4830 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libpq5 amd64 16.14-0ubuntu0.24.04.1 [147 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client-16 amd64 16.14-0ubuntu0.24.04.1 [1300 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 postgresql-client all 16+257build1.1 [11.6 kB]
Fetched 1458 kB in 0s (24.6 MB/s)
Selecting previously unselected package libpq5:amd64.
(Reading database ... 106759 files and directories currently installed.)
Preparing to unpack .../libpq5_16.14-0ubuntu0.24.04.1_amd64.deb ...
Unpacking libpq5:amd64 (16.14-0ubuntu0.24.04.1) ...
Selecting previously unselected package postgresql-client-16.
Preparing to unpack .../postgresql-client-16_16.14-0ubuntu0.24.04.1_amd64.deb ...
Unpacking postgresql-client-16 (16.14-0ubuntu0.24.04.1) ...
Selecting previously unselected package postgresql-client.
Preparing to unpack .../postgresql-client_16+257build1.1_all.deb ...
Unpacking postgresql-client (16+257build1.1) ...
Setting up libpq5:amd64 (16.14-0ubuntu0.24.04.1) ...
Setting up postgresql-client-16 (16.14-0ubuntu0.24.04.1) ...
update-alternatives: using /usr/share/postgresql/16/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
Setting up postgresql-client (16+257build1.1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1070]
 asvpg @ user manager service: systemd[1080]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy2:~$

asvpg@vm-haproxy2:~$ psql -p 6432 -d thai -h 10.130.0.33 -U postgres
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#
```

####
Далее настраиваем конфиг HAProxy. В нем прописываем частоту опросов на роль ноды, выполняем проверку портов (6432 - для данных; 8008 - для ответов).
Проверяем 3 сервера на мастер и 3 сервера на реплику. Как только роль ноды меняется, HAProxy автоматически это проверяет и перенаправляет новые соединения с учетом смены роли. При этом старые соединения будут оставаться на старой ноде и поэтому приложение должно ретраить запросы при разрыве соединения.
####
```sh
asvpg@vm-haproxy1:~$ sudo bash
root@haproxynode:/home/asvpg# cd /etc/haproxy/
root@haproxynode:/etc/haproxy# ls -altr
total 16
-rw-r--r--   1 root root 1285 Dec  3  2025 haproxy.cfg
drwxr-xr-x   2 root root 4096 Aug 16 09:56 errors
drwxr-xr-x 108 root root 4096 Aug 16 10:11 ..
drwxr-xr-x   3 root root 4096 Aug 16 10:19 .
root@haproxynode:/etc/haproxy# cp haproxy.cfg haproxy.cfg.default
root@haproxynode:/etc/haproxy# rm haproxy.cfg
root@haproxynode:/etc/haproxy# touch haproxy.cfg
root@haproxynode:/etc/haproxy# ls -altr
total 16
drwxr-xr-x   2 root root 4096 Aug 16 09:56 errors
drwxr-xr-x 108 root root 4096 Aug 16 10:11 ..
-rw-r--r--   1 root root 1285 Aug 16 10:20 haproxy.cfg.default
-rw-r--r--   1 root root    0 Aug 16 10:20 haproxy.cfg
drwxr-xr-x   3 root root 4096 Aug 16 10:20 .
root@haproxynode:/etc/haproxy# nano haproxy.cfg
root@haproxynode:/etc/haproxy# cat haproxy.cfg
listen postgres_write
    bind *:5433
    mode            tcp
    option httpchk
    http-check connect
    http-check send meth GET uri /master
    http-check expect status 200
    default-server inter 10s fall 3 rise 3 on-marked-down shutdown-sessions
    server vm-pg1 10.130.0.13:6432 check port 8008
    server vm-pg2 10.130.0.28:6432 check port 8008
    server vm-pg3 10.130.0.33:6432 check port 8008

listen postgres_read
    bind *:5434
    mode            tcp
    http-check connect
    http-check send meth GET uri /replica
    http-check expect status 200
    default-server inter 10s fall 3 rise 3 on-marked-down shutdown-sessions
    server vm-pg1 10.130.0.13:6432 check port 8008
    server vm-pg2 10.130.0.28:6432 check port 8008
    server vm-pg3 10.130.0.33:6432 check port 8008
root@haproxynode:/etc/haproxy#

asvpg@vm-haproxy2:~$ sudo  bash
root@haproxynode:/home/asvpg# cd /etc/haproxy/
root@haproxynode:/etc/haproxy# ls -altr
total 16
-rw-r--r--   1 root root 1285 Dec  3  2025 haproxy.cfg
drwxr-xr-x   2 root root 4096 Aug 16 09:58 errors
drwxr-xr-x   3 root root 4096 Aug 16 09:58 .
drwxr-xr-x 108 root root 4096 Aug 16 10:12 ..
root@haproxynode:/etc/haproxy# cp haproxy.cfg haproxy.cfg.default
root@haproxynode:/etc/haproxy# rm haproxy.cfg
root@haproxynode:/etc/haproxy# touch haproxy.cfg
root@haproxynode:/etc/haproxy# nano haproxy.cfg
root@haproxynode:/etc/haproxy# cat haproxy.cfg
 bind *:5433
    mode            tcp
    option httpchk
    http-check connect
    http-check send meth GET uri /master
    http-check expect status 200
    default-server inter 10s fall 3 rise 3 on-marked-down shutdown-sessions
    server vm-pg1 10.130.0.13:6432 check port 8008
    server vm-pg2 10.130.0.28:6432 check port 8008
    server vm-pg3 10.130.0.33:6432 check port 8008

listen postgres_read
    bind *:5434
    mode            tcp
    http-check connect
    http-check send meth GET uri /replica
    http-check expect status 200
    default-server inter 10s fall 3 rise 3 on-marked-down shutdown-sessions
    server vm-pg1 10.130.0.13:6432 check port 8008
    server vm-pg2 10.130.0.28:6432 check port 8008
    server vm-pg3 10.130.0.33:6432 check port 8008
root@haproxynode:/etc/haproxy#
```

####
Выполняем рестарт налету. HAProxy и pgbouncer в целом предназначены для частой смены конфигов, поэтому можно делать без предварительного останова сервиса, в отличие от etcd
####
```sh
asvpg@vm-haproxy1:~$ sudo systemctl restart haproxy.service
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ sudo systemctl status haproxy.service
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-17 16:10:19 UTC; 2min 30s ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
   Main PID: 1455 (haproxy)
     Status: "Ready."
      Tasks: 3 (limit: 2313)
     Memory: 39.1M (peak: 39.6M)
        CPU: 124ms
     CGroup: /system.slice/haproxy.service
             ├─1455 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─1457 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

Aug 17 16:10:19 haproxynode haproxy[1455]: [WARNING]  (1455) : config : proxy 'postgres_read' uses http-check rules without 'option >
Aug 17 16:10:19 haproxynode haproxy[1455]: [WARNING]  (1455) : config : missing timeouts for proxy 'postgres_read'.
Aug 17 16:10:19 haproxynode haproxy[1455]:    | While not properly invalid, you will certainly encounter various problems
Aug 17 16:10:19 haproxynode haproxy[1455]:    | with such a configuration. To fix this, please ensure that all following
Aug 17 16:10:19 haproxynode haproxy[1455]:    | timeouts are set to a non-zero value: 'client', 'connect', 'server'.
Aug 17 16:10:19 haproxynode haproxy[1455]: [NOTICE]   (1455) : New worker (1457) forked
Aug 17 16:10:19 haproxynode haproxy[1455]: [NOTICE]   (1455) : Loading success.
Aug 17 16:10:19 haproxynode systemd[1]: Started haproxy.service - HAProxy Load Balancer.
Aug 17 16:10:21 haproxynode haproxy[1457]: [WARNING]  (1457) : Server postgres_write/vm-pg2 is DOWN, reason: Layer7 wrong status, co>
Aug 17 16:10:22 haproxynode haproxy[1457]: [WARNING]  (1457) : Server postgres_write/vm-pg3 is DOWN, reason: Layer7 wrong status, co>
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo systemctl restart haproxy.service
asvpg@vm-haproxy2:~$
asvpg@haproxynode:~$ sudo systemctl status haproxy.service
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-17 16:11:52 UTC; 8s ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
   Main PID: 1381 (haproxy)
     Status: "Ready."
      Tasks: 3 (limit: 2313)
     Memory: 39.1M (peak: 39.4M)
        CPU: 109ms
     CGroup: /system.slice/haproxy.service
             ├─1381 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─1385 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

Aug 17 16:11:52 haproxynode haproxy[1381]: [WARNING]  (1381) : config : proxy 'postgres_read' uses http-check rules wit>
Aug 17 16:11:52 haproxynode haproxy[1381]: [WARNING]  (1381) : config : missing timeouts for proxy 'postgres_read'.
Aug 17 16:11:52 haproxynode haproxy[1381]:    | While not properly invalid, you will certainly encounter various proble>
Aug 17 16:11:52 haproxynode haproxy[1381]:    | with such a configuration. To fix this, please ensure that all following
Aug 17 16:11:52 haproxynode haproxy[1381]:    | timeouts are set to a non-zero value: 'client', 'connect', 'server'.
Aug 17 16:11:52 haproxynode haproxy[1381]: [NOTICE]   (1381) : New worker (1385) forked
Aug 17 16:11:52 haproxynode haproxy[1381]: [NOTICE]   (1381) : Loading success.
Aug 17 16:11:52 haproxynode systemd[1]: Started haproxy.service - HAProxy Load Balancer.
Aug 17 16:11:53 haproxynode haproxy[1385]: [WARNING]  (1385) : Server postgres_write/vm-pg2 is DOWN, reason: Layer7 wro>
Aug 17 16:11:55 haproxynode haproxy[1385]: [WARNING]  (1385) : Server postgres_write/vm-pg3 is DOWN, reason: Layer7 wro>
asvpg@vm-haproxy2:~$

--WARNING для нод pgsql1 и pgsql2 логичны, т.к. это реплики.
```

####
Проверим подключение через прокси. Указываем порт 5433, а прокси уже перенаправляет подключение через 6432 
####
```sh
asvpg@vm-haproxy1:~$ psql -h localhost -d thai -U postgres -p 5433
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#

asvpg@vm-haproxy2:~$ psql -h localhost -d thai -U postgres -p 5433
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#

--Мы попадаем на мастер
thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#
```

####
Выполним переключение ролей и проверим, что будет с открытой сессией через прокси к БД
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 11 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml switchover
Current cluster topology
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 11 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
Primary [vm-pg3]:
Candidate ['vm-pg1', 'vm-pg2'] []: vm-pg1
When should the switchover take place (e.g. 2026-08-16T11:39 )  [now]:
Are you sure you want to switchover cluster patroni, demoting current leader vm-pg3? [y/N]: y
2026-08-16 10:40:03.76910 Successfully switched over to "vm-pg1"
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running | 11 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | running | 11 |  0/2600D330 |   0 | 0/2600D330 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | stopped |    |     unknown |     |    unknown |     |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   | 12 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 12 |  0/2600E028 |   0 | 0/2600E028 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 12 |  0/2600E028 |   0 | 0/2600E028 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

--смотрим сессии на 2 нодах HAProxy и видим, что при попытке выполнить новый запрос вызывается переподключение
thai=# select 1;
FATAL:  terminating connection due to administrator command
FATAL:  server conn crashed?
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=# select pg_is_in_recovery();
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=#
thai=#
thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#


thai=# select 1;
FATAL:  terminating connection due to administrator command
FATAL:  server conn crashed?
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=# select 1;
 ?column?
----------
        1
(1 row)

thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#
```

####
Проверка, что попадаем на реплику:
####
```sh
asvpg@haproxynode:~$ psql -h localhost -d thai -U postgres -p 5434
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)

thai=#
```

###
Настройка keepalived на нодах с HAProxy
###

####
keepalived - единая точка входа. Держать белый IP адрес, на который ходят клиенты. Если первая ВМ умирает, то с этим же IP адресом начинает работать вторая ВМ. Т.е. есть один IP адрес для всех, а второй для VRRP протокола.
keepalived не работает в Google облаке, но в Яндексе работает.

Установим сервис
####
```sh
asvpg@vm-haproxy1:~$ sudo apt install -y keepalived
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ipvsadm libsnmp-base libsnmp40t64
Suggested packages:
  heartbeat ldirectord snmp-mibs-downloader
The following NEW packages will be installed:
  ipvsadm keepalived libsnmp-base libsnmp40t64
0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.
Need to get 1773 kB of archives.
After this operation, 5941 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsnmp-base all 5.9.4+dfsg-1.1ubuntu3.2 [206 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsnmp40t64 amd64 5.9.4+dfsg-1.1ubuntu3.2 [1066 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 keepalived amd64 1:2.2.8-1build2 [460 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 ipvsadm amd64 1:1.31-1ubuntu0.1 [40.3 kB]
Fetched 1773 kB in 0s (20.3 MB/s)
Selecting previously unselected package libsnmp-base.
(Reading database ... 107027 files and directories currently installed.)
Preparing to unpack .../libsnmp-base_5.9.4+dfsg-1.1ubuntu3.2_all.deb ...
Unpacking libsnmp-base (5.9.4+dfsg-1.1ubuntu3.2) ...
Selecting previously unselected package libsnmp40t64:amd64.
Preparing to unpack .../libsnmp40t64_5.9.4+dfsg-1.1ubuntu3.2_amd64.deb ...
Unpacking libsnmp40t64:amd64 (5.9.4+dfsg-1.1ubuntu3.2) ...
Selecting previously unselected package keepalived.
Preparing to unpack .../keepalived_1%3a2.2.8-1build2_amd64.deb ...
Unpacking keepalived (1:2.2.8-1build2) ...
Selecting previously unselected package ipvsadm.
Preparing to unpack .../ipvsadm_1%3a1.31-1ubuntu0.1_amd64.deb ...
Unpacking ipvsadm (1:1.31-1ubuntu0.1) ...
Setting up ipvsadm (1:1.31-1ubuntu0.1) ...
Setting up libsnmp-base (5.9.4+dfsg-1.1ubuntu3.2) ...
Setting up libsnmp40t64:amd64 (5.9.4+dfsg-1.1ubuntu3.2) ...
Setting up keepalived (1:2.2.8-1build2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /usr/lib/systemd/system/keepalived.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1078]
 asvpg @ user manager service: systemd[1088]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo apt install -y keepalived
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ipvsadm libsnmp-base libsnmp40t64
Suggested packages:
  heartbeat ldirectord snmp-mibs-downloader
The following NEW packages will be installed:
  ipvsadm keepalived libsnmp-base libsnmp40t64
0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.
Need to get 1773 kB of archives.
After this operation, 5941 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsnmp-base all 5.9.4+dfsg-1.1ubuntu3.2 [206 kB]
Get:2 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 libsnmp40t64 amd64 5.9.4+dfsg-1.1ubuntu3.2 [1066 kB]
Get:3 http://mirror.yandex.ru/ubuntu noble/main amd64 keepalived amd64 1:2.2.8-1build2 [460 kB]
Get:4 http://mirror.yandex.ru/ubuntu noble-updates/main amd64 ipvsadm amd64 1:1.31-1ubuntu0.1 [40.3 kB]
Fetched 1773 kB in 0s (15.8 MB/s)
Selecting previously unselected package libsnmp-base.
(Reading database ... 107027 files and directories currently installed.)
Preparing to unpack .../libsnmp-base_5.9.4+dfsg-1.1ubuntu3.2_all.deb ...
Unpacking libsnmp-base (5.9.4+dfsg-1.1ubuntu3.2) ...
Selecting previously unselected package libsnmp40t64:amd64.
Preparing to unpack .../libsnmp40t64_5.9.4+dfsg-1.1ubuntu3.2_amd64.deb ...
Unpacking libsnmp40t64:amd64 (5.9.4+dfsg-1.1ubuntu3.2) ...
Selecting previously unselected package keepalived.
Preparing to unpack .../keepalived_1%3a2.2.8-1build2_amd64.deb ...
Unpacking keepalived (1:2.2.8-1build2) ...
Selecting previously unselected package ipvsadm.
Preparing to unpack .../ipvsadm_1%3a1.31-1ubuntu0.1_amd64.deb ...
Unpacking ipvsadm (1:1.31-1ubuntu0.1) ...
Setting up ipvsadm (1:1.31-1ubuntu0.1) ...
Setting up libsnmp-base (5.9.4+dfsg-1.1ubuntu3.2) ...
Setting up libsnmp40t64:amd64 (5.9.4+dfsg-1.1ubuntu3.2) ...
Setting up keepalived (1:2.2.8-1build2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /usr/lib/systemd/system/keepalived.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

User sessions running outdated binaries:
 asvpg @ session #1: sshd[1070]
 asvpg @ user manager service: systemd[1080]

No VM guests are running outdated hypervisor (qemu) binaries on this host.
asvpg@vm-haproxy2:~$
```

####
В настройках необходимо задать приоритет ноде, чем выше приоритет, тем раньше нода поднимает белый IP адрес.
####
```sh
--сначала необходимо настроить ядро Linux, по умолчанию оно запрещает программе занимать IP-адрес, который ей не принадлежит физически. Эта опция разрешает HAProxy слушать Virtual IP (VIP), пока тот еще "припаркован" у соседа или находится в пути.
asvpg@vm-haproxy1:~$ sudo sysctl -p
net.ipv4.ip_nonlocal_bind = 1
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo nano /etc/sysctl.conf
asvpg@vm-haproxy2:~$ sudo sysctl -p
net.ipv4.ip_nonlocal_bind = 1
asvpg@vm-haproxy2:~$

--далее настраиваем локальные конфиги на 2 нодах через /etc/keepalived/keepalived.conf
--мастер нода
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    script "killall -0 haproxy" # Проверка наличия процесса
    interval 2                  # Каждые 2 секунды
    weight 2                    # Если ок, прибавим +2 к приоритету
    fall 2                      # Считать упавшим после 2 неудачных проверок
    rise 2                      # Считать поднявшимся после 2 успешных
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 101                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass SuperSecretPasswordHere
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
asvpg@vm-haproxy1:~$

--бэкап нода
asvpg@vm-haproxy2:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy2:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_02 # Уникальное имя
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state BACKUP               # Роль ведомого
    interface eth0             # Тот же интерфейс
    virtual_router_id 51       # Тот же ID группы!
    priority 100               # Приоритет ниже мастера

    advert_int 1
    authentication {
        auth_type PASS
        auth_pass SuperSecretPasswordHere # Пароль должен совпадать с мастером
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip
    }

    track_script {
        check_haproxy
    }
}
asvpg@vm-haproxy2:~$
```

####
Описание используемых опций\атрибутов
####
```sh
router_id	- текстовая метка для удобства чтения системных логов (journalctl -u keepalived).
vrrp_script	- блок здоровья. Keepalived запускает команду каждые interval секунд. Если команда завершилась успешно (код 0), скрипт считается пройденным.
state - (MASTER/BACKUP),	желаемое состояние. Важно: машина со статусом MASTER не обязательно станет главной сразу. Решающим фактором является Priority.
priority (приоритет) - самый главный параметр. Узел с наибольшим числом становится Master. Если Мастер падает, VIP переходит к узлу со следующим по величине приоритетом. 101 (Master) > 100 (Backup). Разрыв в единицу безопасен, так как позволяет использовать параметр weight в скрипте.
virtual_router_id (VRID) -	идентификатор группы от 0 до 255. Ноды с разными VRID не будут видеть друг друга. Должен быть уникальным внутри одной локальной сети (подсети), чтобы не конфликтовать с другими кластерами.
advert_int	- как часто Master уведомляет: «Я тут! Я держу IP!». Стандарт — 1 секунда.
authentication -	защита от того, чтобы чужой сервер в той же подсети случайно не перехватил трафик. Тип PASS небезопасен против хакеров, но защищает от ошибок администратора. Пароли должны быть одинаковыми.
virtual_ipaddress	- список адресов, которые перекидываются между машинами. Параметр label помогает визуально отличить его в выводе команды ip a.
track_script	- инструкция следить за блоком check_haproxy. Если HAProxy упал, вес приоритета уменьшается на значение weight (на 2). Так как разрыв между серверами всего 1 балл (101 − 100 = 1), падение HAProxy автоматически сделает текущего лидера проигравшим, и VIP уйдет ко второму узлу.
```

####
Выполняем старт и проверку
####
```sh
asvpg@vm-haproxy1:~$ sudo systemctl enable keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
asvpg@vm-haproxy1:~$ sudo systemctl start keepalived
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: There are 1 missing '}'s or extra '{'s
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:28:20 haproxynode Keepalived[8788]: Startup complete
Aug 16 11:28:20 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: VRRP_Script(check_haproxy) succeeded
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (VI_01) Changing effective priority from 101 to 103
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8789]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: There are 1 missing '}'s or extra '{'s
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:28:20 haproxynode Keepalived[8788]: Startup complete
Aug 16 11:28:20 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: VRRP_Script(check_haproxy) succeeded
Aug 16 11:28:20 haproxynode Keepalived_vrrp[8789]: (VI_01) Changing effective priority from 101 to 103
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8789]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$


asvpg@vm-haproxy2:~$ sudo systemctl enable keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
asvpg@vm-haproxy2:~$ sudo systemctl start keepalived
asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: Script user 'keepalived_script' does not exist
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:28:24 haproxynode Keepalived[8396]: Startup complete
Aug 16 11:28:24 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: VRRP_Script(check_haproxy) succeeded
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Changing effective priority from 100 to 102
Aug 16 11:28:27 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: Script user 'keepalived_script' does not exist
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:28:24 haproxynode Keepalived[8396]: Startup complete
Aug 16 11:28:24 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: VRRP_Script(check_haproxy) succeeded
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Changing effective priority from 100 to 102
Aug 16 11:28:27 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: Script user 'keepalived_script' does not exist
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:28:24 haproxynode Keepalived[8396]: Startup complete
Aug 16 11:28:24 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: VRRP_Script(check_haproxy) succeeded
Aug 16 11:28:24 haproxynode Keepalived_vrrp[8397]: (VI_01) Changing effective priority from 100 to 102
Aug 16 11:28:27 haproxynode Keepalived_vrrp[8397]: (VI_01) Entering MASTER STATE

asvpg@vm-haproxy2:~$
```

####
Есть критические ошибки. Обе ноды стали MASTER (split brain!). 
Видимо разница приоритетов (101 и 100) слишком мала для облака с возможными задержками сети. Пакет объявления от второй ноды пришел чуть позже, но её веса хватило, чтобы она тоже посчитала себя главной. Сейчас обе машины отвечают по IP 10.164.0.14.
Процесс keepalived пытается запустить скрипт killall от пользователя keepalived_script, которого не существует, так как сам процесс работает от root. Это небезопасно и может привести к падению проверки при обновлении системы.

Далее исправляем выявленные ошибки
####
```sh
--создание системного пользователя для скриптов
asvpg@vm-haproxy1:~$ sudo adduser --system --no-create-home --group --home /nonexistent keepalived_script
info: The home dir /nonexistent you specified can't be accessed: No such file or directory

info: Selecting UID from range 100 to 999 ...

info: Selecting GID from range 100 to 999 ...
info: Adding system user `keepalived_script' (UID 111) ...
info: Adding new group `keepalived_script' (GID 114) ...
info: Adding new user `keepalived_script' (UID 111) with group `keepalived_script' ...
info: Not creating `/nonexistent'.
asvpg@vm-haproxy1:~$ id keepalived_script
uid=111(keepalived_script) gid=114(keepalived_script) groups=114(keepalived_script)
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo adduser --system --no-create-home --group --home /nonexistent keepalived_script
info: The home dir /nonexistent you specified can't be accessed: No such file or directory

info: Selecting UID from range 100 to 999 ...

info: Selecting GID from range 100 to 999 ...
info: Adding system user `keepalived_script' (UID 111) ...
info: Adding new group `keepalived_script' (GID 114) ...
info: Adding new user `keepalived_script' (UID 111) with group `keepalived_script' ...
info: Not creating `/nonexistent'.
asvpg@vm-haproxy2:~$ id keepalived_script
uid=111(keepalived_script) gid=114(keepalived_script) groups=114(keepalived_script)
asvpg@vm-haproxy2:~$

--добавляем в конфиг блок с безопасностью для созданного пользователя и добавляем закрывающую скобку
asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    script "killall -0 haproxy" # Проверка наличия процесса
    interval 2                  # Каждые 2 секунды
    weight 2                    # Если ок, прибавим +2 к приоритету
    fall 2                      # Считать упавшим после 2 неудачных проверок
    rise 2                      # Считать поднявшимся после 2 успешных
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 101                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass SuperSecretPasswordHere
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
(/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Disabling track script check_haproxy since not found/accessible
asvpg@vm-haproxy1:~$

--уменьшаем пароль до 8 символов (ограничения старой версии)
asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    script "killall -0 haproxy" # Проверка наличия процесса
    interval 2                  # Каждые 2 секунды
    weight 2                    # Если ок, прибавим +2 к приоритету
    fall 2                      # Считать упавшим после 2 неудачных проверок
    rise 2                      # Считать поднявшимся после 2 успешных
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 101                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass PASS1
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
Disabling track script check_haproxy since not found/accessible
asvpg@vm-haproxy1:~$

--указываем полный путь для killall
asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy" # Проверка наличия процесса
    interval 2                  # Каждые 2 секунды
    weight 2                    # Если ок, прибавим +2 к приоритету
    fall 2                      # Считать упавшим после 2 неудачных проверок
    rise 2                      # Считать поднявшимся после 2 успешных
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 101                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass PASS1
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
SECURITY VIOLATION - scripts are being executed but script_security not enabled.
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy2:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_02 # Уникальное имя
}

vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state BACKUP               # Роль ведомого
    interface eth0             # Тот же интерфейс
    virtual_router_id 51       # Тот же ID группы!
    priority 100               # Приоритет ниже мастера

    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASS1 # Пароль должен совпадать с мастером
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip
    }

    track_script {
        check_haproxy
    }
}
asvpg@vm-haproxy2:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
SECURITY VIOLATION - scripts are being executed but script_security not enabled.
asvpg@vm-haproxy2:~$
```

####
Запускаем и проверяем
####
```sh
asvpg@vm-haproxy1:~$ sudo systemctl enable keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
asvpg@vm-haproxy1:~$ sudo systemctl start keepalived
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: (/etc/keepalived/keepalived.conf: Line 33) Unexpected '{' - ignoring
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: (/etc/keepalived/keepalived.conf: Line 34) Unknown keyword 'script_user'
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: (/etc/keepalived/keepalived.conf: Line 36) Unknown keyword '}'
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: Unable to set supplementary gids (Operation not permitted)
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: Permissions failure for script killall in path - disabling
Aug 16 11:47:02 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: Disabling track script check_haproxy since not found/accessible
Aug 16 11:47:02 haproxynode Keepalived_vrrp[10096]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:47:02 haproxynode Keepalived[10095]: Startup complete
Aug 16 11:47:05 haproxynode Keepalived_vrrp[10096]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo systemctl enable keepalived
Synchronizing state of keepalived.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable keepalived
asvpg@vm-haproxy2:~$ sudo systemctl start keepalived
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 11:47:07 haproxynode Keepalived[9698]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 11:47:07 haproxynode Keepalived[9698]: Starting VRRP child process, pid=9699
Aug 16 11:47:07 haproxynode Keepalived_vrrp[9699]: (/etc/keepalived/keepalived.conf: Line 22) Truncating auth_pass to 8 characters
Aug 16 11:47:07 haproxynode Keepalived_vrrp[9699]: Unable to set supplementary gids (Operation not permitted)
Aug 16 11:47:07 haproxynode Keepalived_vrrp[9699]: Permissions failure for script killall in path - disabling
Aug 16 11:47:07 haproxynode Keepalived_vrrp[9699]: Disabling track script check_haproxy since not found/accessible
Aug 16 11:47:07 haproxynode Keepalived_vrrp[9699]: (VI_01) Entering BACKUP STATE (init)
Aug 16 11:47:07 haproxynode Keepalived[9698]: Startup complete
Aug 16 11:47:07 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 11:47:10 haproxynode Keepalived_vrrp[9699]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$
```

####
Снова Split Brain, увеличим разницы в приоритете с 1 до 5
####
```sh
asvpg@vm-haproxy1:~$ sudo systemctl stop keepalived
asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy" # Проверка наличия процесса
    interval 2                  # Каждые 2 секунды
    weight 2                    # Если ок, прибавим +2 к приоритету
    fall 2                      # Считать упавшим после 2 неудачных проверок
    rise 2                      # Считать поднявшимся после 2 успешных
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 105                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass PASS1
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
SECURITY VIOLATION - scripts are being executed but script_security not enabled.
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy1:~$ sudo systemctl edit keepalived
--добавим следующий параметр в настройки ядра через службу systemd для исправления ошибки permission failure
[Service]
ExecStart=
ExecStart=/usr/sbin/keepalived --dont-fork --log-console --release-vips


asvpg@vm-haproxy1:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy1:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:06:39 haproxynode Keepalived[10495]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:06:39 haproxynode Keepalived[10495]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:06:39 haproxynode Keepalived[10495]: Starting VRRP child process, pid=10496
Aug 16 12:06:39 haproxynode Keepalived_vrrp[10496]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:06:39 haproxynode Keepalived[10495]: Startup complete
Aug 16 12:06:39 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:06:39 haproxynode Keepalived_vrrp[10496]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:06:39 haproxynode Keepalived_vrrp[10496]: Script `check_haproxy` now returning 1
Aug 16 12:06:39 haproxynode Keepalived_vrrp[10496]: VRRP_Script(check_haproxy) failed (exited with status 1)
Aug 16 12:06:42 haproxynode Keepalived_vrrp[10496]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy2:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:06:42 haproxynode Keepalived[10070]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:06:42 haproxynode Keepalived[10070]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:06:42 haproxynode Keepalived[10070]: Starting VRRP child process, pid=10071
Aug 16 12:06:42 haproxynode Keepalived_vrrp[10071]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:06:42 haproxynode Keepalived_vrrp[10071]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:06:42 haproxynode Keepalived[10070]: Startup complete
Aug 16 12:06:42 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:06:42 haproxynode Keepalived_vrrp[10071]: Script `check_haproxy` now returning 1
Aug 16 12:06:42 haproxynode Keepalived_vrrp[10071]: VRRP_Script(check_haproxy) failed (exited with status 1)
Aug 16 12:06:46 haproxynode Keepalived_vrrp[10071]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$

asvpg@vm-haproxy1:~$ ps aux | grep [h]aproxy
root        7752  0.0  0.6  96880 13296 ?        Ss   10:26   0:00 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
root        7756  0.0  2.0 151712 40436 ?        Sl   10:26   0:01 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
asvpg@vm-haproxy1:~$

--Команда killall -0 haproxy не срабатывает, потому что:
Бинарник лежит в папке /usr/sbin/. Процесс keepalived) может просто не видеть эту директорию при поиске по короткому имени.
Запущено два процесса (7752 — родительский мастер-процесс для проверки конфигурации, и 7756 — рабочий дочерний). Команда killall -0 отправляет сигнал "ноль" всем найденным процессам. Если хотя бы один из них принадлежит другому пользователю или имеет другие права доступа, команда может вернуть код ошибки 1.

asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    # Проверяем наличие файла и существование процесса по его ID внутри
    script "[ -f /run/haproxy.pid ] && kill -0 $(cat /run/haproxy.pid)"

    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 105                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass PASS1
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
Disabling track script check_haproxy since not found/accessible
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ ps aux | grep [h]aproxy
root        7747  0.0  0.6  96880 13364 ?        Ss   10:26   0:00 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
root        7751  0.0  2.0 151712 40452 ?        Sl   10:26   0:01 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
asvpg@vm-haproxy2:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy2:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_02 # Уникальное имя
}

vrrp_script check_haproxy {
    # Проверяем наличие файла и существование процесса по его ID внутри
    script "[ -f /run/haproxy.pid ] && kill -0 $(cat /run/haproxy.pid)"

    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state BACKUP               # Роль ведомого
    interface eth0             # Тот же интерфейс
    virtual_router_id 51       # Тот же ID группы!
    priority 100               # Приоритет ниже мастера

    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASS1 # Пароль должен совпадать с мастером
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip
    }

    track_script {
        check_haproxy
    }
}
asvpg@vm-haproxy2:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
Disabling track script check_haproxy since not found/accessible
asvpg@vm-haproxy2:~$

asvpg@vm-haproxy1:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy1:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:18:13 haproxynode Keepalived[10954]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:18:13 haproxynode Keepalived[10954]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:18:13 haproxynode Keepalived[10954]: Starting VRRP child process, pid=10957
Aug 16 12:18:13 haproxynode Keepalived_vrrp[10957]: Unable to set supplementary gids (Operation not permitted)
Aug 16 12:18:13 haproxynode Keepalived_vrrp[10957]: Permissions failure for script [ in path - disabling
Aug 16 12:18:13 haproxynode Keepalived_vrrp[10957]: Disabling track script check_haproxy since not found/accessible
Aug 16 12:18:13 haproxynode Keepalived_vrrp[10957]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:18:13 haproxynode Keepalived[10954]: Startup complete
Aug 16 12:18:13 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:18:17 haproxynode Keepalived_vrrp[10957]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy2:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:18:17 haproxynode Keepalived[10533]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:18:17 haproxynode Keepalived[10533]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:18:17 haproxynode Keepalived[10533]: Starting VRRP child process, pid=10534
Aug 16 12:18:17 haproxynode Keepalived_vrrp[10534]: Unable to set supplementary gids (Operation not permitted)
Aug 16 12:18:17 haproxynode Keepalived_vrrp[10534]: Permissions failure for script [ in path - disabling
Aug 16 12:18:17 haproxynode Keepalived_vrrp[10534]: Disabling track script check_haproxy since not found/accessible
Aug 16 12:18:17 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:18:17 haproxynode Keepalived_vrrp[10534]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:18:17 haproxynode Keepalived[10533]: Startup complete
Aug 16 12:18:21 haproxynode Keepalived_vrrp[10534]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$



asvpg@vm-haproxy1:~$ keepalived --version
Keepalived v2.2.8 (04/04,2023), git commit v2.2.7-154-g292b299e+

Copyright(C) 2001-2023 Alexandre Cassen, <acassen@gmail.com>

Built with kernel headers for Linux 6.8.0
Running on Linux 6.8.0-137-generic #137-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul 17 20:28:23 UTC 2026
Distro: Ubuntu 24.04.4 LTS

configure options: --build=x86_64-linux-gnu --prefix=/usr --includedir=${prefix}/include --mandir=${prefix}/share/man --infodir=${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-option-checking --disable-silent-rules --libdir=${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking --enable-snmp --enable-sha1 --enable-snmp-rfcv2 --enable-snmp-rfcv3 --enable-dbus --enable-json --enable-bfd --enable-regex --with-init=systemd build_alias=x86_64-linux-gnu CFLAGS=-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer  -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/keepalived-F53vvG/keepalived-2.2.8=/usr/src/keepalived-1:2.2.8-1build2 LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=3

Config options:  NFTABLES LVS REGEX VRRP VRRP_AUTH VRRP_VMAC JSON BFD OLD_CHKSUM_COMPAT SNMP_V3_FOR_V2 SNMP_VRRP SNMP_CHECKER SNMP_RFCV2 SNMP_RFCV3 DBUS INIT=systemd SYSTEMD_NOTIFY

System options:  VSYSLOG MEMFD_CREATE IPV6_MULTICAST_ALL IPV4_DEVCONF LIBNL3 RTA_ENCAP RTA_EXPIRES RTA_NEWDST RTA_PREF FRA_SUPPRESS_PREFIXLEN FRA_SUPPRESS_IFGROUP FRA_TUN_ID RTAX_CC_ALGO RTAX_QUICKACK RTEXT_FILTER_SKIP_STATS FRA_L3MDEV FRA_UID_RANGE RTAX_FASTOPEN_NO_COOKIE RTA_VIA FRA_PROTOCOL FRA_IP_PROTO FRA_SPORT_RANGE FRA_DPORT_RANGE RTA_TTL_PROPAGATE IFA_FLAGS LWTUNNEL_ENCAP_MPLS LWTUNNEL_ENCAP_ILA NET_LINUX_IF_H_COLLISION LIBIPVS_NETLINK IPVS_DEST_ATTR_ADDR_FAMILY IPVS_SYNCD_ATTRIBUTES IPVS_64BIT_STATS IPVS_TUN_TYPE IPVS_TUN_CSUM IPVS_TUN_GRE VRRP_IPVLAN IFLA_LINK_NETNSID GLOB_BRACE GLOB_ALTDIRFUNC INET6_ADDR_GEN_MODE VRF SO_MARK
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ keepalived --version
Keepalived v2.2.8 (04/04,2023), git commit v2.2.7-154-g292b299e+

Copyright(C) 2001-2023 Alexandre Cassen, <acassen@gmail.com>

Built with kernel headers for Linux 6.8.0
Running on Linux 6.8.0-137-generic #137-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul 17 20:28:23 UTC 2026
Distro: Ubuntu 24.04.4 LTS

configure options: --build=x86_64-linux-gnu --prefix=/usr --includedir=${prefix}/include --mandir=${prefix}/share/man --infodir=${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-option-checking --disable-silent-rules --libdir=${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking --enable-snmp --enable-sha1 --enable-snmp-rfcv2 --enable-snmp-rfcv3 --enable-dbus --enable-json --enable-bfd --enable-regex --with-init=systemd build_alias=x86_64-linux-gnu CFLAGS=-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer  -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/keepalived-F53vvG/keepalived-2.2.8=/usr/src/keepalived-1:2.2.8-1build2 LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=3

Config options:  NFTABLES LVS REGEX VRRP VRRP_AUTH VRRP_VMAC JSON BFD OLD_CHKSUM_COMPAT SNMP_V3_FOR_V2 SNMP_VRRP SNMP_CHECKER SNMP_RFCV2 SNMP_RFCV3 DBUS INIT=systemd SYSTEMD_NOTIFY

System options:  VSYSLOG MEMFD_CREATE IPV6_MULTICAST_ALL IPV4_DEVCONF LIBNL3 RTA_ENCAP RTA_EXPIRES RTA_NEWDST RTA_PREF FRA_SUPPRESS_PREFIXLEN FRA_SUPPRESS_IFGROUP FRA_TUN_ID RTAX_CC_ALGO RTAX_QUICKACK RTEXT_FILTER_SKIP_STATS FRA_L3MDEV FRA_UID_RANGE RTAX_FASTOPEN_NO_COOKIE RTA_VIA FRA_PROTOCOL FRA_IP_PROTO FRA_SPORT_RANGE FRA_DPORT_RANGE RTA_TTL_PROPAGATE IFA_FLAGS LWTUNNEL_ENCAP_MPLS LWTUNNEL_ENCAP_ILA NET_LINUX_IF_H_COLLISION LIBIPVS_NETLINK IPVS_DEST_ATTR_ADDR_FAMILY IPVS_SYNCD_ATTRIBUTES IPVS_64BIT_STATS IPVS_TUN_TYPE IPVS_TUN_CSUM IPVS_TUN_GRE VRRP_IPVLAN IFLA_LINK_NETNSID GLOB_BRACE GLOB_ALTDIRFUNC INET6_ADDR_GEN_MODE VRF SO_MARK
asvpg@vm-haproxy2:~$

--проверяем существование файлов PID, убеждаемся, что HAProxy пишет свой ID туда, куда мы смотрим - путь верный
asvpg@vm-haproxy1:~$ cat /run/haproxy.pid
7752
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ cat /run/haproxy.pid
7747
asvpg@vm-haproxy2:~$

--выносим секцию vrrp_script в отдельный скрипт
asvpg@vm-haproxy1:~$ sudo mkdir -p /usr/local/bin/keepalived_scripts
asvpg@vm-haproxy1:~$ echo '#!/bin/bash
if [ -f /run/haproxy.pid ]; then
    kill -0 $(cat /run/haproxy.pid)
    exit $?
else
    exit 1
fi' | sudo tee /usr/local/bin/check_haproxy.sh
#!/bin/bash
if [ -f /run/haproxy.pid ]; then
    kill -0 $(cat /run/haproxy.pid)
    exit $?
else
    exit 1
fi
asvpg@vm-haproxy1:~$ sudo chmod +x /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy1:~$ ls -l /usr/local/bin/check_haproxy.sh
-rwxr-xr-x 1 root root 112 Aug 16 12:25 /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy1:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy1:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_01 # Уникальное имя узла в логах
}

vrrp_script check_haproxy {
    #указываем путь к скрипту
    script "/usr/local/bin/check_haproxy.sh"
    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state MASTER                # Роль этой машины
    interface eth0              # Смотрим интерфейс через ip a
    virtual_router_id 51        # ID группы (должен совпадать у обеих нод)
    priority 105                # Приоритет (чем выше, тем главнее)

    advert_int 1                # Интервал объявлений о себе (секунды)
    authentication {            # Простая защита протокола VRRP
        auth_type PASS
        auth_pass PASS1
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip # Плавающий IP
    }

    track_script {
        check_haproxy           # Связываем проверку с экземпляром
    }
}
asvpg@vm-haproxy1:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
SECURITY VIOLATION - scripts are being executed but script_security not enabled.
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo chmod +x /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy2:~$ ls -l /usr/local/bin/check_haproxy.sh
-rwxr-xr-x 1 root root 112 Aug 16 12:25 /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy2:~$ sudo nano /etc/keepalived/keepalived.conf
asvpg@vm-haproxy2:~$ sudo cat /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_haproxy_02 # Уникальное имя
}

vrrp_script check_haproxy {
    script "/usr/local/bin/check_haproxy.sh"
    interval 2
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_01 {
    state BACKUP               # Роль ведомого
    interface eth0             # Тот же интерфейс
    virtual_router_id 51       # Тот же ID группы!
    priority 100               # Приоритет ниже мастера

    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASS1 # Пароль должен совпадать с мастером
    }

    virtual_ipaddress {
        10.128.0.14 dev eth0 label eth0:vip
    }

    track_script {
        check_haproxy
    }
}
asvpg@vm-haproxy2:~$ sudo keepalived --config-test -f /etc/keepalived/keepalived.conf
SECURITY VIOLATION - scripts are being executed but script_security not enabled.
asvpg@vm-haproxy2:~$

asvpg@vm-haproxy1:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy1:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:29:53 haproxynode Keepalived[11066]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:29:53 haproxynode Keepalived[11066]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:29:53 haproxynode Keepalived[11066]: Starting VRRP child process, pid=11068
Aug 16 12:29:53 haproxynode Keepalived_vrrp[11068]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:29:53 haproxynode Keepalived_vrrp[11068]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:29:53 haproxynode Keepalived[11066]: Startup complete
Aug 16 12:29:53 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:29:53 haproxynode Keepalived_vrrp[11068]: Script `check_haproxy` now returning 1
Aug 16 12:29:53 haproxynode Keepalived_vrrp[11068]: VRRP_Script(check_haproxy) failed (exited with status 1)
Aug 16 12:29:57 haproxynode Keepalived_vrrp[11068]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy2:~$ sudo systemctl restart keepalived
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:29:57 haproxynode Keepalived[10629]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:29:57 haproxynode Keepalived[10629]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:29:57 haproxynode Keepalived[10629]: Starting VRRP child process, pid=10632
Aug 16 12:29:57 haproxynode Keepalived_vrrp[10632]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:29:57 haproxynode Keepalived_vrrp[10632]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:29:57 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:29:57 haproxynode Keepalived[10629]: Startup complete
Aug 16 12:29:57 haproxynode Keepalived_vrrp[10632]: Script `check_haproxy` now returning 1
Aug 16 12:29:57 haproxynode Keepalived_vrrp[10632]: VRRP_Script(check_haproxy) failed (exited with status 1)
Aug 16 12:30:01 haproxynode Keepalived_vrrp[10632]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$


--меняем скрипт /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy1:~$ sudo nano /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy1:~$ sudo /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy1:~$ echo $?
0
asvpg@vm-haproxy1:~$ sudo systemctl stop keepalived
asvpg@vm-haproxy1:~$ sudo cat /usr/local/bin/check_haproxy.sh
#!/bin/bash
if [ -f /run/haproxy.pid ] && ps -p "$(cat /run/haproxy.pid)" > /dev/null 2>&1; then
    exit 0
else
    exit 1
fi
asvpg@vm-haproxy1:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy1:~$ sudo systemctl start keepalived
asvpg@vm-haproxy1:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:44:27 haproxynode Keepalived[11877]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:44:27 haproxynode Keepalived[11877]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:44:27 haproxynode Keepalived[11877]: Starting VRRP child process, pid=11878
Aug 16 12:44:27 haproxynode Keepalived_vrrp[11878]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:44:27 haproxynode Keepalived[11877]: Startup complete
Aug 16 12:44:27 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:44:27 haproxynode Keepalived_vrrp[11878]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:44:27 haproxynode Keepalived_vrrp[11878]: VRRP_Script(check_haproxy) succeeded
Aug 16 12:44:27 haproxynode Keepalived_vrrp[11878]: (VI_01) Changing effective priority from 105 to 107
Aug 16 12:44:31 haproxynode Keepalived_vrrp[11878]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ sudo nano /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy2:~$ sudo cat /usr/local/bin/check_haproxy.sh
#!/bin/bash
if systemctl is-active --quiet haproxy; then
    exit 0
else
    exit 1
fi
asvpg@vm-haproxy2:~$ sudo /usr/local/bin/check_haproxy.sh
asvpg@vm-haproxy2:~$ echo $?
0
asvpg@vm-haproxy2:~$ sudo systemctl stop keepalived
asvpg@vm-haproxy2:~$ sudo systemctl daemon-reload
asvpg@vm-haproxy2:~$ sudo systemctl start keepalived
asvpg@vm-haproxy2:~$ sudo journalctl -f -u keepalived --no-pager
Aug 16 12:44:54 haproxynode Keepalived[11470]: Configuration file /etc/keepalived/keepalived.conf
Aug 16 12:44:54 haproxynode Keepalived[11470]: NOTICE: setting config option max_auto_priority should result in better keepalived performance
Aug 16 12:44:54 haproxynode Keepalived[11470]: Starting VRRP child process, pid=11471
Aug 16 12:44:54 haproxynode Keepalived_vrrp[11471]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Aug 16 12:44:54 haproxynode Keepalived_vrrp[11471]: (VI_01) Entering BACKUP STATE (init)
Aug 16 12:44:54 haproxynode Keepalived[11470]: Startup complete
Aug 16 12:44:54 haproxynode systemd[1]: Started keepalived.service - Keepalive Daemon (LVS and VRRP).
Aug 16 12:44:54 haproxynode Keepalived_vrrp[11471]: VRRP_Script(check_haproxy) succeeded
Aug 16 12:44:54 haproxynode Keepalived_vrrp[11471]: (VI_01) Changing effective priority from 100 to 102
Aug 16 12:44:58 haproxynode Keepalived_vrrp[11471]: (VI_01) Entering MASTER STATE
asvpg@vm-haproxy2:~$
```

####
Видимо, инфраструктура блокирует мультикаст-трафик VRRP (протокол 112), т.к. пинг проходит
####
```sh
asvpg@vm-haproxy1:~$ ping -c 3 10.128.0.4
PING 10.128.0.4 (10.128.0.4) 56(84) bytes of data.
64 bytes from 10.128.0.4: icmp_seq=1 ttl=61 time=1.27 ms
64 bytes from 10.128.0.4: icmp_seq=2 ttl=61 time=0.361 ms
64 bytes from 10.128.0.4: icmp_seq=3 ttl=61 time=0.339 ms

--- 10.128.0.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2055ms
rtt min/avg/max/mdev = 0.339/0.657/1.273/0.435 ms
asvpg@vm-haproxy1:~$

asvpg@vm-haproxy2:~$ ping -c 3 10.128.0.31
PING 10.128.0.31 (10.128.0.31) 56(84) bytes of data.
64 bytes from 10.128.0.31: icmp_seq=1 ttl=61 time=0.312 ms
64 bytes from 10.128.0.31: icmp_seq=2 ttl=61 time=0.546 ms
64 bytes from 10.128.0.31: icmp_seq=3 ttl=61 time=0.441 ms

--- 10.128.0.31 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2049ms
rtt min/avg/max/mdev = 0.312/0.433/0.546/0.095 ms
asvpg@vm-haproxy2:~$

--2 ВМ не видят пакеты VRRP - и это как я считаю, главная причина split brain.
В интерфейсе Яндекс Облака добавлял правила на открытие входящего и исходящего трафика, но не помогло. Может быть ошибся в настройке.
```

###
Тестирование
###

####
Выполнение switchover
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   | 12 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 12 |  0/2600FEA0 |   0 | 0/2600FEA0 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 12 |  0/2600FEA0 |   0 | 0/2600FEA0 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml switchover
Current cluster topology
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   | 12 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 12 |  0/2600FEA0 |   0 | 0/2600FEA0 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 12 |  0/2600FEA0 |   0 | 0/2600FEA0 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
Primary [vm-pg1]:
Candidate ['vm-pg2', 'vm-pg3'] []: vm-pg2
When should the switchover take place (e.g. 2026-08-16T19:05 )  [now]:
Are you sure you want to switchover cluster patroni, demoting current leader vm-pg1? [y/N]: y
2026-08-16 18:05:20.61880 Successfully switched over to "vm-pg2"
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | stopped |    |     unknown |     |    unknown |     |
| vm-pg2 | 10.130.0.28 | Leader  | running | 12 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | running | 12 |  0/2600FFE8 |   0 | 0/2600FFE8 |   0 |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 13 |  0/26010140 |   0 | 0/26010140 |   0 |
| vm-pg2 | 10.130.0.28 | Leader  | running   | 13 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 13 |  0/26010140 |   0 | 0/26010140 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

asvpg@vm-pg1:~$ sudo journalctl -u patroni.service -n 40 -f
Aug 16 18:05:22 vm-pg1 patroni[775]: 10        0/26000990        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 11        0/2600D330        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 12        0/2600FFE8        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,016 INFO: closed patroni connections to postgres
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,367 INFO: postmaster pid=5720
Aug 16 18:05:22 vm-pg1 patroni[5720]: 2026-08-16 18:05:22.373 UTC [5720] LOG:  starting PostgreSQL 17.11 (Ubuntu 17.11-1.pgdg24.04+2) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0, 64-bit
Aug 16 18:05:22 vm-pg1 patroni[5720]: 2026-08-16 18:05:22.373 UTC [5720] LOG:  listening on IPv4 address "127.0.0.1", port 5432
Aug 16 18:05:22 vm-pg1 patroni[5720]: 2026-08-16 18:05:22.377 UTC [5720] LOG:  listening on IPv4 address "10.130.0.13", port 5432
Aug 16 18:05:22 vm-pg1 patroni[5720]: 2026-08-16 18:05:22.383 UTC [5720] LOG:  listening on Unix socket "./.s.PGSQL.5432"
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.395 UTC [5725] LOG:  database system was shut down at 2026-08-16 18:05:18 UTC
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.395 UTC [5725] LOG:  entering standby mode
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.403 UTC [5725] LOG:  consistent recovery state reached at 0/2600FFE8
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.403 UTC [5725] LOG:  invalid record length at 0/2600FFE8: expected at least 24, got 0
Aug 16 18:05:22 vm-pg1 patroni[5720]: 2026-08-16 18:05:22.403 UTC [5720] LOG:  database system is ready to accept read-only connections
Aug 16 18:05:22 vm-pg1 patroni[5726]: 2026-08-16 18:05:22.404 UTC [5726] postgres@postgres FATAL:  the database system is starting up
Aug 16 18:05:22 vm-pg1 patroni[5721]: localhost:5432 - rejecting connections
Aug 16 18:05:22 vm-pg1 patroni[5727]: localhost:5432 - accepting connections
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,425 INFO: Lock owner: vm-pg2; I am vm-pg1
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,425 INFO: establishing a new patroni heartbeat connection to postgres
Aug 16 18:05:22 vm-pg1 patroni[5728]: 2026-08-16 18:05:22.427 UTC [5728] LOG:  fetching timeline history file for timeline 13 from primary server
Aug 16 18:05:22 vm-pg1 patroni[5728]: 2026-08-16 18:05:22.435 UTC [5728] LOG:  started streaming WAL from primary at 0/26000000 on timeline 12
Aug 16 18:05:22 vm-pg1 patroni[5728]: 2026-08-16 18:05:22.436 UTC [5728] LOG:  replication terminated by primary server
Aug 16 18:05:22 vm-pg1 patroni[5728]: 2026-08-16 18:05:22.436 UTC [5728] DETAIL:  End of WAL reached on timeline 12 at 0/2600FFE8.
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.437 UTC [5725] LOG:  new target timeline is 13
Aug 16 18:05:22 vm-pg1 patroni[5728]: 2026-08-16 18:05:22.437 UTC [5728] LOG:  restarted WAL streaming at 0/26000000 on timeline 13
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,479 INFO: Local timeline=12 lsn=0/2600FFE8
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,519 INFO: primary_timeline=13
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,520 INFO: primary: history=9        0/260007D8        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 10        0/26000990        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 11        0/2600D330        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[775]: 12        0/2600FFE8        no recovery target specified
Aug 16 18:05:22 vm-pg1 patroni[5725]: 2026-08-16 18:05:22.559 UTC [5725] LOG:  redo starts at 0/2600FFE8
Aug 16 18:05:22 vm-pg1 patroni[775]: 2026-08-16 18:05:22,633 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:25 vm-pg1 patroni[775]: 2026-08-16 18:05:25,184 INFO: establishing a new patroni restapi connection to postgres
Aug 16 18:05:31 vm-pg1 patroni[775]: 2026-08-16 18:05:31,399 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:41 vm-pg1 patroni[775]: 2026-08-16 18:05:41,842 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:51 vm-pg1 patroni[775]: 2026-08-16 18:05:51,843 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:01 vm-pg1 patroni[775]: 2026-08-16 18:06:01,841 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:11 vm-pg1 patroni[775]: 2026-08-16 18:06:11,841 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:21 vm-pg1 patroni[775]: 2026-08-16 18:06:21,841 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:31 vm-pg1 patroni[775]: 2026-08-16 18:06:31,841 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
asvpg@vm-pg1:~$

Aug 16 18:05:05 vm-pg2 patroni[775]: 2026-08-16 18:05:05,027 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 16 18:05:15 vm-pg2 patroni[775]: 2026-08-16 18:05:15,027 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 16 18:05:19 vm-pg2 patroni[775]: 2026-08-16 18:05:19,944 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg1)
Aug 16 18:05:20 vm-pg2 patroni[775]: 2026-08-16 18:05:20,055 INFO: Cleaning up failover key after acquiring leader lock...
Aug 16 18:05:20 vm-pg2 patroni[775]: 2026-08-16 18:05:20,169 INFO: promoted self to leader by acquiring session lock
Aug 16 18:05:20 vm-pg2 patroni[3521]: server promoting
Aug 16 18:05:21 vm-pg2 patroni[775]: 2026-08-16 18:05:21,359 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:05:31 vm-pg2 patroni[775]: 2026-08-16 18:05:31,342 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:05:41 vm-pg2 patroni[775]: 2026-08-16 18:05:41,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:05:51 vm-pg2 patroni[775]: 2026-08-16 18:05:51,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:01 vm-pg2 patroni[775]: 2026-08-16 18:06:01,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:11 vm-pg2 patroni[775]: 2026-08-16 18:06:11,284 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:21 vm-pg2 patroni[775]: 2026-08-16 18:06:21,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:31 vm-pg2 patroni[775]: 2026-08-16 18:06:31,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:41 vm-pg2 patroni[775]: 2026-08-16 18:06:41,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:06:51 vm-pg2 patroni[775]: 2026-08-16 18:06:51,228 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:07:01 vm-pg2 patroni[775]: 2026-08-16 18:07:01,229 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:07:11 vm-pg2 patroni[775]: 2026-08-16 18:07:11,228 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:07:21 vm-pg2 patroni[775]: 2026-08-16 18:07:21,228 INFO: no action. I am (vm-pg2), the leader with the lock
asvpg@vm-pg2:~$

asvpg@vm-pg3:~$ sudo journalctl -u patroni.service -n 40 -f
Aug 16 18:05:22 vm-pg3 patroni[3075]: 2026-08-16 18:05:22.015 UTC [3075] LOG:  parameter "primary_conninfo" changed to "dbname=postgres user=repl_user passfile=/tmp/pgpass0 host=10.130.0.28 port=5432 sslmode=prefer application_name=vm-pg3 gssencmode=prefer channel_binding=prefer sslnegotiation=postgres"
Aug 16 18:05:22 vm-pg3 patroni[5262]: 2026-08-16 18:05:22.035 UTC [5262] LOG:  fetching timeline history file for timeline 13 from primary server
Aug 16 18:05:22 vm-pg3 patroni[5262]: 2026-08-16 18:05:22.042 UTC [5262] LOG:  started streaming WAL from primary at 0/26000000 on timeline 12
Aug 16 18:05:22 vm-pg3 patroni[5262]: 2026-08-16 18:05:22.043 UTC [5262] LOG:  replication terminated by primary server
Aug 16 18:05:22 vm-pg3 patroni[5262]: 2026-08-16 18:05:22.043 UTC [5262] DETAIL:  End of WAL reached on timeline 12 at 0/2600FFE8.
Aug 16 18:05:22 vm-pg3 patroni[3079]: 2026-08-16 18:05:22.043 UTC [3079] LOG:  new target timeline is 13
Aug 16 18:05:22 vm-pg3 patroni[5262]: 2026-08-16 18:05:22.044 UTC [5262] LOG:  restarted WAL streaming at 0/26000000 on timeline 13
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,088 INFO: following a different leader because i am not the healthiest node
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,089 INFO: Lock owner: vm-pg2; I am vm-pg3
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,101 INFO: Local timeline=12 lsn=0/2600FFE8
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,129 INFO: primary_timeline=13
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,129 INFO: primary: history=9        0/260007D8        no recovery target specified
Aug 16 18:05:22 vm-pg3 patroni[777]: 10        0/26000990        no recovery target specified
Aug 16 18:05:22 vm-pg3 patroni[777]: 11        0/2600D330        no recovery target specified
Aug 16 18:05:22 vm-pg3 patroni[777]: 12        0/2600FFE8        no recovery target specified
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,154 INFO: Local timeline=13 lsn=0/2600FFE8
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,185 INFO: primary_timeline=13
Aug 16 18:05:22 vm-pg3 patroni[777]: 2026-08-16 18:05:22,297 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:25 vm-pg3 patroni[3077]: 2026-08-16 18:05:25.020 UTC [3077] LOG:  restartpoint starting: time
Aug 16 18:05:25 vm-pg3 patroni[3077]: 2026-08-16 18:05:25.041 UTC [3077] LOG:  restartpoint complete: wrote 0 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.001 s, sync=0.001 s, total=0.021 s; sync files=0, longest=0.000 s, average=0.000 s; distance=0 kB, estimate=9 kB; lsn=0/26010090, redo lsn=0/26010038
Aug 16 18:05:25 vm-pg3 patroni[3077]: 2026-08-16 18:05:25.041 UTC [3077] LOG:  recovery restart point at 0/26010038
Aug 16 18:05:31 vm-pg3 patroni[777]: 2026-08-16 18:05:31,404 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:41 vm-pg3 patroni[777]: 2026-08-16 18:05:41,840 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:05:51 vm-pg3 patroni[777]: 2026-08-16 18:05:51,840 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:01 vm-pg3 patroni[777]: 2026-08-16 18:06:01,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:11 vm-pg3 patroni[777]: 2026-08-16 18:06:11,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:21 vm-pg3 patroni[777]: 2026-08-16 18:06:21,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:31 vm-pg3 patroni[777]: 2026-08-16 18:06:31,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:41 vm-pg3 patroni[777]: 2026-08-16 18:06:41,843 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:06:51 vm-pg3 patroni[777]: 2026-08-16 18:06:51,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:01 vm-pg3 patroni[777]: 2026-08-16 18:07:01,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:11 vm-pg3 patroni[777]: 2026-08-16 18:07:11,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:21 vm-pg3 patroni[777]: 2026-08-16 18:07:21,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:31 vm-pg3 patroni[777]: 2026-08-16 18:07:31,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:41 vm-pg3 patroni[777]: 2026-08-16 18:07:41,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:07:51 vm-pg3 patroni[777]: 2026-08-16 18:07:51,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:01 vm-pg3 patroni[777]: 2026-08-16 18:08:01,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:11 vm-pg3 patroni[777]: 2026-08-16 18:08:11,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:21 vm-pg3 patroni[777]: 2026-08-16 18:08:21,895 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:31 vm-pg3 patroni[777]: 2026-08-16 18:08:31,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:41 vm-pg3 patroni[777]: 2026-08-16 18:08:41,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
asvpg@vm-pg3:~$
```

####
Выполнение failover
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 13 |  0/26010140 |   0 | 0/26010140 |   0 |
| vm-pg2 | 10.130.0.28 | Leader  | running   | 13 |             |     |            |     |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 13 |  0/26010140 |   0 | 0/26010140 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+

asvpg@vm-pg2:~$ sudo systemctl stop patroni
asvpg@vm-pg2:~$

asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 14 |  0/260102F8 |   0 | 0/260102F8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | stopped   |    |     unknown |     |    unknown |     |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 14 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

asvpg@vm-pg3:~$ sudo journalctl -u patroni.service -n 40 -f
Aug 16 18:08:21 vm-pg3 patroni[777]: 2026-08-16 18:08:21,895 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:31 vm-pg3 patroni[777]: 2026-08-16 18:08:31,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:41 vm-pg3 patroni[777]: 2026-08-16 18:08:41,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:08:51 vm-pg3 patroni[777]: 2026-08-16 18:08:51,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:01 vm-pg3 patroni[777]: 2026-08-16 18:09:01,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:11 vm-pg3 patroni[777]: 2026-08-16 18:09:11,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:21 vm-pg3 patroni[777]: 2026-08-16 18:09:21,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:31 vm-pg3 patroni[777]: 2026-08-16 18:09:31,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:41 vm-pg3 patroni[777]: 2026-08-16 18:09:41,842 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:09:51 vm-pg3 patroni[777]: 2026-08-16 18:09:51,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:01 vm-pg3 patroni[777]: 2026-08-16 18:10:01,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:11 vm-pg3 patroni[777]: 2026-08-16 18:10:11,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:21 vm-pg3 patroni[777]: 2026-08-16 18:10:21,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:31 vm-pg3 patroni[777]: 2026-08-16 18:10:31,841 INFO: no action. I am (vm-pg3), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:36 vm-pg3 patroni[5262]: 2026-08-16 18:10:36.395 UTC [5262] LOG:  replication terminated by primary server
Aug 16 18:10:36 vm-pg3 patroni[5262]: 2026-08-16 18:10:36.395 UTC [5262] DETAIL:  End of WAL reached on timeline 13 at 0/260101B8.
Aug 16 18:10:36 vm-pg3 patroni[5262]: 2026-08-16 18:10:36.395 UTC [5262] FATAL:  could not send end-of-streaming message to primary: SSL connection has been closed unexpectedly
Aug 16 18:10:36 vm-pg3 patroni[5262]:         no COPY in progress
Aug 16 18:10:36 vm-pg3 patroni[3079]: 2026-08-16 18:10:36.396 UTC [3079] LOG:  invalid record length at 0/260101B8: expected at least 24, got 0
Aug 16 18:10:36 vm-pg3 patroni[5317]: 2026-08-16 18:10:36.399 UTC [5317] FATAL:  could not connect to the primary server: connection to server at "10.130.0.28", port 5432 failed: Connection refused
Aug 16 18:10:36 vm-pg3 patroni[5317]:                 Is the server running on that host and accepting TCP/IP connections?
Aug 16 18:10:36 vm-pg3 patroni[3079]: 2026-08-16 18:10:36.399 UTC [3079] LOG:  waiting for WAL to become available at 0/260101D0
Aug 16 18:10:37 vm-pg3 patroni[777]: 2026-08-16 18:10:37,424 INFO: Got response from vm-pg1 http://10.130.0.13:8008/patroni: {"state": "running", "postmaster_start_time": "2026-08-16 18:05:22.387526+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637600184, "replayed_location": 637600184, "replayed_timestamp": null, "paused": false}, "timeline": 13, "cluster_unlocked": true, "dcs_last_seen": 1786903837, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg1"}}
Aug 16 18:10:37 vm-pg3 patroni[777]: 2026-08-16 18:10:37,527 WARNING: Request failed to vm-pg2: GET http://10.130.0.28:8008/patroni (HTTPConnectionPool(host='10.130.0.28', port=8008): Max retries exceeded with url: /patroni (Caused by ProtocolError('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer'))))
Aug 16 18:10:37 vm-pg3 patroni[777]: 2026-08-16 18:10:37,695 INFO: promoted self to leader by acquiring session lock
Aug 16 18:10:37 vm-pg3 patroni[5320]: server promoting
Aug 16 18:10:37 vm-pg3 patroni[3079]: 2026-08-16 18:10:37.697 UTC [3079] LOG:  received promote request
Aug 16 18:10:37 vm-pg3 patroni[3079]: 2026-08-16 18:10:37.697 UTC [3079] LOG:  redo done at 0/26010140 system usage: CPU: user: 0.00 s, system: 0.13 s, elapsed: 27031.96 s
Aug 16 18:10:37 vm-pg3 patroni[3079]: 2026-08-16 18:10:37.714 UTC [3079] LOG:  selected new timeline ID: 14
Aug 16 18:10:37 vm-pg3 patroni[777]: 2026-08-16 18:10:37,696 INFO: Lock owner: vm-pg3; I am vm-pg3
Aug 16 18:10:37 vm-pg3 patroni[777]: 2026-08-16 18:10:37,809 INFO: updated leader lock during promote
Aug 16 18:10:37 vm-pg3 patroni[3079]: 2026-08-16 18:10:37.838 UTC [3079] LOG:  archive recovery complete
Aug 16 18:10:37 vm-pg3 patroni[3077]: 2026-08-16 18:10:37.858 UTC [3077] LOG:  checkpoint starting: force
Aug 16 18:10:37 vm-pg3 patroni[3075]: 2026-08-16 18:10:37.861 UTC [3075] LOG:  database system is ready to accept connections
Aug 16 18:10:37 vm-pg3 patroni[3077]: 2026-08-16 18:10:37.925 UTC [3077] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.008 s, sync=0.002 s, total=0.068 s; sync files=2, longest=0.002 s, average=0.001 s; distance=0 kB, estimate=8 kB; lsn=0/26010248, redo lsn=0/260101F0
Aug 16 18:10:38 vm-pg3 patroni[777]: 2026-08-16 18:10:38,893 INFO: no action. I am (vm-pg3), the leader with the lock
Aug 16 18:10:48 vm-pg3 patroni[777]: 2026-08-16 18:10:48,757 INFO: no action. I am (vm-pg3), the leader with the lock
Aug 16 18:10:58 vm-pg3 patroni[777]: 2026-08-16 18:10:58,757 INFO: no action. I am (vm-pg3), the leader with the lock
Aug 16 18:11:08 vm-pg3 patroni[777]: 2026-08-16 18:11:08,757 INFO: no action. I am (vm-pg3), the leader with the lock
Aug 16 18:11:18 vm-pg3 patroni[777]: 2026-08-16 18:11:18,757 INFO: no action. I am (vm-pg3), the leader with the lock
asvpg@vm-pg3:~$

asvpg@vm-pg1:~$ sudo journalctl -u patroni.service -n 40 -f
Aug 16 18:10:22 vm-pg1 patroni[5723]: 2026-08-16 18:10:22.485 UTC [5723] LOG:  recovery restart point at 0/26010038
Aug 16 18:10:31 vm-pg1 patroni[775]: 2026-08-16 18:10:31,841 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg2)
Aug 16 18:10:36 vm-pg1 patroni[5728]: 2026-08-16 18:10:36.395 UTC [5728] LOG:  replication terminated by primary server
Aug 16 18:10:36 vm-pg1 patroni[5728]: 2026-08-16 18:10:36.395 UTC [5728] DETAIL:  End of WAL reached on timeline 13 at 0/260101B8.
Aug 16 18:10:36 vm-pg1 patroni[5728]: 2026-08-16 18:10:36.395 UTC [5728] FATAL:  could not send end-of-streaming message to primary: SSL connection has been closed unexpectedly
Aug 16 18:10:36 vm-pg1 patroni[5728]:         no COPY in progress
Aug 16 18:10:36 vm-pg1 patroni[5725]: 2026-08-16 18:10:36.396 UTC [5725] LOG:  invalid record length at 0/260101B8: expected at least 24, got 0
Aug 16 18:10:36 vm-pg1 patroni[5767]: 2026-08-16 18:10:36.400 UTC [5767] FATAL:  could not connect to the primary server: connection to server at "10.130.0.28", port 5432 failed: Connection refused
Aug 16 18:10:36 vm-pg1 patroni[5767]:                 Is the server running on that host and accepting TCP/IP connections?
Aug 16 18:10:36 vm-pg1 patroni[5725]: 2026-08-16 18:10:36.400 UTC [5725] LOG:  waiting for WAL to become available at 0/260101D0
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,423 INFO: Got response from vm-pg3 http://10.130.0.33:8008/patroni: {"state": "running", "postmaster_start_time": "2026-08-16 10:40:05.585274+00:00", "role": "replica", "server_version": 170011, "xlog": {"received_location": 637600184, "replayed_location": 637600184, "replayed_timestamp": null, "paused": false}, "timeline": 13, "cluster_unlocked": true, "dcs_last_seen": 1786903837, "database_system_identifier": "7673599298398442043", "patroni": {"version": "4.1.5", "scope": "patroni", "name": "vm-pg3"}}
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,527 WARNING: Request failed to vm-pg2: GET http://10.130.0.28:8008/patroni (HTTPConnectionPool(host='10.130.0.28', port=8008): Max retries exceeded with url: /patroni (Caused by ProtocolError('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer'))))
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,638 INFO: Could not take out TTL lock
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,639 ERROR: watchprefix failed: ProtocolError("Connection broken: InvalidChunkLength(got length b'', 0 bytes read)", InvalidChunkLength(got length b'', 0 bytes read))
Aug 16 18:10:37 vm-pg1 patroni[5769]: server signaled
Aug 16 18:10:37 vm-pg1 patroni[5720]: 2026-08-16 18:10:37.642 UTC [5720] LOG:  received SIGHUP, reloading configuration files
Aug 16 18:10:37 vm-pg1 patroni[5720]: 2026-08-16 18:10:37.643 UTC [5720] LOG:  parameter "primary_conninfo" changed to "dbname=postgres user=repl_user passfile=/tmp/pgpass0 host=10.130.0.33 port=5432 sslmode=prefer application_name=vm-pg1 gssencmode=prefer channel_binding=prefer sslnegotiation=postgres"
Aug 16 18:10:37 vm-pg1 patroni[5771]: 2026-08-16 18:10:37.662 UTC [5771] LOG:  started streaming WAL from primary at 0/26000000 on timeline 13
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,709 INFO: following new leader after trying and failing to obtain lock
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,710 INFO: Lock owner: vm-pg3; I am vm-pg1
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,725 INFO: Local timeline=13 lsn=0/260101B8
Aug 16 18:10:37 vm-pg1 patroni[775]: 2026-08-16 18:10:37,844 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:10:37 vm-pg1 patroni[5771]: 2026-08-16 18:10:37.857 UTC [5771] LOG:  replication terminated by primary server
Aug 16 18:10:37 vm-pg1 patroni[5771]: 2026-08-16 18:10:37.857 UTC [5771] DETAIL:  End of WAL reached on timeline 13 at 0/260101B8.
Aug 16 18:10:37 vm-pg1 patroni[5771]: 2026-08-16 18:10:37.858 UTC [5771] LOG:  fetching timeline history file for timeline 14 from primary server
Aug 16 18:10:37 vm-pg1 patroni[5725]: 2026-08-16 18:10:37.866 UTC [5725] LOG:  new target timeline is 14
Aug 16 18:10:37 vm-pg1 patroni[5771]: 2026-08-16 18:10:37.867 UTC [5771] LOG:  restarted WAL streaming at 0/26000000 on timeline 14
Aug 16 18:10:38 vm-pg1 patroni[775]: 2026-08-16 18:10:38,813 INFO: Lock owner: vm-pg3; I am vm-pg1
Aug 16 18:10:38 vm-pg1 patroni[775]: 2026-08-16 18:10:38,834 INFO: Local timeline=14 lsn=0/260102F8
Aug 16 18:10:38 vm-pg1 patroni[775]: 2026-08-16 18:10:38,946 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:10:49 vm-pg1 patroni[775]: 2026-08-16 18:10:49,313 INFO: Lock owner: vm-pg3; I am vm-pg1
Aug 16 18:10:49 vm-pg1 patroni[775]: 2026-08-16 18:10:49,326 INFO: Local timeline=14 lsn=0/260102F8
Aug 16 18:10:49 vm-pg1 patroni[775]: 2026-08-16 18:10:49,359 INFO: primary_timeline=14
Aug 16 18:10:49 vm-pg1 patroni[775]: 2026-08-16 18:10:49,415 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:10:59 vm-pg1 patroni[775]: 2026-08-16 18:10:59,369 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:11:09 vm-pg1 patroni[775]: 2026-08-16 18:11:09,369 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:11:19 vm-pg1 patroni[775]: 2026-08-16 18:11:19,370 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:11:29 vm-pg1 patroni[775]: 2026-08-16 18:11:29,370 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:11:39 vm-pg1 patroni[775]: 2026-08-16 18:11:39,369 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
Aug 16 18:11:49 vm-pg1 patroni[775]: 2026-08-16 18:11:49,368 INFO: no action. I am (vm-pg1), a secondary, and following a leader (vm-pg3)
asvpg@vm-pg1:~$

Aug 16 18:10:21 vm-pg2 patroni[775]: 2026-08-16 18:10:21,228 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:10:31 vm-pg2 patroni[775]: 2026-08-16 18:10:31,228 INFO: no action. I am (vm-pg2), the leader with the lock
Aug 16 18:10:36 vm-pg2 systemd[1]: Stopping patroni.service - Runners to orchestrate a high-availability PostgreSQL...
Aug 16 18:10:37 vm-pg2 systemd[1]: patroni.service: Deactivated successfully.
Aug 16 18:10:37 vm-pg2 systemd[1]: Stopped patroni.service - Runners to orchestrate a high-availability PostgreSQL.
Aug 16 18:10:37 vm-pg2 systemd[1]: patroni.service: Consumed 20.011s CPU time, 42.6M memory peak, 0B memory swap peak.
asvpg@vm-pg2:~$

asvpg@vm-pg2:~$ sudo systemctl start patroni
asvpg@vm-pg2:~$
asvpg@vm-pg2:~$ sudo journalctl -u patroni.service -n 40 -f
Aug 16 18:12:37 vm-pg2 patroni[3588]:   Data page checksum version: 0
Aug 16 18:12:37 vm-pg2 patroni[3588]:   Mock authentication nonce: 64ff4c4c751a10f364360e73a19655d6dd41d667752e8e896f7f2c21a3a0f3da
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,279 INFO: Lock owner: vm-pg3; I am vm-pg2
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,285 INFO: Local timeline=13 lsn=0/26010140
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,315 INFO: primary_timeline=14
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,318 INFO: primary: history=10        0/26000990        no recovery target specified
Aug 16 18:12:37 vm-pg2 patroni[3588]: 11        0/2600D330        no recovery target specified
Aug 16 18:12:37 vm-pg2 patroni[3588]: 12        0/2600FFE8        no recovery target specified
Aug 16 18:12:37 vm-pg2 patroni[3588]: 13        0/260101B8        no recovery target specified
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,318 INFO: Lock owner: vm-pg3; I am vm-pg2
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,318 INFO: starting as a secondary
Aug 16 18:12:37 vm-pg2 patroni[3617]: 2026-08-16 18:12:37.698 UTC [3617] LOG:  starting PostgreSQL 17.11 (Ubuntu 17.11-1.pgdg24.04+2) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0, 64-bit
Aug 16 18:12:37 vm-pg2 patroni[3617]: 2026-08-16 18:12:37.698 UTC [3617] LOG:  listening on IPv4 address "127.0.0.1", port 5432
Aug 16 18:12:37 vm-pg2 patroni[3588]: 2026-08-16 18:12:37,699 INFO: postmaster pid=3617
Aug 16 18:12:37 vm-pg2 patroni[3617]: 2026-08-16 18:12:37.703 UTC [3617] LOG:  listening on IPv4 address "10.130.0.28", port 5432
Aug 16 18:12:37 vm-pg2 patroni[3617]: 2026-08-16 18:12:37.706 UTC [3617] LOG:  listening on Unix socket "./.s.PGSQL.5432"
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.721 UTC [3621] LOG:  database system was shut down at 2026-08-16 18:10:36 UTC
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.721 UTC [3621] LOG:  entering standby mode
Aug 16 18:12:37 vm-pg2 patroni[3622]: 2026-08-16 18:12:37.728 UTC [3622] postgres@postgres FATAL:  the database system is starting up
Aug 16 18:12:37 vm-pg2 patroni[3618]: localhost:5432 - rejecting connections
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.737 UTC [3621] LOG:  consistent recovery state reached at 0/260101B8
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.737 UTC [3621] LOG:  invalid record length at 0/260101B8: expected at least 24, got 0
Aug 16 18:12:37 vm-pg2 patroni[3617]: 2026-08-16 18:12:37.737 UTC [3617] LOG:  database system is ready to accept read-only connections
Aug 16 18:12:37 vm-pg2 patroni[3624]: 2026-08-16 18:12:37.739 UTC [3624] postgres@postgres FATAL:  the database system is starting up
Aug 16 18:12:37 vm-pg2 patroni[3623]: localhost:5432 - rejecting connections
Aug 16 18:12:37 vm-pg2 patroni[3625]: 2026-08-16 18:12:37.756 UTC [3625] LOG:  fetching timeline history file for timeline 14 from primary server
Aug 16 18:12:37 vm-pg2 patroni[3625]: 2026-08-16 18:12:37.781 UTC [3625] LOG:  started streaming WAL from primary at 0/26000000 on timeline 13
Aug 16 18:12:37 vm-pg2 patroni[3625]: 2026-08-16 18:12:37.782 UTC [3625] LOG:  replication terminated by primary server
Aug 16 18:12:37 vm-pg2 patroni[3625]: 2026-08-16 18:12:37.782 UTC [3625] DETAIL:  End of WAL reached on timeline 13 at 0/260101B8.
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.783 UTC [3621] LOG:  new target timeline is 14
Aug 16 18:12:37 vm-pg2 patroni[3625]: 2026-08-16 18:12:37.783 UTC [3625] LOG:  restarted WAL streaming at 0/26000000 on timeline 14
Aug 16 18:12:37 vm-pg2 patroni[3621]: 2026-08-16 18:12:37.879 UTC [3621] LOG:  redo starts at 0/260101B8
Aug 16 18:12:38 vm-pg2 patroni[3588]: 2026-08-16 18:12:38,625 INFO: establishing a new patroni heartbeat connection to postgres
Aug 16 18:12:38 vm-pg2 patroni[3588]: 2026-08-16 18:12:38,641 INFO: establishing a new patroni restapi connection to postgres
Aug 16 18:12:38 vm-pg2 patroni[3629]: localhost:5432 - accepting connections
Aug 16 18:12:38 vm-pg2 patroni[3588]: 2026-08-16 18:12:38,869 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg3)
Aug 16 18:12:38 vm-pg2 patroni[3588]: 2026-08-16 18:12:38,925 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg3)
Aug 16 18:12:49 vm-pg2 patroni[3588]: 2026-08-16 18:12:49,426 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg3)
Aug 16 18:12:59 vm-pg2 patroni[3588]: 2026-08-16 18:12:59,426 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg3)
Aug 16 18:13:09 vm-pg2 patroni[3588]: 2026-08-16 18:13:09,426 INFO: no action. I am (vm-pg2), a secondary, and following a leader (vm-pg3)
asvpg@vm-pg2:~$

asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 14 |  0/260102F8 |   0 | 0/260102F8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 14 |  0/260102F8 |   0 | 0/260102F8 |   0 |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 14 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$
```

###
Подключение через pgbouncer
###
```sh
asvpg@vm-pg3:~$ sudo -u postgres psql -p 6432 -d thai -h 10.130.0.33
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)

thai=# \q
asvpg@vm-pg3:~$ sudo -u postgres psql -p 6432 -d thai -h 10.130.0.13
Password for user postgres:
psql (17.11 (Ubuntu 17.11-1.pgdg24.04+2))
Type "help" for help.

thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=# \q
asvpg@vm-pg3:~$
```

###
Проверим подключение через прокси. Указываем порт 5432, а прокси уже перенаправляет подключение через 6432 
###
```sh
asvpg@vm-haproxy1:~$ psql -h localhost -d thai -U postgres -p 5432
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#

asvpg@vm-haproxy2:~$ psql -h localhost -d thai -U postgres -p 5432
Password for user postgres:
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
Type "help" for help.

thai=# \dt+ book.*
                                         List of relations
 Schema |     Name     | Type  |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+-------+----------+-------------+---------------+------------+-------------
 book   | bus          | table | postgres | permanent   | heap          | 16 kB      |
 book   | busroute     | table | postgres | permanent   | heap          | 8192 bytes |
 book   | busstation   | table | postgres | permanent   | heap          | 16 kB      |
 book   | fam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | nam          | table | postgres | permanent   | heap          | 16 kB      |
 book   | ride         | table | postgres | permanent   | heap          | 6432 kB    |
 book   | schedule     | table | postgres | permanent   | heap          | 120 kB     |
 book   | seat         | table | postgres | permanent   | heap          | 40 kB      |
 book   | seatcategory | table | postgres | permanent   | heap          | 16 kB      |
 book   | tickets      | table | postgres | permanent   | heap          | 461 MB     |
(10 rows)

thai=#

--Мы попадаем на мастер
thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#
```

####
Выполним переключение ролей и проверим, что будет с открытой сессией через прокси к БД
####
```sh
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 11 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml switchover
Current cluster topology
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 11 |  0/2600D1E8 |   0 | 0/2600D1E8 |   0 |
| vm-pg3 | 10.130.0.33 | Leader  | running   | 11 |             |     |            |     |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
Primary [vm-pg3]:
Candidate ['vm-pg1', 'vm-pg2'] []: vm-pg1
When should the switchover take place (e.g. 2026-08-16T11:39 )  [now]:
Are you sure you want to switchover cluster patroni, demoting current leader vm-pg3? [y/N]: y
2026-08-16 10:40:03.76910 Successfully switched over to "vm-pg1"
+ Cluster: patroni (7673599298398442043) --+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State   | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running | 11 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | running | 11 |  0/2600D330 |   0 | 0/2600D330 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | stopped |    |     unknown |     |    unknown |     |
+--------+-------------+---------+---------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$ sudo patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni (7673599298398442043) ----+----+-------------+-----+------------+-----+
| Member | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| vm-pg1 | 10.130.0.13 | Leader  | running   | 12 |             |     |            |     |
| vm-pg2 | 10.130.0.28 | Replica | streaming | 12 |  0/2600E028 |   0 | 0/2600E028 |   0 |
| vm-pg3 | 10.130.0.33 | Replica | streaming | 12 |  0/2600E028 |   0 | 0/2600E028 |   0 |
+--------+-------------+---------+-----------+----+-------------+-----+------------+-----+
asvpg@vm-pg1:~$

--смотрим сессии на 2 нодах HAProxy и видим, что при попытке выполнить новый запрос вызывается переподключение
thai=# select 1;
FATAL:  terminating connection due to administrator command
FATAL:  server conn crashed?
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=# select pg_is_in_recovery();
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=#
thai=#
thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#


thai=# select 1;
FATAL:  terminating connection due to administrator command
FATAL:  server conn crashed?
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (16.14 (Ubuntu 16.14-0ubuntu0.24.04.1), server 17.11 (Ubuntu 17.11-1.pgdg24.04+2))
WARNING: psql major version 16, server major version 17.
         Some psql features might not work.
thai=# select 1;
 ?column?
----------
        1
(1 row)

thai=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

thai=#
```
