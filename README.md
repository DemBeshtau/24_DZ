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

   
