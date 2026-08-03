
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

