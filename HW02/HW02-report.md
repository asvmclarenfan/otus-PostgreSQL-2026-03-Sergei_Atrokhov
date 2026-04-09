
##
Создайте ВМ с Ubuntu 22.04/24.04 или подготовьте хост, на котором будет развёрнут Docker;
##

###
Проверка версии установленной ОС
###
```sh
asvpg@asvpg:/home$ cat /etc/os-release

PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
asvpg@asvpg:/home$
```

#
Далее устанвока бинарников и БД будет выполнена именно на ВМ, без использования докера.
Ранее уточнял информацию, было сказано, что разницы нет.
Если требуется именно через докер, просьба сообщить.
#


##
Создайте каталог для данных PostgreSQL на хосте: /var/lib/postgresql;
##

###
Переходим на пользователя postgres:
###
```sh
asvpg@asvpg:/home$ su - postgres
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

postgres@asvpg:~$ id
uid=1000(postgres) gid=1000(postgres) groups=1000(postgres),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)
postgres@asvpg:~$
```

###
проверяем целевой каталог:
###
```sh
postgres@asvpg:~$ ls /var/lib/postgresql
ls: cannot access '/var/lib/postgresql': No such file or directory
```

###
Создаем каталог (получаем ошибку):
###

```sh
postgres@asvpg:/var/lib$ mkdir -p /var/lib/postgresql
mkdir: cannot create directory ‘/var/lib/postgresql’: Permission denied
postgres@asvpg:/var/lib$
```

###
Т.к. раздел /var имеет права root:root, создаем целевой каталог под root (если это неправильно, просьба сообщить):
###

```sh
drwxr-xr-x  14 root root       4096 Jan 26 23:03 var

root@asvpg:~# id
uid=0(root) gid=0(root) groups=0(root)
root@asvpg:~# mkdir -p /var/lib/postgresql
root@asvpg:~# cd /var/lib

root@asvpg:/var/lib# ls -altr | grep postgresql
drwxr-xr-x  2 root                 root                 4096 Apr  7 16:08 postgresql
root@asvpg:/var/lib#
```

###
Установка пакетов\бинарников
###
```sh
asvpg@asvpg:/home$ sudo apt install -y postgresql-common
[sudo] password for asvpg: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  libllvm19
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libtypes-serialiser-perl postgresql-client-common
The following NEW packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libtypes-serialiser-perl postgresql-client-common postgresql-common
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 395 kB of archives.
After this operation, 1,326 kB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libjson-perl all 4.10000-1 [81.9 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 postgresql-client-common all 257build1.1 [36.4 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 postgresql-common all 257build1.1 [161 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libcommon-sense-perl amd64 3.75-3build3 [20.4 kB]
Get:5 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libtypes-serialiser-perl all 1.01-1 [11.6 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libjson-xs-perl amd64 4.040-0ubuntu0.24.04.1 [83.7 kB]
Fetched 395 kB in 4s (102 kB/s)            
Preconfiguring packages ...
Selecting previously unselected package libjson-perl.
(Reading database ... 150603 files and directories currently installed.)
Preparing to unpack .../0-libjson-perl_4.10000-1_all.deb ...
Unpacking libjson-perl (4.10000-1) ...
Selecting previously unselected package postgresql-client-common.
Preparing to unpack .../1-postgresql-client-common_257build1.1_all.deb ...
Unpacking postgresql-client-common (257build1.1) ...
Selecting previously unselected package postgresql-common.
Preparing to unpack .../2-postgresql-common_257build1.1_all.deb ...
Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
Unpacking postgresql-common (257build1.1) ...
Selecting previously unselected package libcommon-sense-perl:amd64.
Preparing to unpack .../3-libcommon-sense-perl_3.75-3build3_amd64.deb ...
Unpacking libcommon-sense-perl:amd64 (3.75-3build3) ...
Selecting previously unselected package libtypes-serialiser-perl.
Preparing to unpack .../4-libtypes-serialiser-perl_1.01-1_all.deb ...
Unpacking libtypes-serialiser-perl (1.01-1) ...
Selecting previously unselected package libjson-xs-perl.
Preparing to unpack .../5-libjson-xs-perl_4.040-0ubuntu0.24.04.1_amd64.deb ...
Unpacking libjson-xs-perl (4.040-0ubuntu0.24.04.1) ...
Setting up postgresql-client-common (257build1.1) ...
Setting up libcommon-sense-perl:amd64 (3.75-3build3) ...
Setting up libtypes-serialiser-perl (1.01-1) ...
Setting up libjson-perl (4.10000-1) ...
Setting up libjson-xs-perl (4.040-0ubuntu0.24.04.1) ...
Setting up postgresql-common (257build1.1) ...

Creating config file /etc/postgresql-common/createcluster.conf with new version
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
  en_us
Removing obsolete dictionary files:
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
Processing triggers for man-db (2.12.0-4build2) ...
asvpg@asvpg:/home$

asvpg@asvpg:/home$ sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
This script will enable the PostgreSQL APT repository on apt.postgresql.org on
your system. The distribution codename used will be noble-pgdg.

Press Enter to continue, or Ctrl-C to abort.

Using keyring /usr/share/postgresql-common/pgdg/apt.postgresql.org.gpg
Writing /etc/apt/sources.list.d/pgdg.sources ...

Running apt-get update ...
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]                                
Get:4 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1,875 kB]                     
Get:5 https://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease [180 kB]   
Get:6 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 Packages [441 kB]   
Get:7 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]                            
Get:8 http://ru.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [342 kB]                            
Get:9 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]                     
Get:10 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 c-n-f Metadata [16.9 kB]                                
Get:11 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [2,894 kB]                
Get:12 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,576 kB]                                   
Get:13 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [252 kB]                                                        
Get:14 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [21.6 kB]                                                     
Get:15 http://security.ubuntu.com/ubuntu noble-security/main amd64 c-n-f Metadata [10.7 kB]                                                 
Get:16 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [2,757 kB]                                                
Get:17 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [671 kB]                                                 
Get:18 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Components [212 B]                                                
Get:19 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 c-n-f Metadata [556 B]                                            
Get:20 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,663 kB]                                                 
Get:21 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [642 kB]                                                  
Get:22 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Components [208 B]                                                 
Get:23 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 c-n-f Metadata [544 B]                                             
Get:24 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1,167 kB]                                                  
Get:25 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [225 kB]                                                    
Get:26 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]                                                 
Get:27 http://security.ubuntu.com/ubuntu noble-security/universe amd64 c-n-f Metadata [22.8 kB]                                             
Get:28 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Packages [28.8 kB]                                                 
Get:29 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [6,980 B]                                                 
Get:30 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]                                                 
Get:31 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 c-n-f Metadata [396 B]                                             
Get:32 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [322 kB]                                                   
Get:33 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]                                                 
Get:34 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe Icons (64x64) [373 kB]                                                    
Get:35 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 c-n-f Metadata [34.2 kB]                                            
Get:36 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Packages [32.1 kB]                                                
Get:37 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [7,520 B]                                                
Get:38 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]                                                
Get:39 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 c-n-f Metadata [500 B]                                            
Get:40 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Packages [40.4 kB]                                                    
Get:41 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [7,380 B]                                                  
Get:42 http://ru.archive.ubuntu.com/ubuntu noble-backports/main Icons (48x48) [9,537 B]                                                     
Get:43 http://ru.archive.ubuntu.com/ubuntu noble-backports/main Icons (64x64) [11.3 kB]                                                     
Get:44 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 c-n-f Metadata [368 B]                                                
Get:45 http://ru.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [216 B]                                              
Get:46 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Packages [30.7 kB]                                                
Get:47 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe Translation-en [18.2 kB]                                                
Get:48 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [13.2 kB]                                              
Get:49 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe Icons (48x48) [23.3 kB]                                                 
Get:50 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe Icons (64x64) [32.3 kB]                                                 
Get:51 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 c-n-f Metadata [1,480 B]                                          
Get:52 http://ru.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Packages [780 B]                                                
Get:53 http://ru.archive.ubuntu.com/ubuntu noble-backports/multiverse Translation-en [372 B]                                                
Get:54 http://ru.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]                                              
Fetched 16.8 MB in 38s (445 kB/s)                                                                                                           
Reading package lists... Done

You can now start installing packages from apt.postgresql.org.

Have a look at https://wiki.postgresql.org/wiki/Apt for more information;
most notably the FAQ at https://wiki.postgresql.org/wiki/Apt/FAQ
asvpg@asvpg:/home$
```
###
Здесь было бы интересно узнать более подробно про установку пакетов
###


