
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

