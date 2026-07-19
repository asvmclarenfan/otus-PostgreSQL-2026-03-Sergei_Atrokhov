###
Реализовать свой миникластер на трех виртуальных машинах
###

###
Для создания кластера из трех ВМ будем использовать внутреннюю сеть VB (Internal Network). В этом режиме ВМ видят только друг друга и представляют из себя изолированное окружпение для настройки репликации.
На момент начала ДЗ имеется одна ВМ с настроенным PG18.3 на Ubuntu 24, 2 новые ВМ создадим через полное клонирование имеющейся ВМ.

Настройка сетевой связанности:
для каждой ВМ во вкладке Адаптер 1 выбираем тип подключения Internal Network со стандартным именем intnet. 
Все ВМ, подключенные к одной внутренней сети с одинаковым именем, должны автоматически увидят друг друга 
Тип адаптера -Intel PRO/1000 MT Server для лучшей совместимости с драйверами Linux.

Настройка IP-адресации:
выполняем через настройку статических адресов 
Загрузите каждую ВМ и задайте фиксированные адреса в файле конфигурации сети вашей ОС (обычно /etc/netplan/*.yaml для Ubuntu или файлы в /etc/sysconfig/network-scripts/ для RHEL-дистрибутивов):

Машина	IP-адрес	      Маска
ВМ1	   192.168.50.11	255.255.255.0 (/24)
ВМ2	   192.168.50.12	255.255.255.0 (/24)
ВМ3	   192.168.50.13	255.255.255.0 (/24)
###

```sh
--ВМ1
asvpg@asvpg:~$ sudo bash
[sudo] password for asvpg: 
root@asvpg:/home/asvpg# cd /etc/netplan/
root@asvpg:/etc/netplan# ls -altr
total 24
-rw-r--r--   1 root root   104 Aug  5  2025 01-network-manager-all.yaml
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x   2 root root  4096 Jan 26 23:03 .
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..

root@asvpg:/etc/netplan# sudo netplan apply

** (generate:4593): WARNING **: 13:18:43.102: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4591): WARNING **: 13:18:43.369: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4591): WARNING **: 13:18:43.461: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.
root@asvpg:/etc/netplan# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:04:67 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.11/24 brd 192.168.50.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe27:467/64 scope link 
       valid_lft forever preferred_lft forever
3: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 42:b3:48:ff:00:4f brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether fe:d4:e5:57:6f:43 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
root@asvpg:/etc/netplan#

--ВМ2
root@asvpg:/etc/netplan# sudo netplan apply

** (generate:4818): WARNING **: 13:13:07.975: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4816): WARNING **: 13:13:08.226: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4816): WARNING **: 13:13:08.309: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.
root@asvpg:/etc/netplan# cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.50.12/24]
root@asvpg:/etc/netplan# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:85:d5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.12/24 brd 192.168.50.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe27:85d5/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 32:a4:28:b7:ce:e1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether b2:ff:6f:05:8e:18 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
root@asvpg:/etc/netplan#

--ВМ3
root@asvpg:/etc/netplan# nano 01-network-manager-all.yaml 
root@asvpg:/etc/netplan# cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.50.13/24]
root@asvpg:/etc/netplan# sudo netplan apply

** (generate:4486): WARNING **: 13:22:26.495: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4484): WARNING **: 13:22:26.795: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.

** (process:4484): WARNING **: 13:22:26.873: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.
root@asvpg:/etc/netplan# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1c:ec:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.13/24 brd 192.168.50.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1c:ec6f/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 86:33:75:e0:fc:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 86:80:c4:8a:14:a8 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
root@asvpg:/etc/netplan#

--Если оставить, как есть, то работать с текущими настройками не будет.
--Пинг не проходит!
--Перенастраиваем имя renderer и права на yaml:

root@asvpg:/etc/netplan# cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.50.11/24]
root@asvpg:/etc/netplan#

root@asvpg:/etc/netplan# chmod 600 /etc/netplan/01-network-manager-all.yaml
root@asvpg:/etc/netplan# ls -altr
total 24
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..
-rw-------   1 root root   175 Jul 19 14:02 01-network-manager-all.yaml
drwxr-xr-x   2 root root  4096 Jul 19 14:02 .
root@asvpg:/etc/netplan#


root@asvpg:/etc/netplan# cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.50.12/24]
root@asvpg:/etc/netplan#

root@asvpg:/etc/netplan# ls -altr
total 24
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..
-rw-r--r--   1 root root   175 Jul 19 14:08 01-network-manager-all.yaml
drwxr-xr-x   2 root root  4096 Jul 19 14:08 .
root@asvpg:/etc/netplan# chmod 600 /etc/netplan/01-network-manager-all.yaml
root@asvpg:/etc/netplan# ls -altr
total 24
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..
-rw-------   1 root root   175 Jul 19 14:08 01-network-manager-all.yaml
drwxr-xr-x   2 root root  4096 Jul 19 14:08 .
root@asvpg:/etc/netplan#


root@asvpg:/etc/netplan# cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.50.13/24]
root@asvpg:/etc/netplan#

root@asvpg:/etc/netplan# ls -altr
total 24
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..
-rw-r--r--   1 root root   175 Jul 19 14:06 01-network-manager-all.yaml
drwxr-xr-x   2 root root  4096 Jul 19 14:06 .
root@asvpg:/etc/netplan# chmod 600 /etc/netplan/01-network-manager-all.yaml
root@asvpg:/etc/netplan# ls -altr
total 24
-rw-------   1 root root    65 Jan 26 23:03 50-cloud-init.yaml
drwxr-xr-x 143 root root 12288 Jun 13 15:30 ..
-rw-------   1 root root   175 Jul 19 14:06 01-network-manager-all.yaml
drwxr-xr-x   2 root root  4096 Jul 19 14:06 .
root@asvpg:/etc/netplan#

--Проверяем сетевую связность каждой ВМ с каждой через ping:

--ВМ1
root@asvpg:/home/asvpg# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:04:67 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.11/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 56:83:10:cb:c0:ae brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 42:f7:34:9c:e7:f4 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
root@asvpg:/home/asvpg# ping -c 10 192.168.50.12
PING 192.168.50.12 (192.168.50.12) 56(84) bytes of data.
64 bytes from 192.168.50.12: icmp_seq=1 ttl=64 time=1.14 ms
64 bytes from 192.168.50.12: icmp_seq=2 ttl=64 time=0.350 ms
64 bytes from 192.168.50.12: icmp_seq=3 ttl=64 time=0.929 ms
64 bytes from 192.168.50.12: icmp_seq=4 ttl=64 time=0.487 ms
64 bytes from 192.168.50.12: icmp_seq=5 ttl=64 time=0.347 ms
64 bytes from 192.168.50.12: icmp_seq=6 ttl=64 time=0.548 ms
64 bytes from 192.168.50.12: icmp_seq=7 ttl=64 time=0.729 ms
64 bytes from 192.168.50.12: icmp_seq=8 ttl=64 time=0.543 ms
64 bytes from 192.168.50.12: icmp_seq=9 ttl=64 time=0.552 ms
64 bytes from 192.168.50.12: icmp_seq=10 ttl=64 time=0.657 ms

--- 192.168.50.12 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9549ms
rtt min/avg/max/mdev = 0.347/0.628/1.138/0.236 ms
root@asvpg:/home/asvpg# 
root@asvpg:/home/asvpg# 
root@asvpg:/home/asvpg# 
root@asvpg:/home/asvpg# ping -c 10 192.168.50.13
PING 192.168.50.13 (192.168.50.13) 56(84) bytes of data.
64 bytes from 192.168.50.13: icmp_seq=1 ttl=64 time=1.22 ms
64 bytes from 192.168.50.13: icmp_seq=2 ttl=64 time=0.372 ms
64 bytes from 192.168.50.13: icmp_seq=3 ttl=64 time=0.371 ms
64 bytes from 192.168.50.13: icmp_seq=4 ttl=64 time=0.451 ms
64 bytes from 192.168.50.13: icmp_seq=5 ttl=64 time=0.479 ms
64 bytes from 192.168.50.13: icmp_seq=6 ttl=64 time=0.257 ms
64 bytes from 192.168.50.13: icmp_seq=7 ttl=64 time=0.621 ms
64 bytes from 192.168.50.13: icmp_seq=8 ttl=64 time=0.409 ms
64 bytes from 192.168.50.13: icmp_seq=9 ttl=64 time=0.550 ms
64 bytes from 192.168.50.13: icmp_seq=10 ttl=64 time=0.618 ms

--- 192.168.50.13 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9368ms
rtt min/avg/max/mdev = 0.257/0.534/1.220/0.253 ms
root@asvpg:/home/asvpg#

--ВМ2
root@asvpg:/etc/netplan# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:27:85:d5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.12/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 3e:21:24:0e:b0:c5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether ee:73:fd:c2:80:76 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# ping -c 10 192.168.50.11
PING 192.168.50.11 (192.168.50.11) 56(84) bytes of data.
64 bytes from 192.168.50.11: icmp_seq=1 ttl=64 time=0.860 ms
64 bytes from 192.168.50.11: icmp_seq=2 ttl=64 time=0.880 ms
64 bytes from 192.168.50.11: icmp_seq=3 ttl=64 time=0.584 ms
64 bytes from 192.168.50.11: icmp_seq=4 ttl=64 time=0.595 ms
64 bytes from 192.168.50.11: icmp_seq=5 ttl=64 time=0.570 ms
64 bytes from 192.168.50.11: icmp_seq=6 ttl=64 time=0.470 ms
64 bytes from 192.168.50.11: icmp_seq=7 ttl=64 time=0.420 ms
64 bytes from 192.168.50.11: icmp_seq=8 ttl=64 time=0.431 ms
64 bytes from 192.168.50.11: icmp_seq=9 ttl=64 time=0.671 ms
64 bytes from 192.168.50.11: icmp_seq=10 ttl=64 time=0.762 ms

--- 192.168.50.11 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9401ms
rtt min/avg/max/mdev = 0.420/0.624/0.880/0.158 ms
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# ping -c 10 192.168.50.13
PING 192.168.50.13 (192.168.50.13) 56(84) bytes of data.
64 bytes from 192.168.50.13: icmp_seq=1 ttl=64 time=1.02 ms
64 bytes from 192.168.50.13: icmp_seq=2 ttl=64 time=0.470 ms
64 bytes from 192.168.50.13: icmp_seq=3 ttl=64 time=0.426 ms
64 bytes from 192.168.50.13: icmp_seq=4 ttl=64 time=0.757 ms
64 bytes from 192.168.50.13: icmp_seq=5 ttl=64 time=0.454 ms
64 bytes from 192.168.50.13: icmp_seq=6 ttl=64 time=0.380 ms
64 bytes from 192.168.50.13: icmp_seq=7 ttl=64 time=0.854 ms
64 bytes from 192.168.50.13: icmp_seq=8 ttl=64 time=0.390 ms
64 bytes from 192.168.50.13: icmp_seq=9 ttl=64 time=0.339 ms
64 bytes from 192.168.50.13: icmp_seq=10 ttl=64 time=0.590 ms

--- 192.168.50.13 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9715ms
rtt min/avg/max/mdev = 0.339/0.568/1.020/0.220 ms
root@asvpg:/etc/netplan# 


--ВМ3
root@asvpg:/etc/netplan# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1c:ec:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.13/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 6e:0f:56:53:29:26 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-deefc43700c9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 22:44:58:40:a6:f6 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-deefc43700c9
       valid_lft forever preferred_lft forever
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# ping -c 10 192.168.50.11
PING 192.168.50.11 (192.168.50.11) 56(84) bytes of data.
64 bytes from 192.168.50.11: icmp_seq=1 ttl=64 time=0.653 ms
64 bytes from 192.168.50.11: icmp_seq=2 ttl=64 time=0.438 ms
64 bytes from 192.168.50.11: icmp_seq=3 ttl=64 time=0.453 ms
64 bytes from 192.168.50.11: icmp_seq=4 ttl=64 time=0.465 ms
64 bytes from 192.168.50.11: icmp_seq=5 ttl=64 time=0.347 ms
64 bytes from 192.168.50.11: icmp_seq=6 ttl=64 time=0.490 ms
64 bytes from 192.168.50.11: icmp_seq=7 ttl=64 time=0.559 ms
64 bytes from 192.168.50.11: icmp_seq=8 ttl=64 time=0.481 ms
64 bytes from 192.168.50.11: icmp_seq=9 ttl=64 time=0.476 ms
64 bytes from 192.168.50.11: icmp_seq=10 ttl=64 time=0.464 ms

--- 192.168.50.11 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9720ms
rtt min/avg/max/mdev = 0.347/0.482/0.653/0.075 ms
root@asvpg:/etc/netplan# 
root@asvpg:/etc/netplan# ping -c 10 192.168.50.12
PING 192.168.50.12 (192.168.50.12) 56(84) bytes of data.
64 bytes from 192.168.50.12: icmp_seq=1 ttl=64 time=0.801 ms
64 bytes from 192.168.50.12: icmp_seq=2 ttl=64 time=0.845 ms
64 bytes from 192.168.50.12: icmp_seq=3 ttl=64 time=0.477 ms
64 bytes from 192.168.50.12: icmp_seq=4 ttl=64 time=0.785 ms
64 bytes from 192.168.50.12: icmp_seq=5 ttl=64 time=0.436 ms
64 bytes from 192.168.50.12: icmp_seq=6 ttl=64 time=0.424 ms
64 bytes from 192.168.50.12: icmp_seq=7 ttl=64 time=0.562 ms
64 bytes from 192.168.50.12: icmp_seq=8 ttl=64 time=0.563 ms
64 bytes from 192.168.50.12: icmp_seq=9 ttl=64 time=0.303 ms
64 bytes from 192.168.50.12: icmp_seq=10 ttl=64 time=0.415 ms

--- 192.168.50.12 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9538ms
rtt min/avg/max/mdev = 0.303/0.561/0.845/0.178 ms
root@asvpg:/etc/netplan#
```

###
Выполняем предварительные настройки по параметрам БД и pg_hba на всех 3 ВМ:
###
otus_dba1=# 
otus_dba1=# select name, setting, context from pg_settings where name = 'wal_level';
   name    | setting |  context   
-----------+---------+------------
 wal_level | replica | postmaster
(1 row)

otus_dba1=# alter system set wal_level = 'logical';
ALTER SYSTEM
otus_dba1=# \q
postgres@asvpg:~$

--Т.к. параметр статический, выполняем рестарт кластера БД:
postgres@asvpg:~$ sudo pg_lsclusters 18 main status
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ sudo pg_ctlcluster 18 main stop
Warning: The unit file, source configuration file or drop-ins of postgresql@18-main.service changed on disk. Run 'systemctl daemon-reload' to reload units.
postgres@asvpg:~$ sudo pg_lsclusters 18 main status
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 down   postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ sudo pg_ctlcluster 18 main start
Warning: The unit file, source configuration file or drop-ins of postgresql@18-main.service changed on disk. Run 'systemctl daemon-reload' to reload units.
postgres@asvpg:~$ sudo pg_lsclusters 18 main status
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$

```sh
otus_dba1=# select name, setting, context from pg_settings where name in ('wal_level', 'max_replication_slots', 'max_wal_senders', 'listen_addresses');
         name          | setting |  context   
-----------------------+---------+------------
 listen_addresses      | *       | postmaster
 max_replication_slots | 10      | postmaster
 max_wal_senders       | 10      | postmaster
 wal_level             | logical | postmaster
(4 rows)

otus_dba1=#

--Настройка pg_hba:
postgres@asvpg:/etc/postgresql/18/main$ pwd
/etc/postgresql/18/main
postgres@asvpg:/etc/postgresql/18/main$ cat pg_hba.conf 
...
# Allow replication connections from localhost, by a user with the
# replication privilege.
host    replication     repl_user        192.168.50.0/24        scram-sha-256
host    all             postgres         192.168.50.0/24        scram-sha-256
host all all 0.0.0.0/0 scram-sha-256
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
postgres@asvpg:/etc/postgresql/18/main$ 

--создаем пользователя для репликации:
otus_dba1=# create role repl_user with replication login encrypted password 'repl_user';
CREATE ROLE
otus_dba1=#

--Выполняем рестарт кластера БД и проверяем, что он успешно стартован:
postgres@asvpg:/etc/postgresql/18/main$ sudo pg_ctlcluster 18 main stop
postgres@asvpg:/etc/postgresql/18/main$ sudo pg_lsclusters 18 main
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 down   postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:/etc/postgresql/18/main$

postgres@asvpg:/etc/postgresql/18/main$ sudo pg_ctlcluster 18 main start
postgres@asvpg:/etc/postgresql/18/main$ sudo pg_lsclusters 18 main
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:/etc/postgresql/18/main$

--Подготовка завершена, приступаем к непосредственному выполнению ДЗ
```

###
1. Настройте ВМ1:
Создайте таблицу test, которая будет для операций записи
Создайте таблицу test2, которая будет для чтения
Настройте публикацию таблицы test
###
```sh
otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

otus_dba1=# CREATE TABLE test (id int4 primary key, descr text);
CREATE TABLE
                                           ^
otus_dba1=# CREATE TABLE test2 (id int4 primary key, ro_data text);
CREATE TABLE
otus_dba1=#

--создаем публикацию таблицы test:
otus_dba1=# CREATE PUBLICATION pub_vm1 FOR TABLE test;
CREATE PUBLICATION
otus_dba1=#
```

###
2. Настройте ВМ2:
Создайте таблицу test2, которая будет для операций записи
Создайте таблицу test, которая будет для чтения
Настройте публикацию таблицы test2
Сделайте подписку таблицы test на публикацию таблицы test с ВМ1
###
```sh
otus_dba1=# CREATE TABLE test (id int4 primary key, descr text);
CREATE TABLE
otus_dba1=# CREATE TABLE test2 (id int4 primary key, ro_data text);
CREATE TABLE

--создаем публикацию таблицы test2:
otus_dba1=# CREATE PUBLICATION pub_vm2 FOR TABLE test2;
CREATE PUBLICATION
otus_dba1=#
--создаем подписку таблицы test на публикацию таблицы test с ВМ1, по умолчанию система использует дефолтовый порт 5432, у меня используется порт 5433. Если не поменять, будет ошибка:
otus_dba1=# CREATE SUBSCRIPTION sub_from_vm1 CONNECTION 'host=192.168.50.11 dbname=otus_dba1 user=repl_user password=repl_user' PUBLICATION pub_vm1;
ERROR:  subscription "sub_from_vm1" could not connect to the publisher: connection to server at "192.168.50.11", port 5432 failed: Connection refused
	Is the server running on that host and accepting TCP/IP connections?
otus_dba1=# CREATE SUBSCRIPTION sub_from_vm1 CONNECTION 'host=192.168.50.11 dbname=otus_dba1 port=5433 user=repl_user password=repl_user' PUBLICATION pub_vm1;
NOTICE:  created replication slot "sub_from_vm1" on publisher
CREATE SUBSCRIPTION
otus_dba1=#
--настроена начальная синхронизация таблицы test с ВМ1
```

###
3. на ВМ1:
Сделайте подписку таблицы test2 на публикацию таблицы test2 с ВМ2
###
```sh
otus_dba1=# CREATE SUBSCRIPTION sub_from_vm2 CONNECTION 'host=192.168.50.12 dbname=otus_dba1 port=5433 user=repl_user password=repl_user' PUBLICATION pub_vm2;
NOTICE:  created replication slot "sub_from_vm2" on publisher
CREATE SUBSCRIPTION
otus_dba1=#


otus_dba1=# SELECT pubname, puballtables FROM pg_publication;
 pubname | puballtables 
---------+--------------
 pub_vm1 | f
(1 row)

otus_dba1=# SELECT * FROM pg_publication_tables WHERE pubname = 'pub_vm1';
 pubname | schemaname | tablename |  attnames  | rowfilter 
---------+------------+-----------+------------+-----------
 pub_vm1 | public     | test      | {id,descr} | 
(1 row)

otus_dba1=# 
```

###
4. Настройте ВМ3:
Создайте таблицы: test и test2
Подпишите test на публикацию таблицы test с ВМ1
Подпишите test2 на публикацию таблицы test2 с ВМ2
Используйте этот узел для чтения объединённых данных и резервного копирования
###
```sh
otus_dba1=# CREATE TABLE test (id int4 primary key, descr text);
CREATE TABLE
otus_dba1=# CREATE TABLE test2 (id int4 primary key, ro_data text);
CREATE TABLE

--подписка на таблицу test с ВМ1:
otus_dba1=# CREATE SUBSCRIPTION sub_test_from_vm1 CONNECTION 'host=192.168.50.11 dbname=otus_dba1 port=5433 user=repl_user password=repl_user' PUBLICATION pub_vm1;
NOTICE:  created replication slot "sub_test_from_vm1" on publisher
CREATE SUBSCRIPTION
otus_dba1=#

--подписка на таблицу test2 с ВМ2:
otus_dba1=# CREATE SUBSCRIPTION sub_test2_from_vm2 CONNECTION 'host=192.168.50.12 dbname=otus_dba1 port=5433 user=repl_user password=repl_user' PUBLICATION pub_vm2;
NOTICE:  created replication slot "sub_test2_from_vm2" on publisher
CREATE SUBSCRIPTION
otus_dba1=#
```

###
Проверьте работу системы:
Выполните вставку в test на ВМ1 — убедитесь, что данные появились в test на ВМ2 и ВМ3
Выполните вставку в test2 на ВМ2 — убедитесь, что данные появились в test2 на ВМ1 и ВМ3
###
```sh
--Проверим состояние подписок:
--ВМ1:
otus_dba1=# SELECT * FROM pg_stat_subscription;
subid  |   subname    | worker_type | pid  | leader_pid | relid | received_lsn |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time        
--------+--------------+-------------+------+------------+-------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------
 131114 | sub_from_vm2 | apply       | 6564 |            |       | F/924ACA10   | 2026-07-19 15:44:04.790043+03 | 2026-07-19 15:44:04.790643+03 | F/924ACA10     | 2026-07-19 15:44:04.790043+03
(1 row)

otus_dba1=#

--ВМ2:
otus_dba1=# SELECT * FROM pg_stat_subscription;
subid  |   subname    | worker_type | pid  | leader_pid | relid | received_lsn |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time        
--------+--------------+-------------+------+------------+-------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------
 131115 | sub_from_vm1 | apply       | 6813 |            |       | F/924AAD28   | 2026-07-19 15:44:29.077558+03 | 2026-07-19 15:44:29.077697+03 | F/924AAD28     | 2026-07-19 15:44:29.077558+03
(1 row)

--ВМ3:
otus_dba1=# SELECT * FROM pg_stat_subscription;
subid  |      subname       | worker_type | pid  | leader_pid | relid | received_lsn |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time        
--------+--------------------+-------------+------+------------+-------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------
 131112 | sub_test_from_vm1  | apply       | 6481 |            |       | F/924AA930   | 2026-07-19 15:44:18.164208+03 | 2026-07-19 15:44:18.163587+03 | F/924AA930     | 2026-07-19 15:44:18.164208+03
 131113 | sub_test2_from_vm2 | apply       | 6497 |            |       | F/924B6EC0   | 2026-07-19 15:44:18.142009+03 | 2026-07-19 15:44:18.141402+03 | F/924B6EC0     | 2026-07-19 15:44:18.142009+03
(2 rows)

--ВМ1:
otus_dba1=# INSERT INTO test (id, descr) VALUES (1, 'Row from VM1');
INSERT 0 1
otus_dba1=#

otus_dba1=# SELECT slot_name, active, restart_lsn FROM pg_replication_slots WHERE slot_name LIKE 'sub_%';
     slot_name     | active | restart_lsn 
-------------------+--------+-------------
 sub_from_vm1      | t      | F/924C2840
 sub_test_from_vm1 | t      | F/924C2840
(2 rows)

otus_dba1=#

--На ВМ2 и ВМ3 строка не появилась!
--Если смотреть в лог СУБД, то есть ошибка по привилегиям:
2026-07-19 15:55:45.625 MSK [7558] ERROR:  could not start initial contents copy for table "public.test": ERROR:  permission denied for table test
2026-07-19 15:55:45.627 MSK [6616] LOG:  background worker "logical replication tablesync worker" (PID 7558) exited with exit code 1
2026-07-19 15:55:46.663 MSK [7559] repl_user@otus_dba1 LOG:  logical decoding found consistent point at F/924DD778
2026-07-19 15:55:46.663 MSK [7559] repl_user@otus_dba1 DETAIL:  There are no running transactions.
2026-07-19 15:55:46.663 MSK [7559] repl_user@otus_dba1 STATEMENT:  CREATE_REPLICATION_SLOT "pg_131114_sync_131104_7626037940681749860" LOGICAL pgoutput (SNAPSHOT 'use')
2026-07-19 15:55:46.674 MSK [7559] repl_user@otus_dba1 ERROR:  permission denied for table test2
2026-07-19 15:55:46.674 MSK [7559] repl_user@otus_dba1 STATEMENT:  COPY public.test2 (id, ro_data) TO STDOUT
2026-07-19 15:55:50.751 MSK [7561] LOG:  logical replication table synchronization worker for subscription "sub_from_vm1", table "test" has started
2026-07-19 15:55:50.756 MSK [7560] repl_user@otus_dba1 LOG:  logical decoding found consistent point at F/924DD7B0
2026-07-19 15:55:50.756 MSK [7560] repl_user@otus_dba1 DETAIL:  There are no running transactions.
2026-07-19 15:55:50.756 MSK [7560] repl_user@otus_dba1 STATEMENT:  CREATE_REPLICATION_SLOT "pg_131113_sync_131104_7626037940681749860" LOGICAL pgoutput (SNAPSHOT 'use')
2026-07-19 15:55:50.765 MSK [7560] repl_user@otus_dba1 ERROR:  permission denied for table test2
2026-07-19 15:55:50.765 MSK [7560] repl_user@otus_dba1 STATEMENT:  COPY public.test2 (id, ro_data) TO STDOUT
2026-07-19 15:55:50.814 MSK [7561] ERROR:  could not start initial contents copy for table "public.test": ERROR:  permission denied for table test
2026-07-19 15:55:50.816 MSK [6616] LOG:  background worker "logical replication tablesync worker" (PID 7561) exited with exit code 1
2026-07-19 15:55:51.933 MSK [7562] repl_user@otus_dba1 LOG:  logical decoding found consistent point at F/924DD958
2026-07-19 15:55:51.933 MSK [7562] repl_user@otus_dba1 DETAIL:  There are no running transactions.
2026-07-19 15:55:51.933 MSK [7562] repl_user@otus_dba1 STATEMENT:  CREATE_REPLICATION_SLOT "pg_131114_sync_131104_7626037940681749860" LOGICAL pgoutput (SNAPSHOT 'use')
2026-07-19 15:55:51.941 MSK [7562] repl_user@otus_dba1 ERROR:  permission denied for table test2
2026-07-19 15:55:51.941 MSK [7562] repl_user@otus_dba1 STATEMENT:  COPY public.test2 (id, ro_data) TO STDOUT
2026-07-19 15:55:55.943 MSK [7563] LOG:  logical replication table synchronization worker for subscription "sub_from_vm1", table "test" has started
2026-07-19 15:55:56.001 MSK [7563] ERROR:  could not start initial contents copy for table "public.test": ERROR:  permission denied for table test


--Необходимо удалить подписки, выдать права и создать подписки заново:

```