###
Проверяем, что директория postgresql имеет владельцем пользователя postgres:
###

```sh
asvpg@asvpg:/etc$ ls -altr | grep postgresql
drwxr-xr-x   2 postgres             postgres              4096 Aug  9  2024 postgresql
drwxr-xr-x   3 root                 root                  4096 Apr  7 16:21 postgresql-common
asvpg@asvpg:/etc$
```

###
Проверяем, что кроме пакетов никаких бинарников и процессов PostgreSQL нет:
###

```sh
svpg@asvpg:/etc$ cd /var/lib/postgresql/
asvpg@asvpg:/var/lib/postgresql$ ls -altr
total 8
drwxr-xr-x 70 root     root     4096 Apr  7 16:08 ..
drwxr-xr-x  2 postgres postgres 4096 Apr  7 16:08 .
asvpg@asvpg:/var/lib/postgresql$ cd /etc/postgresql
asvpg@asvpg:/etc/postgresql$ ls -altr
total 16
drwxr-xr-x   2 postgres postgres  4096 Aug  9  2024 .
drwxr-xr-x 140 root     root     12288 Apr  7 16:21 ..
asvpg@asvpg:/etc/postgresql$ ls -altr /usr/lib
lib/     lib64/   libexec/ 
asvpg@asvpg:/etc/postgresql$ ls -altr | grep /usr/lib/postgresql
asvpg@asvpg:/etc/postgresql$ 
asvpg@asvpg:/etc/postgresql$ ps auxf | grep postgres
asvpg       6323  0.0  0.0   9144  2268 pts/0    S+   16:36   0:00          \_ grep --color=auto postgres
asvpg@asvpg:/etc/postgresql$
```

###
Устанавливаем бинарники 18 версии: помимо самого сервера БД, устанавливаем клиентские утилиты для взаимодействия с сервером БД, а также дополнительные расширения. Судя по логу установки, владельцем бинарников является пользователь postgres; локаль и кодировка - юникод (UTF8):
###

```sh
asvpg@asvpg:/etc/postgresql$ sudo apt install -y postgresql-18 postgresql-client-18 postgresql-contrib
[sudo] password for asvpg: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libpq5 liburing2 postgresql-16 postgresql-18-jit postgresql-client-16 postgresql-client-common postgresql-common
Suggested packages:
  libpq-oauth postgresql-doc-16 postgresql-doc-18
The following NEW packages will be installed:
  libpq5 liburing2 postgresql-16 postgresql-18 postgresql-18-jit postgresql-client-16 postgresql-client-18 postgresql-contrib
The following packages will be upgraded:
  postgresql-client-common postgresql-common
2 upgraded, 8 newly installed, 0 to remove and 224 not upgraded.
Need to get 38.2 MB of archives.
After this operation, 140 MB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 liburing2 amd64 2.5-1build1 [21.1 kB]
Get:2 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-common all 290.pgdg24.04+1 [113 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 postgresql-contrib all 16+257build1.1 [11.6 kB]                          
Get:4 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-client-common all 290.pgdg24.04+1 [48.1 kB]                 
Get:5 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 libpq5 amd64 18.3-1.pgdg24.04+1 [255 kB]                               
Get:6 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-client-16 amd64 16.13-1.pgdg24.04+1 [1,929 kB]              
Get:7 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-16 amd64 16.13-1.pgdg24.04+1 [16.4 MB]                      
Get:8 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-client-18 amd64 18.3-1.pgdg24.04+1 [2,086 kB]               
Get:9 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-18 amd64 18.3-1.pgdg24.04+1 [7,527 kB]                      
Get:10 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 postgresql-18-jit amd64 18.3-1.pgdg24.04+1 [9,867 kB]                 
Fetched 38.2 MB in 3min 43s (172 kB/s)                                                                                                      
Preconfiguring packages ...
(Reading database ... 150824 files and directories currently installed.)
Preparing to unpack .../0-postgresql-common_290.pgdg24.04+1_all.deb ...
Leaving 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
Unpacking postgresql-common (290.pgdg24.04+1) over (257build1.1) ...
Preparing to unpack .../1-postgresql-client-common_290.pgdg24.04+1_all.deb ...
Unpacking postgresql-client-common (290.pgdg24.04+1) over (257build1.1) ...
Selecting previously unselected package libpq5:amd64.
Preparing to unpack .../2-libpq5_18.3-1.pgdg24.04+1_amd64.deb ...
Unpacking libpq5:amd64 (18.3-1.pgdg24.04+1) ...
Selecting previously unselected package liburing2:amd64.
Preparing to unpack .../3-liburing2_2.5-1build1_amd64.deb ...
Unpacking liburing2:amd64 (2.5-1build1) ...
Selecting previously unselected package postgresql-client-16.
Preparing to unpack .../4-postgresql-client-16_16.13-1.pgdg24.04+1_amd64.deb ...
Unpacking postgresql-client-16 (16.13-1.pgdg24.04+1) ...
Selecting previously unselected package postgresql-16.
Preparing to unpack .../5-postgresql-16_16.13-1.pgdg24.04+1_amd64.deb ...
Unpacking postgresql-16 (16.13-1.pgdg24.04+1) ...
Selecting previously unselected package postgresql-client-18.
Preparing to unpack .../6-postgresql-client-18_18.3-1.pgdg24.04+1_amd64.deb ...
Unpacking postgresql-client-18 (18.3-1.pgdg24.04+1) ...
Selecting previously unselected package postgresql-18.
Preparing to unpack .../7-postgresql-18_18.3-1.pgdg24.04+1_amd64.deb ...
Unpacking postgresql-18 (18.3-1.pgdg24.04+1) ...
Selecting previously unselected package postgresql-18-jit.
Preparing to unpack .../8-postgresql-18-jit_18.3-1.pgdg24.04+1_amd64.deb ...
Unpacking postgresql-18-jit (18.3-1.pgdg24.04+1) ...
Selecting previously unselected package postgresql-contrib.
Preparing to unpack .../9-postgresql-contrib_16+257build1.1_all.deb ...
Unpacking postgresql-contrib (16+257build1.1) ...
Setting up postgresql-client-common (290.pgdg24.04+1) ...
Removing obsolete conffile /etc/postgresql-common/supported_versions ...
Setting up libpq5:amd64 (18.3-1.pgdg24.04+1) ...
Setting up postgresql-common (290.pgdg24.04+1) ...
Installing new version of config file /etc/postgresql-common/pg_upgradecluster.d/analyze ...
Replacing config file /etc/postgresql-common/createcluster.conf with new version
Setting up liburing2:amd64 (2.5-1build1) ...
Setting up postgresql-client-18 (18.3-1.pgdg24.04+1) ...
update-alternatives: using /usr/share/postgresql/18/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
Setting up postgresql-client-16 (16.13-1.pgdg24.04+1) ...
Setting up postgresql-18 (18.3-1.pgdg24.04+1) ...
Creating new PostgreSQL cluster 18/main ...
/usr/lib/postgresql/18/bin/initdb -D /var/lib/postgresql/18/main --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are enabled.

fixing permissions on existing directory /var/lib/postgresql/18/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default "max_connections" ... 100
selecting default "shared_buffers" ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Setting up postgresql-18-jit (18.3-1.pgdg24.04+1) ...
Setting up postgresql-16 (16.13-1.pgdg24.04+1) ...
Setting up postgresql-contrib (16+257build1.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.6) ...
asvpg@asvpg:/etc/postgresql$
```

###
Проверяем созданные директории:
###
###
Директория файлов конфигурации:
###
```sh
asvpg@asvpg:/etc/postgresql/18/main$ pwd
/etc/postgresql/18/main
asvpg@asvpg:/etc/postgresql/18/main$ ls -altr
total 68
drwxr-xr-x 3 postgres postgres  4096 Apr  7 18:22 ..
-rw-r--r-- 1 postgres postgres   317 Apr  7 18:22 start.conf
-rw-r--r-- 1 postgres postgres   143 Apr  7 18:22 pg_ctl.conf
-rw-r----- 1 postgres postgres  2681 Apr  7 18:22 pg_ident.conf
-rw-r----- 1 postgres postgres  5934 Apr  7 18:22 pg_hba.conf
-rw-r--r-- 1 postgres postgres   315 Apr  7 18:22 environment
drwxr-xr-x 2 postgres postgres  4096 Apr  7 18:22 conf.d
-rw-r--r-- 1 postgres postgres 32755 Apr  7 18:22 postgresql.conf
drwxr-xr-x 3 postgres postgres  4096 Apr  7 18:22 .
asvpg@asvpg:/etc/postgresql/18/main$
```

