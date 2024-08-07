# Сетевые пакеты. VLAN. LACP.
1. В тестовой подсети Office1 появляются сервера с дополнительными интерфейсами и адресами в internal сети testLAN:
   - testClient1 - 10.10.10.254;
   - testClient2 - 10.10.10.254;
   - testServer1 - 10.10.10.1;
   - testServer2 - 10.10.10.1;
2. Развести сети посредством VLAN:
   - testClient1 <-> testServer1;
   - testClient2 <-> testServer2;
3. Между centralRouter и inetRouter пробросить 2 линка (общая internal сеть) и объединить их в бонд, проверить работу с отключением интерфейсов.
### Исходные данные ###
&ensp;&ensp;ПК на Linux c 8 ГБ ОЗУ или виртуальная машина (ВМ) с включенной Nested Virtualization.<br/>
&ensp;&ensp;Предварительно установленное и настроенное ПО:<br/>
&ensp;&ensp;&ensp;Hashicorp Vagrant (https://www.vagrantup.com/downloads);<br/>
&ensp;&ensp;&ensp;Oracle VirtualBox (https://www.virtualbox.org/wiki/Linux_Downloads).<br/>
&ensp;&ensp;&ensp;Ansible (https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).<br/>
&ensp;&ensp;Все действия проводились с использованием Vagrant 2.4.0, VirtualBox 7.0.18, Ansible 9.4.0, образов CentOS 7 версии 1804_2 и ubuntu/jammy64 версии 20240301.0.0. <br/>
### Ход решения ###
&ensp;&ensp;Топология сети:<br/>
![24_final](https://github.com/user-attachments/assets/360dfcad-4ecd-4440-b850-08eaae67919e)

#### 1. Настройка VLAN ####
- Для настройки VLAN на хостах под управлением операционной системы (OC) CentOS (testClient1, testServer1) создаётся конфигурационный файл /etc/sysconfig/network-scripts/ifcfg-vlan1 с соответствующими интерфейсами и IP-адресами:
```shell
VLAN=yes
TYPE=Vlan
PHYSDEV=eth1
VLAN_ID=1
VLAN_NAME_TYPE=DEV_PLUS_VID_NO_PAD
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=10.10.10.254
PREFIX=24
NAME=1
DEVICE=eth1.1
ONBOOT=yes
```
- Перезапуск сервиса NetworkManager:
```shell
systemctl restart NetworkManager
```
- Для настройки VLAN на хостах под управлением ОС Ubuntu (testClient2, testServer2) производится конфигурация netplan с указанием соответствующих интерфейсов и IP-адресов:
```shell
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8: {}
    vlans:
        vlan2:
          id: 2
          link: enp0s8
          dhcp4: no
          addresses: [10.10.10.254/24]
    version: 2
```
- Применение настроек netplan:
```shell
netplan apply
```
- Проверка работоспособности конфигурации, оценка видимости хостов в пределах своих VLAN:
```shell
[root@testClient1 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.230 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.374 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.353 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.341 ms
^C
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.230/0.324/0.374/0.058 ms

vagrant@testClient2:~$ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.224 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.285 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.321 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.247 ms
^C
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3066ms
rtt min/avg/max/mdev = 0.224/0.269/0.321/0.036 ms
```
#### 2. Настройка Bond ####
- Создание bond-интерфейса на серверах inetRouter и centralRouter путём конфигурирования файла /etc/sysconfig/network-scripts/ifcfg-bond0 (указываются соответствующие IP-адреса):
```shell
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.255.1
NETMASK=255.255.255.252
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
NM_CONTROLED=yes
```
- Добавление интерфейсов eth1 и eth2 в bond-интерфейс, путём конфигурирования файлов ifcfg-eth1 и ifcfg-eth2 в директории /etc/sysconfig/network-scripts/ (выполняется на обоих серверах):
```shell
DEVICE=eth1
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=yes
USERCTL=no
```
- Применение настроек:
```shell
systemctl restart NetworkManager
```
- Для того, чтобы проверить работоспособность конфигурации, необходимо запустить пинг с сервера inetRouter в направлении centralRouter, после чего отключить один физический интерфейс на centralRouter:  
```shell
[root@centralRouter ~]# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
eth0             UP             52:54:00:c9:c7:04 <BROADCAST,MULTICAST,UP,LOWER_UP> 
bond0            UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> 
eth1             UP             08:00:27:97:b2:63 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth2             UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth3             UP             08:00:27:62:11:57 <BROADCAST,MULTICAST,UP,LOWER_UP> 
eth4             UP             08:00:27:03:f4:d5 <BROADCAST,MULTICAST,UP,LOWER_UP> 

[root@inetRouter ~]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.010 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 192.168.255.1: icmp_seq=3 ttl=64 time=0.026 ms
64 bytes from 192.168.255.1: icmp_seq=4 ttl=64 time=0.027 ms
64 bytes from 192.168.255.1: icmp_seq=5 ttl=64 time=0.027 ms
64 bytes from 192.168.255.1: icmp_seq=6 ttl=64 time=0.030 ms

[root@centralRouter ~]# ip link set down eth1
[root@centralRouter ~]# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
eth0             UP             52:54:00:c9:c7:04 <BROADCAST,MULTICAST,UP,LOWER_UP> 
bond0            UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> 
eth1             DOWN           08:00:27:97:b2:63 <BROADCAST,MULTICAST,SLAVE> 
eth2             UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth3             UP             08:00:27:62:11:57 <BROADCAST,MULTICAST,UP,LOWER_UP> 
eth4             UP             08:00:27:03:f4:d5 <BROADCAST,MULTICAST,UP,LOWER_UP> 

...
64 bytes from 192.168.255.1: icmp_seq=60 ttl=64 time=0.040 ms
64 bytes from 192.168.255.1: icmp_seq=61 ttl=64 time=0.035 ms
64 bytes from 192.168.255.1: icmp_seq=62 ttl=64 time=0.038 ms
64 bytes from 192.168.255.1: icmp_seq=63 ttl=64 time=0.036 ms
64 bytes from 192.168.255.1: icmp_seq=64 ttl=64 time=0.024 ms
64 bytes from 192.168.255.1: icmp_seq=65 ttl=64 time=0.024 ms
64 bytes from 192.168.255.1: icmp_seq=66 ttl=64 time=0.024 ms
...

[root@centralRouter ~]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth2
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:db:b1:00
Slave queue ID: 0

Slave Interface: eth1
MII Status: down
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 08:00:27:97:b2:63
Slave queue ID: 0
```
&ensp;&ensp;Для автоматического конфигурирования инфраструктуры с помощью Ansible, подготовлен плейбук playbook.yml.
