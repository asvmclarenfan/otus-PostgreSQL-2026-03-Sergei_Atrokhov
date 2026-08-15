

###
1. Настройка кластера ETCD
###
###

Intel
2 vCPU
2 ГБ RAM
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