###
Директория данных (права есть только у пользователя postgres):
###
```sh
asvpg@asvpg:/var/lib/postgresql/18$ pwd
/var/lib/postgresql/18
asvpg@asvpg:/var/lib/postgresql/18$ ls -altr
total 12
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 ..
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 .
drwx------ 19 postgres postgres 4096 Apr  7 18:22 main
asvpg@asvpg:/var/lib/postgresql/18$

asvpg@asvpg:/var/lib/postgresql/18$ su - postgres
Password: 
postgres@asvpg:~$ cd /var/lib/postgresql/18/main/
postgres@asvpg:/var/lib/postgresql/18/main$ ls -altr
total 92
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 ..
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_twophase
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_tblspc
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_stat_tmp
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_snapshots
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_serial
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_replslot
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_notify
drwx------  4 postgres postgres 4096 Apr  7 18:22 pg_multixact
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_dynshmem
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_commit_ts
-rw-------  1 postgres postgres    3 Apr  7 18:22 PG_VERSION
-rw-------  1 postgres postgres   88 Apr  7 18:22 postgresql.auto.conf
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_xact
drwx------  4 postgres postgres 4096 Apr  7 18:22 pg_wal
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_subtrans
drwx------  5 postgres postgres 4096 Apr  7 18:22 base
drwx------ 19 postgres postgres 4096 Apr  7 18:22 .
-rw-------  1 postgres postgres  130 Apr  7 18:22 postmaster.opts
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_stat
-rw-------  1 postgres postgres  108 Apr  7 18:22 postmaster.pid
drwx------  2 postgres postgres 4096 Apr  7 18:25 global
drwx------  4 postgres postgres 4096 Apr  7 18:27 pg_logical
postgres@asvpg:/var/lib/postgresql/18/main$
```

###
Директория логов:
###
```sh
postgres@asvpg:/var/log/postgresql$ pwd
/var/log/postgresql
postgres@asvpg:/var/log/postgresql$ ls -altr
total 12
drwxrwxr-x 17 root     syslog   4096 Apr  7 16:21 ..
drwxrwxr-t  2 root     postgres 4096 Apr  7 18:22 .
-rw-r-----  1 postgres postgres  941 Apr  7 18:27 postgresql-18-main.log
postgres@asvpg:/var/log/postgresql$
```

###
Директория бинарников (исполняемых файлов):
###
```sh
postgres@asvpg:/var/log/postgresql$ cd /usr/lib/postgresql/18/bin
postgres@asvpg:/usr/lib/postgresql/18/bin$ ls -altr
total 15396
-rwxr-xr-x 1 root root    39592 Feb 24 14:48 vacuumlo
-rwxr-xr-x 1 root root    93624 Feb 24 14:48 vacuumdb
-rwxr-xr-x 1 root root    89080 Feb 24 14:48 reindexdb
-rwxr-xr-x 1 root root   958040 Feb 24 14:48 psql
-rwxr-xr-x 1 root root 11413888 Feb 24 14:48 postgres
-rwxr-xr-x 1 root root    47640 Feb 24 14:48 pg_walsummary
-rwxr-xr-x 1 root root   105496 Feb 24 14:48 pg_waldump
-rwxr-xr-x 1 root root   117536 Feb 24 14:48 pg_verifybackup
-rwxr-xr-x 1 root root   188104 Feb 24 14:48 pg_upgrade
-rwxr-xr-x 1 root root    31112 Feb 24 14:48 pg_test_timing
-rwxr-xr-x 1 root root    39344 Feb 24 14:48 pg_test_fsync
-rwxr-xr-x 1 root root   117688 Feb 24 14:48 pg_rewind
-rwxr-xr-x 1 root root   204864 Feb 24 14:48 pg_restore
-rwxr-xr-x 1 root root    60200 Feb 24 14:48 pg_resetwal
-rwxr-xr-x 1 root root    56504 Feb 24 14:48 pg_recvlogical
-rwxr-xr-x 1 root root    93240 Feb 24 14:48 pg_receivewal
-rwxr-xr-x 1 root root    35448 Feb 24 14:48 pg_isready
-rwxr-xr-x 1 root root   123040 Feb 24 14:48 pg_dumpall
-rwxr-xr-x 1 root root   467824 Feb 24 14:48 pg_dump
-rwxr-xr-x 1 root root    68360 Feb 24 14:48 pg_ctl
-rwxr-xr-x 1 root root    84936 Feb 24 14:48 pg_createsubscriber
-rwxr-xr-x 1 root root    51624 Feb 24 14:48 pg_controldata
-rwxr-xr-x 1 root root    39304 Feb 24 14:48 pg_config
-rwxr-xr-x 1 root root   134040 Feb 24 14:48 pg_combinebackup
-rwxr-xr-x 1 root root    56056 Feb 24 14:48 pg_checksums
-rwxr-xr-x 1 root root   167768 Feb 24 14:48 pgbench
-rwxr-xr-x 1 root root   138816 Feb 24 14:48 pg_basebackup
-rwxr-xr-x 1 root root    39400 Feb 24 14:48 pg_archivecleanup
-rwxr-xr-x 1 root root   106072 Feb 24 14:48 pg_amcheck
-rwxr-xr-x 1 root root    43400 Feb 24 14:48 oid2name
-rwxr-xr-x 1 root root   122592 Feb 24 14:48 initdb
-rwxr-xr-x 1 root root    68280 Feb 24 14:48 dropuser
-rwxr-xr-x 1 root root    68344 Feb 24 14:48 dropdb
-rwxr-xr-x 1 root root    81240 Feb 24 14:48 createuser
-rwxr-xr-x 1 root root    76824 Feb 24 14:48 createdb
-rwxr-xr-x 1 root root    80696 Feb 24 14:48 clusterdb
drwxr-xr-x 4 root root     4096 Apr  7 18:21 ..
drwxr-xr-x 2 root root     4096 Apr  7 18:21 .
postgres@asvpg:/usr/lib/postgresql/18/bin$
```

###
По умолчанию запущен автостарт процессов postgres; проверяем, что экземпляр работает:
###
```sh
postgres@asvpg:/usr/lib/postgresql/18/bin$ ps auxf | grep postgres

postgres    9877  0.0  0.4 233808 34552 ?        Ss   18:22   0:00 /usr/lib/postgresql/18/bin/postgres -D /var/lib/postgresql/18/main -c config_file=/etc/postgresql/18/main/postgresql.conf
postgres    9878  0.0  0.1 233940  8620 ?        Ss   18:22   0:00  \_ postgres: 18/main: io worker 0
postgres    9879  0.0  0.0 233940  6908 ?        Ss   18:22   0:00  \_ postgres: 18/main: io worker 1
postgres    9880  0.0  0.0 233808  6336 ?        Ss   18:22   0:00  \_ postgres: 18/main: io worker 2
postgres    9881  0.0  0.1 233940 10520 ?        Ss   18:22   0:00  \_ postgres: 18/main: checkpointer 
postgres    9882  0.0  0.0 233968  7868 ?        Ss   18:22   0:00  \_ postgres: 18/main: background writer 
postgres    9884  0.0  0.1 233944 10816 ?        Ss   18:22   0:00  \_ postgres: 18/main: walwriter 
postgres    9885  0.0  0.1 235396  9732 ?        Ss   18:22   0:00  \_ postgres: 18/main: autovacuum launcher 
postgres    9886  0.0  0.1 235260  8856 ?        Ss   18:22   0:00  \_ postgres: 18/main: logical replication launcher 
postgres@asvpg:/usr/lib/postgresql/18/bin$ 

```

##
Постустановочные настройки
##

* пароль пользователя postgres установлен ранее
* прослушивание всех адресов
  * параметр статический, требует рестарта кластера БД
  ```
  postgres@asvpg:/usr/lib/postgresql/18/bin$ psql
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.
``` 
postgres=# show listen_addresses;
 listen_addresses 
