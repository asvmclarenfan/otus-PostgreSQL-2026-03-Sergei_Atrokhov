
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

