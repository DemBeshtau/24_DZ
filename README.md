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