------------------
 localhost
(1 row)

postgres=# alter system set listen_addresses = '*';
ALTER SYSTEM
postgres=# 
postgres=# show listen_addresses;
 listen_addresses 
------------------
 localhost
(1 row)


postgres=# select name, setting, context from pg_settings where name = 'listen_addresses';
       name       |  setting  |  context   
------------------+-----------+------------
 listen_addresses | localhost | postmaster
(1 row)

postgres=#

  ```
* меняем порт на 5433
```sh
postgres=# alter system set port = 5433;
ALTER SYSTEM
postgres=#
```
###
На уроке изменение выполнялось через конфиг файл postgresql.conf (в моем понимании - аналог PFILE в Oracle (текстовый файл; все изменения только через рестарт); читал, что изменения лучше вносить через auto.conf файл через команду ALTER SYSTEM (динамические параметры меняются через перечитывание конфигурации - pg_reload_conf()) - если это не так, просьба уточнить).
В директории /etc/postgresql/18/main нет файла auto.conf, он находится в директории с данными:
###
```sh
postgres@asvpg:/var/lib/postgresql/18/main$ pwd
/var/lib/postgresql/18/main
postgres@asvpg:/var/lib/postgresql/18/main$ ls -altr
total 92
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 ..
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_twophase
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_tblspc
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_stat_tmp
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_snapshots
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_serial
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_replslot
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_notify
drwx------  4 postgres postgres 4096 Apr  7 18:22 pg_multixact
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_dynshmem
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_commit_ts
-rw-------  1 postgres postgres    3 Apr  7 18:22 PG_VERSION
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_xact
drwx------  4 postgres postgres 4096 Apr  7 18:22 pg_wal
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_subtrans
drwx------  5 postgres postgres 4096 Apr  7 18:22 base
-rw-------  1 postgres postgres  111 Apr  7 19:20 postgresql.auto.conf
drwx------  4 postgres postgres 4096 Apr  7 19:42 pg_logical
drwx------ 19 postgres postgres 4096 Apr  7 19:42 .
-rw-------  1 postgres postgres  130 Apr  7 19:42 postmaster.opts
drwx------  2 postgres postgres 4096 Apr  7 19:42 pg_stat
-rw-------  1 postgres postgres  101 Apr  7 19:42 postmaster.pid
drwx------  2 postgres postgres 4096 Apr  7 19:43 global
postgres@asvpg:/var/lib/postgresql/18/main$
```
* редактирование конфиг файла pg_hba.conf
  * добавляем строку для подключений с любого IP к любой БД (в рамках тестового окружения)
```sh
# Allow replication connections from localhost, by a user with the
# replication privilege.
host all all 0.0.0.0/0 scram-sha-256
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
postgres@asvpg:/etc/postgresql/18/main$
```

###
По курсу хотелось бы подробнее услышать пояснения насчет назначения и содержимого данного конфиг файла, best practices по его настройке.
###

###
Просмотр установленных директорий конфиг файлов и директории данных и логов:
###

```sh
postgres=# show config_file;
               config_file               
-----------------------------------------
 /etc/postgresql/18/main/postgresql.conf
(1 row)

postgres=# show hba_file;
              hba_file               
-------------------------------------
 /etc/postgresql/18/main/pg_hba.conf
(1 row)


postgres=# show data_directory;
       data_directory        
-----------------------------
 /var/lib/postgresql/18/main
(1 row)

postgres=# show log_directory;
 log_directory 
---------------
 log
(1 row)

postgres=#
```

###
Рестарт кластера БД и проверка:
###

```sh
postgres@asvpg:/etc/postgresql/18/main$ sudo pg_ctlcluster 18 main restart
[sudo] password for postgres: 
postgres@asvpg:/etc/postgresql/18/main$ 
postgres@asvpg:/etc/postgresql/18/main$ ps auxf | grep postgres
postgres   11384  0.2  0.4 233808 34516 ?        Ss   19:42   0:00 /usr/lib/postgresql/18/bin/postgres -D /var/lib/postgresql/18/main -c config_file=/etc/postgresql/18/main/postgresql.conf
postgres   11385  0.0  0.0 233808  7412 ?        Ss   19:42   0:00  \_ postgres: 18/main: io worker 0
postgres   11386  0.0  0.0 233808  6748 ?        Ss   19:42   0:00  \_ postgres: 18/main: io worker 1
postgres   11387  0.0  0.0 233808  6056 ?        Ss   19:42   0:00  \_ postgres: 18/main: io worker 2
postgres   11388  0.0  0.0 233944  6216 ?        Ss   19:42   0:00  \_ postgres: 18/main: checkpointer 
postgres   11389  0.0  0.0 233808  6208 ?        Ss   19:42   0:00  \_ postgres: 18/main: background writer 
postgres   11391  0.0  0.0 233808  6248 ?        Ss   19:42   0:00  \_ postgres: 18/main: walwriter 
postgres   11392  0.0  0.1 235392  9384 ?        Ss   19:42   0:00  \_ postgres: 18/main: autovacuum launcher 
postgres   11393  0.0  0.1 235256  8668 ?        Ss   19:42   0:00  \_ postgres: 18/main: logical replication launcher 
postgres@asvpg:/etc/postgresql/18/main$
```

###
Проверка изменения параметра listen_addresses и port:
###
```sh
postgres@asvpg:/etc/postgresql/18/main$ psql
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

postgres=# show listen_addresses;
 listen_addresses 
------------------
 *
(1 row)

postgres=#

postgres=# show port;
 port 
------
 5433
(1 row)

postgres=#
```

* проверка имеющегося кластера БД (должен быть один main):
```sh
postgres@asvpg:/var/lib/postgresql/18/main$ pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:/var/lib/postgresql/18/main$
```

* проверка подключения к БД под пользователем postgres (без аутентификации на уровне ОС):
###
ошибка пароля (ввожу пароль как на ОС)
###
```sh
postgres@asvpg:/etc/postgresql/18/main$ psql -h localhost -p 5433
Password for user postgres: 
psql: error: connection to server at "localhost" (127.0.0.1), port 5433 failed: FATAL:  password authentication failed for user "postgres"
connection to server at "localhost" (127.0.0.1), port 5433 failed: FATAL:  password authentication failed for user "postgres"
```
###
Меняю пароль в БД для пользователя postgres:
###
```
postgres@asvpg:/etc/postgresql/18/main$ psql
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

postgres=# ALTER USER postgres WITH PASSWORD <pwd>;
ALTER ROLE
postgres=# exit
```
###
Проверяем еще раз, теперь ошибки нет:
###
```sh
postgres@asvpg:/etc/postgresql/18/main$ psql -h localhost -p 5433
Password for user postgres: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

postgres=# \conninfo
            Connection Information
      Parameter       |         Value          
----------------------+------------------------
 Database             | postgres
 Client User          | postgres
 Host                 | localhost
 Host Address         | 127.0.0.1
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 11659
 SSL Connection       | true
 SSL Library          | OpenSSL
 SSL Protocol         | TLSv1.3
 SSL Key Bits         | 256
 SSL Cipher           | TLS_AES_256_GCM_SHA384
 SSL Compression      | false
 ALPN                 | postgresql
 Superuser            | on
 Hot Standby          | off
(19 rows)

postgres=#
```

###
Проверяем список имеющихся БД в кластере:
###
```sh
postgres=# \l
                                                     List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+--------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |             |             |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |             |             |        |           | postgres=CTc/postgres
(3 rows)

postgres=#
```

###
Создаем новую тестовую БД
###
```sh
postgres=# create database otus_dba1;
CREATE DATABASE
postgres=# \l
                                                     List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+--------+-----------+-----------------------
 otus_dba1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |             |             |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |             |             |        |           | postgres=CTc/postgres
(4 rows)

postgres=#
```

###
Создадим тестовую таблицу с PK, проверим список столбцов и индексов:
###
```sh
postgres@asvpg:~$ psql -h localhost -d otus_dba1 -p 5433
Password for user postgres: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

otus_dba1=# create table tb_test (
id bigint GENERATED BY DEFAULT AS IDENTITY,
name text,
now_date TIMESTAMP default CURRENT_TIMESTAMP);
CREATE TABLE
otus_dba1=# 
otus_dba1=# alter table tb_test add constraint pk_tb_test primary key (id);
ALTER TABLE
otus_dba1=# 
otus_dba1=# \d+ tb_test
                                                                 Table "public.tb_test"
  Column  |            Type             | Collation | Nullable |             Default              | Storage  | Compression | Stats target | Description 
----------+-----------------------------+-----------+----------+----------------------------------+----------+-------------+--------------+-------------
 id       | bigint                      |           | not null | generated by default as identity | plain    |             |              | 
 name     | text                        |           |          |                                  | extended |             |              | 
 now_date | timestamp without time zone |           |          | CURRENT_TIMESTAMP                | plain    |             |              | 
Indexes:
    "pk_tb_test" PRIMARY KEY, btree (id)
Not-null constraints:
    "tb_test_id_not_null" NOT NULL "id"
Access method: heap

otus_dba1=# select * from pg_indexes where tablename = 'tb_test';
 schemaname | tablename | indexname  | tablespace |                             indexdef                              
------------+-----------+------------+------------+-------------------------------------------------------------------
 public     | tb_test   | pk_tb_test |            | CREATE UNIQUE INDEX pk_tb_test ON public.tb_test USING btree (id)
(1 row)

otus_dba1=#
```

###
Вставим тестовые 3 строки, проверим содержимое таблицы запросом:
###
```sh
otus_dba1=# insert into tb_test (name) values ('Test1');
INSERT 0 1
otus_dba1=# insert into tb_test (name) values ('Test2');
INSERT 0 1
otus_dba1=# insert into tb_test (name) values ('Test3');
INSERT 0 1
otus_dba1=# select * from tb_test;
 id | name  |          now_date          
----+-------+----------------------------
  1 | Test1 | 2026-04-07 22:21:12.575669
  2 | Test2 | 2026-04-07 22:21:16.054283
  3 | Test3 | 2026-04-07 22:21:19.026294
(3 rows)

otus_dba1=#
```

###
Далее настроен удаленный доступ с ноутбука на ВМ через DBeaver:
в ВМ Network -> Attached to Bridged Adapter.
В ВМ проверяем IP:
###
```sh
postgres@asvpg:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:04:67 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.169/24 brd 192.168.0.255 scope global dynamic noprefixroute enp0s3
...
```
###
На ноутбуке пинг проходит:
###
```sh
C:\Users\asus>ping 192.168.0.169

Pinging 192.168.0.169 with 32 bytes of data:
Reply from 192.168.0.169: bytes=32 time<1ms TTL=64
Reply from 192.168.0.169: bytes=32 time<1ms TTL=64
Reply from 192.168.0.169: bytes=32 time<1ms TTL=64
Reply from 192.168.0.169: bytes=32 time<1ms TTL=64

Ping statistics for 192.168.0.169:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\Users\asus>
```

###
Создаем подключение к БД через DBeaver и проверяем наличие таблицы с данными:
###
<img width="1854" height="1238" alt="hw02_2026-04-07 223015" src="https://github.com/user-attachments/assets/aee80c61-214d-4bfb-b2b3-7431dc06258d" />


#
Установка Docker Engine
#
```sh
asvpg@asvpg:~$ curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker 
Command 'curl' not found, but can be installed with:
sudo snap install curl  # version 8.19.0, or
sudo apt  install curl  # version 8.5.0-2ubuntu10.8
See 'snap info curl' for additional versions.
asvpg@asvpg:~$
```sh

###
отсутствовал пакет curl, устанавливаем:
###

```sh
asvpg@asvpg:~$ sudo apt  install curl
[sudo] password for asvpg: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcurl3t64-gnutls libcurl4t64
The following NEW packages will be installed:
  curl
The following packages will be upgraded:
  libcurl3t64-gnutls libcurl4t64
2 upgraded, 1 newly installed, 0 to remove and 222 not upgraded.
Need to get 902 kB of archives.
After this operation, 534 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libcurl4t64 amd64 8.5.0-2ubuntu10.8 [342 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 curl amd64 8.5.0-2ubuntu10.8 [227 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libcurl3t64-gnutls amd64 8.5.0-2ubuntu10.8 [334 kB]
Fetched 902 kB in 8s (113 kB/s)                                                
(Reading database ... 155018 files and directories currently installed.)
Preparing to unpack .../libcurl4t64_8.5.0-2ubuntu10.8_amd64.deb ...
Unpacking libcurl4t64:amd64 (8.5.0-2ubuntu10.8) over (8.5.0-2ubuntu10.6) ...
Selecting previously unselected package curl.
Preparing to unpack .../curl_8.5.0-2ubuntu10.8_amd64.deb ...
Unpacking curl (8.5.0-2ubuntu10.8) ...
Preparing to unpack .../libcurl3t64-gnutls_8.5.0-2ubuntu10.8_amd64.deb ...
Unpacking libcurl3t64-gnutls:amd64 (8.5.0-2ubuntu10.8) over (8.5.0-2ubuntu10.6) 
...
Setting up libcurl4t64:amd64 (8.5.0-2ubuntu10.8) ...
Setting up libcurl3t64-gnutls:amd64 (8.5.0-2ubuntu10.8) ...
Setting up curl (8.5.0-2ubuntu10.8) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.6) ...
asvpg@asvpg:~$
```

###
повторяем попытку установки:
###

```sh
asvpg@asvpg:~$ curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
# Executing docker install script, commit: c04fb16bb8bd8ed6ce884bb40570cbcd6101ae0c
+ sh -c apt-get -qq update >/dev/null
--!!!выполнялось около 10 минут


+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install ca-certificates curl >/dev/null
+ sh -c install -m 0755 -d /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
+ sh -c chmod a+r /etc/apt/keyrings/docker.asc
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin docker-model-plugin >/dev/null

--!!! выполнялось около 35 минут!!! Такая плохая скорость скачивания?

Using systemd to manage Docker service
+ sh -c systemctl enable --now docker.service
  UNIT                       LOAD   ACTIVE SUB       DESCRIPTION
  proc-sys-fs-binfmt_misc.a… loaded active running   Arbitrary Executable File …
  sys-devices-pci0000:00-00… loaded active plugged   VBOX_CD-ROM
  sys-devices-pci0000:00-00… loaded active plugged   82540EM Gigabit Ethernet C…
  sys-devices-pci0000:00-00… loaded active plugged   /sys/devices/pci0000:00/00…
  sys-devices-pci0000:00-00… loaded active plugged   VBOX_HARDDISK 1
  sys-devices-pci0000:00-00… loaded active plugged   VBOX_HARDDISK 2
  sys-devices-pci0000:00-00… loaded active plugged   VBOX_HARDDISK
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-platform-seri… loaded active plugged   /sys/devices/platform/seri…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-block… loaded active plugged   /sys/devices/virtual/block…
  sys-devices-virtual-misc-… loaded active plugged   /sys/devices/virtual/misc/…
  sys-devices-virtual-net-d… loaded active plugged   /sys/devices/virtual/net/d…
  sys-devices-virtual-tty-t… loaded active plugged   /sys/devices/virtual/tty/t…
  sys-module-configfs.device loaded active plugged   /sys/module/configfs
  sys-module-fuse.device     loaded active plugged   /sys/module/fuse
  sys-subsystem-net-devices… loaded active plugged   /sys/subsystem/net/devices…
  sys-subsystem-net-devices… loaded active plugged   82540EM Gigabit Ethernet C…
  -.mount                    loaded active mounted   Root Mount
  dev-hugepages.mount        loaded active mounted   Huge Pages File System
  dev-mqueue.mount           loaded active mounted   POSIX Message Queue File S…
  proc-sys-fs-binfmt_misc.m… loaded active mounted   Arbitrary Executable File …
  run-snapd-ns-snapd\x2ddes… loaded active mounted   /run/snapd/ns/snapd-deskto…
  run-snapd-ns.mount         loaded active mounted   /run/snapd/ns
  run-user-1001-doc.mount    loaded active mounted   /run/user/1001/doc
  run-user-1001-gvfs.mount   loaded active mounted   /run/user/1001/gvfs
  run-user-1001.mount        loaded active mounted   /run/user/1001
  snap-bare-5.mount          loaded active mounted   Mount unit for bare, revis…
  snap-core22-2045.mount     loaded active mounted   Mount unit for core22, rev…
  snap-firefox-6565.mount    loaded active mounted   Mount unit for firefox, re…
  snap-firmware\x2dupdater-… loaded active mounted   Mount unit for firmware-up…
  snap-gnome\x2d42\x2d2204-… loaded active mounted   Mount unit for gnome-42-22…
  snap-gtk\x2dcommon\x2dthe… loaded active mounted   Mount unit for gtk-common-…
  snap-snap\x2dstore-1270.m… loaded active mounted   Mount unit for snap-store,…
  snap-snapd-24792.mount     loaded active mounted   Mount unit for snapd, revi…
  snap-snapd-26382.mount     loaded active mounted   Mount unit for snapd, revi…
  snap-snapd\x2ddesktop\x2d… loaded active mounted   Mount unit for snapd-deskt…
  sys-fs-fuse-connections.m… loaded active mounted   FUSE Control File System
  sys-kernel-config.mount    loaded active mounted   Kernel Configuration File …
  sys-kernel-debug.mount     loaded active mounted   Kernel Debug File System
  sys-kernel-tracing.mount   loaded active mounted   Kernel Trace File System
  cups.path                  loaded active running   CUPS Scheduler
  systemd-ask-password-plym… loaded active waiting   Forward Password Requests …
  systemd-ask-password-wall… loaded active waiting   Forward Password Requests …
  whoopsie.path              loaded active waiting   Start whoopsie on modifica…
  init.scope                 loaded active running   System and Service Manager
  session-2.scope            loaded active running   Session 2 of User asvpg
  accounts-daemon.service    loaded active running   Accounts Service
  alsa-restore.service       loaded active exited    Save/Restore Sound Card St…
  apparmor.service           loaded active exited    Load AppArmor profiles
  apport.service             loaded active exited    automatic crash report gen…
  avahi-daemon.service       loaded active running   Avahi mDNS/DNS-SD Stack
  colord.service             loaded active running   Manage, Install and Genera…
  console-setup.service      loaded active exited    Set console font and keymap
  containerd.service         loaded active running   containerd container runti…
  cron.service               loaded active running   Regular background program…
  cups-browsed.service       loaded active running   Make remote CUPS printers …
  cups.service               loaded active running   CUPS Scheduler
  dbus.service               loaded active running   D-Bus System Message Bus
  docker.service             loaded active running   Docker Application Contain…
  fwupd.service              loaded active running   Firmware update daemon
  gdm.service                loaded active running   GNOME Display Manager
  gnome-remote-desktop.serv… loaded active running   GNOME Remote Desktop
  kerneloops.service         loaded active running   Tool to automatically coll…
  keyboard-setup.service     loaded active exited    Set the console keyboard l…
  kmod-static-nodes.service  loaded active exited    Create List of Static Devi…
  ModemManager.service       loaded active running   Modem Manager
  NetworkManager-dispatcher… loaded active running   Network Manager Script Dis…
  NetworkManager-wait-onlin… loaded active exited    Network Manager Wait Online
  NetworkManager.service     loaded active running   Network Manager
  openvpn.service            loaded active exited    OpenVPN service
  packagekit.service         loaded active running   PackageKit Daemon
  plymouth-quit-wait.service loaded active exited    Hold until boot process fi…
  plymouth-read-write.servi… loaded active exited    Tell Plymouth To Write Out…
  plymouth-start.service     loaded active exited    Show Plymouth Boot Screen
  polkit.service             loaded active running   Authorization Manager
  postgresql.service         loaded active exited    PostgreSQL RDBMS
  postgresql@18-main.service loaded active running   PostgreSQL Cluster 18-main
  power-profiles-daemon.ser… loaded active running   Power Profiles daemon
  rsyslog.service            loaded active running   System Logging Service
  rtkit-daemon.service       loaded active running   RealtimeKit Scheduling Pol…
  setvtrgb.service           loaded active exited    Set console scheme
  snapd.apparmor.service     loaded active exited    Load AppArmor profiles man…
  snapd.seeded.service       loaded active exited    Wait until snapd is fully …
  snapd.service              loaded active running   Snap Daemon
  switcheroo-control.service loaded active running   Switcheroo Control Proxy s…
  sysstat.service            loaded active exited    Resets System Activity Logs
  systemd-binfmt.service     loaded active exited    Set Up Additional Binary F…
  systemd-journal-flush.ser… loaded active exited    Flush Journal to Persisten…
  systemd-journald.service   loaded active running   Journal Service
  systemd-logind.service     loaded active running   User Login Management
  systemd-modules-load.serv… loaded active exited    Load Kernel Modules
  systemd-oomd.service       loaded active running   Userspace Out-Of-Memory (O…
  systemd-random-seed.servi… loaded active exited    Load/Save OS Random Seed
  systemd-remount-fs.service loaded active exited    Remount Root and Kernel Fi…
  systemd-resolved.service   loaded active running   Network Name Resolution
  systemd-sysctl.service     loaded active exited    Apply Kernel Variables
  systemd-timesyncd.service  loaded active running   Network Time Synchronizati…
  systemd-tmpfiles-setup-de… loaded active exited    Create Static Device Nodes…
  systemd-tmpfiles-setup-de… loaded active exited    Create Static Device Nodes…
  systemd-tmpfiles-setup.se… loaded active exited    Create Volatile Files and …
  systemd-udev-trigger.serv… loaded active exited    Coldplug All udev Devices
  systemd-udevd.service      loaded active running   Rule-based Manager for Dev…
  systemd-update-utmp.servi… loaded active exited    Record System Boot/Shutdow…
  systemd-user-sessions.ser… loaded active exited    Permit User Sessions
  udisks2.service            loaded active running   Disk Manager
  ufw.service                loaded active exited    Uncomplicated firewall
  unattended-upgrades.servi… loaded active running   Unattended Upgrades Shutdo…
  upower.service             loaded active running   Daemon for power management
  user-runtime-dir@1001.ser… loaded active exited    User Runtime Directory /ru…
  user@1001.service          loaded active running   User Manager for UID 1001
  vboxadd-service.service    loaded active exited    vboxadd-service.service
● vboxadd.service            loaded failed failed    vboxadd.service
  wpa_supplicant.service     loaded active running   WPA supplicant
  -.slice                    loaded active active    Root Slice
  system-getty.slice         loaded active active    Slice /system/getty
  system-modprobe.slice      loaded active active    Slice /system/modprobe
  system-postgresql.slice    loaded active active    Slice /system/postgresql
  system.slice               loaded active active    System Slice
  user-1001.slice            loaded active active    User Slice of UID 1001
  user.slice                 loaded active active    User and Session Slice
  avahi-daemon.socket        loaded active running   Avahi mDNS/DNS-SD Stack Ac…
  cups.socket                loaded active running   CUPS Scheduler
  dbus.socket                loaded active running   D-Bus System Message Bus S…
  docker.socket              loaded active running   Docker Socket for the API
  snapd.socket               loaded active running   Socket activation for snap…
  syslog.socket              loaded active running   Syslog Socket
  systemd-fsckd.socket       loaded active listening fsck to fsckd communicatio…
  systemd-initctl.socket     loaded active listening initctl Compatibility Name…
  systemd-journald-dev-log.… loaded active running   Journal Socket (/dev/log)
  systemd-journald.socket    loaded active running   Journal Socket
  systemd-oomd.socket        loaded active running   Userspace Out-Of-Memory (O…
  systemd-rfkill.socket      loaded active listening Load/Save RF Kill Switch S…
  systemd-sysext.socket      loaded active listening System Extension Image Man…
  systemd-udevd-control.soc… loaded active running   udev Control Socket
  systemd-udevd-kernel.sock… loaded active running   udev Kernel Socket
  uuidd.socket               loaded active listening UUID daemon activation soc…
  swap.img.swap              loaded active active    /swap.img
  basic.target               loaded active active    Basic System
  cryptsetup.target          loaded active active    Local Encrypted Volumes
  getty-pre.target           loaded active active    Preparation for Logins
  getty.target               loaded active active    Login Prompts
  graphical.target           loaded active active    Graphical Interface
  integritysetup.target      loaded active active    Local Integrity Protected …
  local-fs-pre.target        loaded active active    Preparation for Local File…
  local-fs.target            loaded active active    Local File Systems
  multi-user.target          loaded active active    Multi-User System
  network-online.target      loaded active active    Network is Online
  network-pre.target         loaded active active    Preparation for Network
  network.target             loaded active active    Network
  nss-lookup.target          loaded active active    Host and Network Name Look…
  nss-user-lookup.target     loaded active active    User and Group Name Lookups
  paths.target               loaded active active    Path Units
  remote-fs.target           loaded active active    Remote File Systems
  slices.target              loaded active active    Slice Units
  snapd.mounts-pre.target    loaded active active    Mounting snaps
  snapd.mounts.target        loaded active active    Mounted snaps
  sockets.target             loaded active active    Socket Units
  sound.target               loaded active active    Sound Card
  swap.target                loaded active active    Swaps
  sysinit.target             loaded active active    System Initialization
  time-set.target            loaded active active    System Time Set
  timers.target              loaded active active    Timer Units
  veritysetup.target         loaded active active    Local Verity Protected Vol…
  anacron.timer              loaded active waiting   Trigger anacron every hour
  apt-daily-upgrade.timer    loaded active waiting   Daily apt upgrade and clea…
  apt-daily.timer            loaded active waiting   Daily apt download activit…
  dpkg-db-backup.timer       loaded active waiting   Daily dpkg database backup…
  e2scrub_all.timer          loaded active waiting   Periodic ext4 Online Metad…
  fstrim.timer               loaded active waiting   Discard unused filesystem …
  fwupd-refresh.timer        loaded active waiting   Refresh fwupd metadata reg…
  logrotate.timer            loaded active waiting   Daily rotation of log files
  man-db.timer               loaded active waiting   Daily man-db regeneration
  motd-news.timer            loaded active waiting   Message of the Day
  sysstat-collect.timer      loaded active waiting   Run system activity accoun…
  sysstat-summary.timer      loaded active waiting   Generate summary of yester…
  systemd-tmpfiles-clean.ti… loaded active waiting   Daily Cleanup of Temporary…
  update-notifier-download.… loaded active waiting   Download data for packages…
  update-notifier-motd.timer loaded active waiting   Check to see whether there…

Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.

217 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
INFO: Docker daemon enabled and started

+ sh -c docker version
Client: Docker Engine - Community
 Version:           29.4.0
 API version:       1.54
 Go version:        go1.26.1
 Git commit:        9d7ad9f
 Built:             Tue Apr  7 08:36:07 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.4.0
  API version:      1.54 (minimum version 1.40)
  Go version:       go1.26.1
  Git commit:       daa0cb7
  Built:            Tue Apr  7 08:36:07 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.2
  GitCommit:        301b2dac98f15c27117da5c8af12118a041a31d9
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

[sudo] password for asvpg: 
asvpg@asvpg:~$ 
```

###
Проверяем, что группа создана:
###
```sh
asvpg@asvpg:~$ cut -d: -f1 /etc/group | sort
...
docker
...
```

###
Добавляем себя в группу docker:
###
```sh
asvpg@asvpg:~$ sudo usermod -a -G docker asvpg
asvpg@asvpg:~$ sudo usermod -a -G docker postgres

asvpg@asvpg:~$ su - asvpg
Password: 
asvpg@asvpg:~$ 
asvpg@asvpg:~$ 
asvpg@asvpg:~$ groups
asvpg adm cdrom sudo dip plugdev users lpadmin docker
asvpg@asvpg:~$
```

###
Поиск официального образа postgres в реестре:
###
```sh
asvpg@asvpg:~$ docker search postgres
NAME                       DESCRIPTION                                     STARS     OFFICIAL
postgres                   The PostgreSQL object-relational database sy…   14871     [OK]
cimg/postgres                                                              9         
circleci/postgres          The PostgreSQL object-relational database sy…   35        
kasmweb/postgres           Postgres image maintained by Kasm Technologi…   6         
elestio/postgres           Postgres, verified and packaged by Elestio      2         
ubuntu/postgres            PostgreSQL is an open source object-relation…   43        
chainguard/postgres        Build, ship and run secure software with Cha…   1         
cleanstart/postgres        Secure by Design, Built for Speed, Hardened …   0         
artifacthub/postgres                                                       0         
geokrety/postgres          Postgres with postgis + quantile and amqp ex…   0         
corpusops/postgres         https://github.com/corpusops/docker-images/     0         
rootpublic/postgres                                                        0         
dockette/postgres          My PostgreSQL image with tunning and preinst…   1         
vulhub/postgres                                                            1         
wayofdev/postgres                                                          0         
pgrouting/postgres          Postgres Docker images with PostGIS and dep…   0         
uselagoon/postgres                                                         0         
openeuler/postgres                                                         0         
clarinpl/postgres                                                          0         
supabase/postgres          Unmodified Postgres with some useful plugins…   75        
mosipdev/postgres                                                          0         
brimstone/postgres         postgres image with traefik-cert support        0         
trainlineeurope/postgres   Extended version of official Postgres https:…   0         
blacklabelops/postgres     Postgres Image for Atlassian Applications       4         
fredboat/postgres          PostgreSQL 10.0 used in FredBoat's docker-co…   1         
asvpg@asvpg:~$ 
```

###
Проверка отсутствия образов:
###
```sh
asvpg@asvpg:~$ docker images
                                                                                             i Info →   U  In Use
IMAGE   ID             DISK USAGE   CONTENT SIZE   EXTRA
asvpg@asvpg:~$
```

###
Загрузка образа на хост:
###
```sh
asvpg@asvpg:~$ docker pull postgres:18
18: Pulling from library/postgres
464872ed796d: Download complete 
f4deb76cb231: Download complete 
484cf2ee784f: Download complete 
7a79188b0f04: Downloading [==========================>                        ]  60.82MB/115.3MB
b21c9fd89a08: Download complete 
11c182969642: Download complete 
eb4cdb4c077a: Download complete 
fecac0093525: Download complete 
2140605cc9e3: Download complete 
5435b2dcdf5c: Downloading [===============>                                   ]  8.946MB/29.78MB
8d02d282ee5a: Download complete 
baf06005f9e9: Download complete 
7cec73702fc3: Download complete 
ffb2fc153481: Download complete 
3f82016dd5ae: Download complete
--очень медленная скорость с обрывом
failed to copy: read tcp 192.168.1.6:53244->172.64.66.1:443: read: connection reset by peer
asvpg@asvpg:~$ docker pull postgres:18
18: Pulling from library/postgres
464872ed796d: Download complete 
f4deb76cb231: Pull complete 
fecac0093525: Pull complete 
7cec73702fc3: Pull complete 
5435b2dcdf5c: Pull complete 
eb4cdb4c077a: Pull complete 
baf06005f9e9: Pull complete 
8d02d282ee5a: Download complete 
484cf2ee784f: Download complete 
7a79188b0f04: Downloading [==>                                                ]  6.291MB/115.3MB
7a79188b0f04: Downloading [===>                                               ]  7.422MB/115.3MB
b21c9fd89a08: Download complete 
2140605cc9e3: Pull complete 
ffb2fc153481: Download complete 
3f82016dd5ae: Downloading [========>                                          ]  1.049MB/5.957MB
3f82016dd5ae: Downloading [==============>                                    ]  1.671MB/5.957MB
failed to copy: read tcp 192.168.1.6:46358->172.64.66.1:443: read: connection reset by peer
asvpg@asvpg:~$ 
```

###
с третьей попытки получилось (ушло около 2.5 часов)
###
```sh
asvpg@asvpg:~$ docker pull postgres:18
18: Pulling from library/postgres
464872ed796d: Pull complete 
5435b2dcdf5c: Pull complete 
fecac0093525: Pull complete 
7a79188b0f04: Pull complete 
eb4cdb4c077a: Pull complete 
7cec73702fc3: Pull complete 
484cf2ee784f: Pull complete 
f4deb76cb231: Pull complete 
baf06005f9e9: Pull complete 
11c182969642: Pull complete 
8d02d282ee5a: Pull complete 
b21c9fd89a08: Pull complete 
2140605cc9e3: Pull complete 
ffb2fc153481: Download complete 
3f82016dd5ae: Download complete 
Digest: sha256:52e6ffd11fddd081ae63880b635b2a61c14008c17fc98cdc7ce5472265516dd0
Status: Downloaded newer image for postgres:18
docker.io/library/postgres:18
asvpg@asvpg:~$
```

###
проверяем наличие образа:
###

```sh
asvpg@asvpg:~$ docker images
                                                                                             i Info →   U  In Use
IMAGE         ID             DISK USAGE   CONTENT SIZE   EXTRA
postgres:18   52e6ffd11fdd        649MB          168MB        
asvpg@asvpg:~$
```

###
Проверяем, что без пароля будет ошибка:
###
```sh
asvpg@asvpg:~$ docker run postgres:18
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html
asvpg@asvpg:~$
```

###
Каталог /var/lib/postgresql уже ранее создан, принадлежит пользователю postgres
###
```sh
asvpg@asvpg:/var/lib$ ls -altr /var/lib/postgres
ls: cannot access '/var/lib/postgres': No such file or directory
asvpg@asvpg:/var/lib$

drwxr-xr-x  3 postgres             postgres             4096 Apr  7 18:22 postgresql
```

###
Получается, создаем другой каталог /var/lib/pg_docker
###
```sh
asvpg@asvpg:/var/lib$ sudo mkdir -p /var/lib/pg_docker
[sudo] password for asvpg: 
asvpg@asvpg:/var/lib$ ls -altr /var/lib/pg_docker
total 8
drwxr-xr-x 74 root root 4096 Apr  9 15:16 ..
drwxr-xr-x  2 root root 4096 Apr  9 15:16 .
asvpg@asvpg:/var/lib$

drwxr-xr-x  2 root                 root                 4096 Apr  9 15:16 pg_docker
```

###
Запускаем контейнер с проброской порта 5432 для внешнего подключения:
###
```sh
asvpg@asvpg:/var/lib$ docker run --rm -d \
        --name postgres18 \
        -e POSTGRES_PASSWORD=123 \
        -p 5432:5432 \
        -v /var/lib/pg_docker:/var/lib/postgresql/18/docker \
        postgres:18
19c467b015296465a49fcf7fd4f0d545c6f40ddcbbdc8348666786c2708cc52e
asvpg@asvpg:/var/lib$
```

###
ВОПРОС: получается, все файлы находятся в /var/lib/pg_docker. Для чего тогда /var/lib/postgresql/18/docker?
###
```sh
root@asvpg:~# cd /var/lib/pg_docker/
root@asvpg:/var/lib/pg_docker# ls -altr
total 136
drwxr-xr-x 74 root    root             4096 Apr  9 15:16 ..
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_twophase
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_tblspc
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_stat_tmp
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_snapshots
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_serial
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_replslot
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_notify
drwx------  4 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_multixact
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_dynshmem
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_commit_ts
-rw-------  1 dnsmasq systemd-journal     3 Apr  9 15:20 PG_VERSION
-rw-------  1 dnsmasq systemd-journal 32557 Apr  9 15:20 postgresql.conf
-rw-------  1 dnsmasq systemd-journal    88 Apr  9 15:20 postgresql.auto.conf
-rw-------  1 dnsmasq systemd-journal  2681 Apr  9 15:20 pg_ident.conf
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_xact
drwx------  4 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_wal
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_subtrans
-rw-------  1 dnsmasq systemd-journal  5753 Apr  9 15:20 pg_hba.conf
drwx------ 19 dnsmasq root             4096 Apr  9 15:20 .
-rw-------  1 dnsmasq systemd-journal    36 Apr  9 15:20 postmaster.opts
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:20 pg_stat
-rw-------  1 dnsmasq systemd-journal    99 Apr  9 15:20 postmaster.pid
drwx------  6 dnsmasq systemd-journal  4096 Apr  9 15:23 base
drwx------  2 dnsmasq systemd-journal  4096 Apr  9 15:23 global
drwx------  4 dnsmasq systemd-journal  4096 Apr  9 15:25 pg_logical
root@asvpg:/var/lib/pg_docker#
```

###
Проверяем наличие активного контейнера:
###
```sh
asvpg@asvpg:/var/lib$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                         NAMES
19c467b01529   postgres:18   "docker-entrypoint.s…"   41 seconds ago   Up 41 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   postgres18
asvpg@asvpg:/var/lib$
```

###
Заходим в контейнер в режиме терминала, создаем новую БД, новую таблицу с 1 строкой:
###
```sh
asvpg@asvpg:/var/lib$ docker exec -it postgres18 bash
root@19c467b01529:/# su postgres
postgres@19c467b01529:/$ psql
psql (18.3 (Debian 18.3-1.pgdg13+1))
Type "help" for help.

postgres=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | postgres
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5432
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 89
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

postgres=# create database test_docker;
CREATE DATABASE
postgres=# \c test_docker;
You are now connected to database "test_docker" as user "postgres".
test_docker=# create table test_docker (id int, name text);
CREATE TABLE
test_docker=# insert into test_docker values(1, 'test');
INSERT 0 1
test_docker=# select * from test_docker;
 id | name 
----+------
  1 | test
(1 row)

test_docker=# 
```

###
При попытке создать внешнее подключение через DBeaver, получаю таймаут..неправильно определяю IP (172.18.0.1)?
###

```sh
asvpg@asvpg:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:04:67 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.6/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 73367sec preferred_lft 73367sec
    inet6 fe80::a00:27ff:fe27:467/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether aa:99:57:2f:60:f4 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::a899:57ff:fe2f:60f4/64 scope link 
       valid_lft forever preferred_lft forever
10: br-deefc43700c9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 56:50:1a:0b:71:4b brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
    inet6 fe80::5450:1aff:fe0b:714b/64 scope link 
       valid_lft forever preferred_lft forever
11: veth483010c@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-deefc43700c9 state UP group default 
    link/ether ee:90:93:fa:7c:2f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::ec90:93ff:fefa:7c2f/64 scope link 
       valid_lft forever preferred_lft forever
12: veth0448566@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-deefc43700c9 state UP group default 
    link/ether b6:a1:68:1f:65:90 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::b4a1:68ff:fe1f:6590/64 scope link 
       valid_lft forever preferred_lft forever
asvpg@asvpg:~$
```


###
Контейнеры друг друга не видят:
###
```sh
asvpg@asvpg:/var/lib$ sudo docker run -it --rm --name pg-client postgres:18 \
        psql -h postgres18 -U postgres
[sudo] password for asvpg: 
psql: error: could not translate host name "postgres18" to address: Name or service not known
asvpg@asvpg:/var/lib$
```

###
Останавливаю контейнер, он автоматически будет удален (опция rm -d):
###
```sh
asvpg@asvpg:/var/lib$ docker stop postgres18
postgres18
asvpg@asvpg:/var/lib$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
asvpg@asvpg:/var/lib$
```

###
Создаем общую сеть, но запустить сразу не можем, т.к. удалили контейнер postgresql18
###

```sh
asvpg@asvpg:/var/lib$ sudo docker network create pg-net
deefc43700c9709975d4739bd32820f0c468dc44f5489462dfdca5be06d30662
asvpg@asvpg:/var/lib$ docker network connect pg-net postgres18
Error response from daemon: No such container: postgres18
asvpg@asvpg:/var/lib$
```

###
Выполняем запуск:
###
```sh
asvpg@asvpg:/var/lib$ docker run -d --rm \
  --name postgres18 \
  --network pg-net \
  -e POSTGRES_PASSWORD=123 \
  -v /var/lib/pg_docker:/var/lib/postgresql/18/docker \
  -p 5432:5432 \
  postgres:18
a0bade20bd4921505fd88ce1ff98bf102b5392a48c7d271355a2e6196ce05cf5

asvpg@asvpg:/var/lib$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                         NAMES
a0bade20bd49   postgres:18   "docker-entrypoint.s…"   8 seconds ago   Up 6 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   postgres18
asvpg@asvpg:/var/lib$
```

###
Запускаем еще один контейнер в той же сети:
###
```sh
asvpg@asvpg:/var/lib$ sudo docker run -it --rm \
        --name pg-client \
        --network pg-net \
        postgres:18 \
        psql -h postgres18 -U postgres
Password for user postgres: 
psql (18.3 (Debian 18.3-1.pgdg13+1))
Type "help" for help.

postgres=#
```

###
Контейнеры видят друг друга; БД и таблица с данными на месте:
###
```sh
postgres=# \c test_docker;
You are now connected to database "test_docker" as user "postgres".
test_docker=# select * from test_docker;
 id | name 
----+------
  1 | test
(1 row)

test_docker=#
```

###
ВОПРОС: В другом сеансе не смог проверить из-за блокировки. Что неправильно сделал?
###
```sh
asvpg@asvpg:~$ docker ps
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
asvpg@asvpg:~$
```

