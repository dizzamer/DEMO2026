# Модуль № 1:Настройка сетевой инфраструктуры  

## 1. Произведите базовую настройку устройств  
 ### ● Настройте имена устройств согласно топологии. Используйте полное доменное имя
    HQ-RTR | BR-RTR:  
    conf t
    hostname {hq-rtr.au-team.irpo, br-rtr.au-team.irpo}    
    HQ-SRV | HQ-CLI | BR-SRV:  
    hostnamectl set-hostname {hq-srv, hq-cli, br-srv}.au-team.irpo; exec bash  
 ### ● На всех устройствах необходимо сконфигурировать IPv4  
    Настройка адресов производится через nmtui  
 ### ● IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918   
    RFC 1918 включает себя следующие адреса:  
    10.0.0.0/8  
    172.16.0.0/12  
    192.168.0.0/16  
 ### ● Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 32 адресов  
    маска /27 255.255.255.224 - 32 адреса (30 используемых)
    192.168.0.0/27 192.168.0.1 – 192.168.0.30	192.168.0.31 - Broadcast  
 ### ● Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не менее 16 адресов  
    маска /27 255.255.255.240 - 32 адреса (30 используемых)
    192.168.0.32/28	192.168.0.33 – 192.168.0.62	192.168.0.63 - Broadcast  
 ### ● Локальная сеть в сторону BR-SRV должна вмещать не более 8 адресов  
    маска /29 255.255.255.248 - 8 адресов (6 используемых)
    192.168.0.64/29	192.168.0.65 – 192.168.0.70	192.168.0.71 - Broadcast  
    Сразу назанчим адрес в сторону сервера
    Настройка производится на BR-RTR:
    en   
    conf t   
    port te1   
    service-instance toSRV   
    encapsulation untagged   
    end   
    wr mem      
    conf t   
    int SRV   
    ip add 192.168.0.65/29   
    connect port te1 service-instance toSRV   
    end   
    wr mem   
 ### ● Локальная сеть для управления(VLAN999) должна вмещать не более 16 адресов  
    маска /28 255.255.255.240 - 16 адресов (14 используемых)
    192.168.0.72/28	192.168.0.73 – 192.168.0.86	192.168.0.87 - Broadcast  
### ● Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 2, в качестве примера используйте Прил_3_О1_КОД 09.02.06-1-2026-М1
   ![Прил_3_О1_КОД 09.02.06-1-2026-М1](https://github.com/dizzamer/DEMO2026-Profile/blob/main/%D0%9F%D1%80%D0%B8%D0%BB_3_%D0%9E%D0%97_%D0%9A%D0%9E%D0%94%2009.02.06-1-2026-%D0%9C1.docx)
    
 | Имя Устройства   | IPv4                     |  Интерфейс  | NIC       | Шлюз         | 
 | ---------        | ---------                | ---------   | --------- | ---------    |
 | ISP              | NAT (inet)               | ens3        | Internet  |              |
 |                  | 172.16.1.14/28           | ens4        | ISP_HQ    |              |
 |                  | 172.16.2.14/28           | ens5        | ISP_BR    |              |
 | HQ-RTR           | 172.16.4.1/28            | te0         | ISP_HQ    | 172.16.1.14  |
 |                  | 192.168.0.73/28          | te1.999     | HQ_NET    |              |
 |                  | 192.168.0.1/27           | te1.100     | -         |              |
 |                  | 192.168.0.33/28          | te1.200     | -         |              |
 |                  | 172.16.0.1/30            | GRE         | TUN       |              |
 | HQ-SW            | 192.168.0.74/28          | ens3        | HQ_NET    |              |
 |                  | -                        | ens4        | SRV_NET   |              |
 |                  | -                        | ens5        | CLI_NET   |              |
 | HQ-SRV           | 192.168.0.2/27           | ens3        | SRV_NET   | 192.168.0.1  |
 | HQ-CLI           | 192.168.0.34/28(DHCP)    | ens3        | CLI_NET   | 192.168.0.33 |
 | BR-RTR           | 172.16.2.1/28            | te0         | ISP_BR    | 172.16.2.14  |
 |                  | 192.168.0.65/29          | te1         | BR_NET    |              |
 |                  | 172.16.0.2/30            | GRE         | TUN       |              |
 | BR-SRV           | 192.168.0.66/29          | ens3        | BR_NET    | 192.168.0.65 |

## 2. Настройте доступ к сети Интернет, на маршрутизаторе ISP:  
 ### ● Настройте адресацию на интерфейсах:  
    Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP  
    o Настройте маршруты по умолчанию там, где это необходимо   
    Маршруты по умолчанию настраиваются на роутерах:  
    HQ-RTR - ip route 0.0.0.0/0 172.16.1.14  
    BR-RTR - ip route 0.0.0.0/0 172.16.2.14  
    o Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.1.0/28  
    Настройка производится на HQ-RTR(EcoRouter):  
    en  
    conf t  
    port te0  
    service-instance toISP  
    encapsulation untagged  
    end  
    wr mem  
    en  
    conf t  
    int ISP  
    ip add 172.16.1.1/28  
    connect port te0 service-instance toISP  
    end  
    wr mem  
    o Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.2.0/28  
    Настройка производится на BR-RTR(EcoRouter):  
    en  
    conf t  
    port te0  
    service-instance toISP  
    encapsulation untagged  
    end  
    wr mem  
    en  
    conf t  
    int ISP  
    ip add 172.16.2.1/28  
    connect port te0 service-instance toISP  
    end  
    wr  mem  
    o На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR  
    для доступа к сети Интернет:  
    echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
    dnf install iptables-services –y   
    systemctl enable ––now iptables  
    iptables –t nat –A POSTROUTING –s 172.16.1.0/28 –o ens3 –j MASQUERADE  
    iptables –t nat –A POSTROUTING –s 172.16.2.0/28 –o ens3 –j MASQUERADE  
    iptables-save > /etc/sysconfig/iptables  
    systemctl restart iptables  
    nano /etc/sysconfig/iptables - не должно быть ничего лишнего.  
    в случае если там есть то, что вы не добавляли - удалить
    iptables –L –t nat - должны высветится в Chain POSTROUTING две настроенные подсети.  
## 3. Создайте локальные учетные записи на серверах HQ-SRV и BR-SRV  
 ### ● Создание пользователя sshuser на серверах HQ-SRV | BR-SRV:    
    useradd -m -u 2026 sshuser  
    o Пароль пользователя sshuser с паролем P@ssw0rd  
    echo "sshuser:P@ssw0rd" | sudo chpasswd  
    o Идентификатор пользователя 2026  
    o Пользователь sshuser должен иметь возможность запускать sudo
    без дополнительной аутентификации.  
    usermod -aG wheel sshuser  
   ### ● Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BR-RTR  
    Настройка производится на EcoRouter HQ-RTR | BR-RTR:  
    conf t  
    username net_admin  
    o Пароль пользователя net_admin с паролем P@ssw0rd  
    password P@ssw0rd   
    o При настройке на EcoRouter пользователь net_admin должен обладать максимальными привилегиями  
    role admin  
    o При настройке ОС на базе Linux, запускать sudo без дополнительной аутентификации  
## 4. Настройте коммутацию в сегменте HQ следующим образом:  
 ### ● Предусмотреть возможность передачи трафика управления в VLAN 999    
    Настройка на HQ-RTR:  
    port te1  
    Service-instance toSW  
    Encapsulation dot1q 999  
    rewrite pop 1  
    end  
    wr mem  
    en  
    conf t  
    Int te1.999  
    ip add 192.168.0.73/28   
    description toSW   
    connect port te1 service-instance toSW   
    end  
    wr mem  
    Настройка на HQ-SW:  
    Перед настройкой линк ens3 в nmtui должен быть в состоянии - отключено
    Адресации так же не должно быть  
    ovs-vsctl add-br hq-sw   
    ovs-vsctl add-port hq-sw ens3 tag=999 trunks=999,100,200   
    ifconfig ovs0-vlan999 inet 192.168.0.74/28 up   
 ### ● Трафик HQ-SRV должен принадлежать VLAN 100    
    Настройка на HQ-RTR:  
    conf t  
    port te1   
    service-instance te1.100   
    encapsulation dot1q 100   
    rewrite pop 1   
    end   
    wr mem   
    int te1.100  
    ip add 192.168.0.1/27  
    connect port te1 service-instance te1.100  
    end  
    wr mem  
    Настройка на HQ-SW:  
    Адресации не должно быть
    Так как при настройке на HQ-SW бридж hq-sw уже создан, его создавать не нужно
    ovs-vsctl add-port hq-sw ens4 tag=100  
 ### ● Трафик HQ-CLI должен принадлежать VLAN 200    
    Настройка на HQ-RTR:  
    conf t  
    port te1  
    service-instance te1.200  
    encapsulation dot1q 200  
    rewrite pop 1  
    end  
    wr mem  
    int te1.200  
    ip add 192.168.0.33/28  
    connect port te1 service-instance te1.200 
    end  
    wr mem  
    Настройка на HQ-SW: 
    Адресации не должно быть
    Так как при настройке на HQ-SW бридж hq-sw уже создан, его создавать не нужно
    ovs-vsctl add-port hq-sw ens5 tag=200 
**● Сведения о настройке коммутации внесите в отчёт**  
## 5. Настройте безопасный удаленный доступ на серверах HQ-SRV и BR-SRV:  
 ### ● Для подключения используйте порт 2026  
    Перед настройкой выполните команду setenforce 0, далее проверяем командой: getenforce   
    должно быть состояние Permissive  
    dnf install openssh - если не установлен
    systemctl enable --now sshd
    echo Port 2026 >> /etc/ssh/sshd_config
  ### ● Разрешите подключения только пользователю sshuser  
    echo AllowUsers sshuser >> nano /etc/ssh/sshd_config
 ### ● Ограничьте количество попыток входа до двух  
    echo MaxAuthTries 2 >> /etc/ssh/sshd_config
 ### ● Настройте баннер «Authorized access only»  
    echo «Authorized access only» > /etc/ssh/sshd_banner
    echo Banner /etc/ssh/sshd_banner >> /etc/ssh/sshd_config
    systemctl restart sshd
## 6. Между офисами HQ и BR, на маршрутизаторах HQ-RTR и BR-RTR необходимо сконфигурировать ip туннель: 
  ### o На выбор технологии GRE или IP in IP - используем GRE
    Настройка на HQ-RTR:  
    Interface tunnel.1   
    Ip add 172.16.0.1/30   
    Ip mtu 1476   
    ip ospf network broadcast   
    ip ospf mtu-ignore   
    Ip tunnel 172.16.1.1 172.16.2.1 mode gre   
    end    
    wr mem    
    Настройка на BR-RTR:  
    Interface tunnel.1  
    Ip add 172.16.0.2/30  
    Ip mtu 1476  
    ip ospf mtu-ignore  
    ip ospf network broadcast  
    Ip tunnel 172.16.2.1 172.16.1.1 mode gre  
    end    
    ПРОВЕРЯЕМ ТУННЕЛЬ ПИНГОМ ОТ РОТУЕРА К РОУТЕРУ, БЕЗ ЭТОГО НЕ ПЕРЕХОДИМ К НАСТРОЙКЕ OSPF!!!
    Проверка на HQ-RTR: 
    ping 172.16.0.2  
    Проверка на BR-RTR:  
    ping 172.16.0.1
    Должно быть успешно с 2-х сторон!!!
  o Сведения о туннеле занесите в отчёт
## 7. Обеспечьте динамическую маршрутизацию на маршрутизаторах HQ-RTR и BR-RTR: сети одного офиса должны быть доступны из другого офиса и наоборот. Для обеспечения динамической маршрутизации используйте link state протокол на усмотрение участника: Будем использовать OSPF      
  ### ● Разрешите выбранный протокол только на интерфейсах в ip туннеле   
  ### ● Маршрутизаторы должны делиться маршрутами только друг с другом:   
    Настройка на HQ-RTR:  
    Conf t
    Router ospf 1
    Ospf router-id  172.16.0.1
    network 172.16.0.0 0.0.0.3 area 0
    network 192.168.0.0 0.0.0.31 area 0
    network 192.168.0.32 0.0.0.15 area 0
    passive-interface default
    no passive-interface tunnel.1
    Настройка на BR-RTR:  
    Conf t
    Router ospf 1
    Ospf router-id 172.16.0.2
    Network 172.16.0.0 0.0.0.3 area 0
    Network 192.168.0.64 0.0.0.7 area 0
    Passive-interface default
    no passive-interface tunnel.1  
  ### ● Обеспечьте защиту выбранного протокола посредством парольной защиты  
    Настройка производится на EcoRouter HQ-RTR:  
    router ospf 1  
    area 0 authentication  
    ex  
    interface tunnel.1   
    ip ospf authentication-key ecorouter  
    wr mem   
    Настройка производится на EcoRouter BR-RTR:  
    router ospf 1   
    area 0 authentication  
    ex   
    interface tunnel.1   
    ip ospf authentication-key ecorouter   
    wr mem   
  ● Сведения о настройке и защите протокола занесите в отчёт  
## 8. Настройка динамической трансляции адресов маршрутизаторах HQ-RTR и BR-RTR  
 ### ● Настройте динамическую трансляцию адресов для обоих офисов в сторону ISP.  
    Настройка производится на EcoRouter HQ-RTR: 
    ip nat pool nat1 192.168.0.0-192.168.0.31  
    ip nat source dynamic inside-to-outside pool nat1 overload interface ISP 
    ip nat pool nat2 192.168.0.32-192.168.0.63  
    ip nat source dynamic inside-to-outside pool nat2 overload interface ISP   
    Настройка производится на EcoRouter BR-RTR: 
    ip nat pool nat3 192.168.0.64-192.168.0.71  
    ip nat source dynamic inside-to-outside pool nat3 overload interface ISP 
### ● Все устройства в офисах должны иметь доступ к сети Интернет  
    Настройка производится на EcoRouter HQ-RTR:
    en
    conf t
    int ISP
    ip nat outside
    ex
    int te1.999
    ip nat inside
    ex
    int te1.100
    ip nat inside
    ex
    int te1.200
    ip nat inside
    Настройка производится на EcoRouter BR-RTR: 
    en
    conf t
    int ISP
    ip nat outside
    ex
    int SRV
    ip nat inside
    ex  
    Настройка производится на HQ-SRV:
    В nmtui прописывеем шлюз - 192.168.0.1  
    Настройка производится на BR-SRV:  
    В nmtui прописывет шлюз - 192.168.0.65  
## 9. Настройте протокол динамической конфигурации хостов для сети в сторону HQ-CLI:    
   ● Настройте нужную подсеть  
  ### ● Для офиса HQ в качестве сервера DHCP выступает маршрутизатор HQ-RTR    
    Настройка производится на EcoRouter HQ-RTR:
    ip pool dhcpHQ 192.168.0.34-192.168.0.62
    en
    conf t
    dhcp-server 1
    pool dhcpHQ 1
    domain-name au-team.irpo
    mask 255.255.255.240  
    dns 192.168.0.2    
    gateway 192.168.0.33    
    end  
    wr mem  
  ###  ● Клиентом является машина HQ-CLI.  
    interface te1.200
    dhcp-server 1
  ● Исключите из выдачи адрес маршрутизатора  
  ● Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR.  
  ● Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV.  
  ● DNS-суффикс для офисов HQ – au-team.irpo  
  ● Сведения о настройке протокола занесите в отчёт  
## 10. Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR:    
  ● Основной DNS-сервер реализован на HQ-SRV    
    dnf install bind -y  
    systemctl enable --now named  
    cp /etc/named.conf /etc/named.conf.backup - делаем бэкап файла  
    nano /etc/named.conf  
    ![named первая часть](https://github.com/dizzamer/DEMO2025/blob/main/dns.png)  
    ![named вторая часть](https://github.com/dizzamer/DEMO2025/blob/main/dns2.png)  
    mkdir /var/named/master  
    nano /var/named/master/au-team  
    ![au team irpo зона](https://github.com/dizzamer/DEMO2025/blob/main/au-teamn.png)  
    nano /var/named/master/168.192.zone    
    ![au team irpo зона](https://github.com/dizzamer/DEMO2025/blob/main/0.168.192zone.png)  
    chown -R root:named /var/named/master/
    chown -R named:named /var/named
    chown -R root:named /etc/named.conf  
    chmod 750 /var/named/  
    chmod 750 /var/named/master/  
    systemctl restart named  
    Проверить зоны можно командой named-checkconf -z  
     ![au team irpo зона](https://github.com/dizzamer/DEMO2025/blob/main/checkconf.png)  
     Для полной работоспособности на HQ-CLI нужно установить в качестве dns севрера HQ-SRV:  
     nano /etc/resolv.conf на всех устройствах должен иметь следюущий вид:  
     ![resolvconf](https://github.com/dizzamer/DEMO2025/blob/main/resolv.conf.png)  
     resolvectl dns ens3 192.168.0.2  
     Для полной работоспособности на HQ-RTR нужно установить в качестве dns севрера HQ-SRV:  
     ip name-server 192.168.0.2  
     На остальных устройствах делаем подобным образом.  
  ● Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с таблицей 3  
### Таблица 3. Таблица имен  
   | Устройство | Запись              | Тип    | 
   | ---------  |  ------             | ----   |
   | HQ-RTR     | hq-rtr.au-team.irpo | A,PTR  | 
   | BR-RTR     | br-rtr.au-team.irpo | A      |
   | HQ-SRV     | hq-srv.au-team.irpo | A,PTR  |
   | HQ-CLI     | hq-cli.au-team.irpo | A,PTR  |
   | BR-SRV     | br-srv.au-team.irpo | A      |
   | ISP (интерфейс направленный в сторону HQ-RTR)     | docker.au-team.irpo | A      | 
   | ISP (интерфейс направленный в сторону BR-RTR)     | web.au-team.irpo    | A      |  
      
  ● В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер  
## 11. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена.  
   ### Настройка проивзодится на всех устройствах:  
    timedatectl set-timezone Europe/Moscow  
## Необходимые приложения для модуля 1:  
   Прил_1_О1_КОД 09.02.06-1-2026-М1: Шаблон отчета  
   Прил_3_О1_КОД 09.02.06-1-2026-М1: Пример заполнения таблицы адресов  
   Прил_4_О1_КОД 09.02.06-1-2026-М1: Инструкции по оформлению отчёта  

# Модуль № 2:Организация сетевого администрирования операционных систем  

## 1.	Настройте доменный контроллер Samba на машине BR-SRV.   
  ### Настройка проивзодится на BR-SRV:  
   ## Предварительная настройка сервера:  
     выставляем 192.168.0.2 в качестве нащего днс сервера на линке в nmtui и домен поиска au-team.irpo  
     setenforce 0  
     nano /etc/selinux  
     Замените в файле конфигурации /etc/selinux/config режим enforcing на permissive   
     dnf install samba* krb5* -y  
     Проверьте доступные серверы имен, просмотрев файл resolv.conf:  
     cat /etc/resolv.conf  
     В выводе должно отобразиться наш dns сервер и домен для поиска.  
   ## Создание домена под управлением контроллера домена Samba DC:  
     Создание резервных копий файлов  
     Переименуйте файл /etc/smb.conf, он будет создан позднее в процессе выполнения команды samba-tool.  
     cp /etc/samba/smb.conf /etc/samba/smb.conf.back  
     Создайте резервную копию используемого по умолчанию конфигурационного файла kerberos:  
     cp /etc/krb5.conf /etc/krb5.conf.back  
   ## Конфигурирование сервера с помощью утилиты samba-tool  
     Файла /etc/samba/smb.conf быть не должно, он сам создаст.  
     rm -rf /etc/samba/smb.conf  
     samba-tool domain provision --use-rfc2307 --interactive  
   ![sambatool](https://github.com/dizzamer/DEMO2025/blob/main/samba-toolprovision.png) 
   ## Удаление использования службы dns  
     systemctl stop samba  
     Подчищаем, все где могут храниться наши записи  
     sudo rm -rf /var/lib/samba/private/dns_update_cache  
     sudo rm -rf /var/lib/samba/private/dns_update_list  
     sudo rm -rf /var/lib/samba/private/dns  
     sudo rm -rf /var/lib/samba/private/dns.keytab  
     Добавляем в файл /etc/samba/smb.conf следующее  
   ![smbconf](https://github.com/dizzamer/DEMO2025/blob/main/smbconf.png)  
   
      Запустите и добавьте в автозагрузку службы samba и named:  
      systemctl enable named samba --now  
      systemctl status named samba  
      Проверка созданного домена с помощью команды samba-tool domain info au-team.irpo:  
   ![sambatool](https://github.com/dizzamer/DEMO2025/blob/main/samba-tool.png)  
   ## •	Создайте 5 пользователей для офиса HQ: имена пользователей формата user№.hq. Создайте группу hq, введите в эту группу созданных пользователей  
     sudo samba-tool group add hq  
    for i in {1..5}; do  
    sudo samba-tool user create user$i.hq P@ssw0rd$i  
    sudo samba-tool group addmembers hq user$i.hq  
    done    
   ## •	Введите в домен машину HQ-CLI  
     Перед вводом необходимо ввести на HQ-SRV:  
     smbclient -L localhost -U administrator  
     Проверка kerberos: 
     kinit administrator@localhost
### Настройка проивзодится на HQ-CLI:  
  https://redos.red-soft.ru/base/redos-7_3/7_3-administation/7_3-domain-redos/7_3-domain-config/7_3-redos-in-samba/?nocache=1730793368537  
•	Пользователи группы hq имеют право аутентифицироваться на клиентском ПК  
•	Пользователи группы hq должны иметь возможность повышать привилегии для выполнения ограниченного набора команд: cat, grep, id. Запускать другие команды с повышенными привилегиями пользователи группы не имеют права  
•	Выполните импорт пользователей из файла users.csv. Файл будет располагаться на виртуальной машине BR-SRV в папке /opt  
## 2.	Сконфигурируйте файловое хранилище:  
 ### Настройка проивзодится на HQ-SRV:
  Перед тем как начать проверяем, что установлены следюущие пакеты
  dnf install mdadm nfs-utils -y
## •	При помощи трёх дополнительных дисков, размером 1Гб каждый, на HQ-SRV сконфигурируйте дисковый массив уровня 5  
    mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd     
 ![mdadmcreate](https://github.com/dizzamer/DEMO2025/blob/main/mdadm_create.png)    
 ![mdaddetail](https://github.com/dizzamer/DEMO2025/blob/main/mdadm_detail.png)  
## •	Имя устройства – md0, конфигурация массива размещается в файле /etc/mdadm.conf  
    mdadm --detail --scan >> /etc/mdadm.conf      
 ## •	Обеспечьте автоматическое монтирование в папку /raid5  
    Добавляем в /etc/fstab:    
    nano /etc/fstab  
    /dev/md0 /raid5 ext4 defaults 0 0  
![fstab](https://github.com/dizzamer/DEMO2025/blob/main/fstab.png)  
 ## •	Создайте раздел, отформатируйте раздел, в качестве файловой системы используйте ext4  
     mkfs.ext4 /dev/md0  
   ![mkfs](https://github.com/dizzamer/DEMO2025/blob/main/mkfs.png)   
 ## •	Создаем точку монтирования и примонтируемся     
    mkdir -p /raid   
    mount -a   
  ![mount](https://github.com/dizzamer/DEMO2025/blob/main/mount.png)   
 ## 3. Настройте сервер сетевой файловой системы(nfs) на HQ-SRV
 ### Настройка проивзодится на HQ-SRV:
  ## • в качестве папки общего доступа выберите /raid/nfs, доступ для чтения и записи для всей сети в сторону HQ-CLI   
  ## •	Создаем папку для NFS  
    mkdir -p /raid/nfs  
    chmod 777 /raid/nfs  
 ![mkdir_nfs](https://github.com/dizzamer/DEMO2025/blob/main/mkdir_nfs.png)  
 ## Настройка экспорта  
    Добавляем в /etc/exports:  
    nano /etc/exports  
    /raid5/nfs 192.168.0.32/28(rw,sync,insecure,nohide,all_squash,no_subtree_check)
![exports](https://github.com/dizzamer/DEMO2025/blob/main/etcexports.png)  
  ## Применяем изменения и перезагружаем службу
    exportfs -rav  
    systemctl restart nfs-server  
 ### Настройка проивзодится на HQ-CLI:
  ## •	На HQ-CLI настройте автомонтирование в папку /mnt/nfs  
      Добавляем в /etc/fstab:    
      nano /etc/fstab  
      hq-srv:/raid/nfs /mnt/nfs nfs defaults 0 0
  ![fstab_hqcli](https://github.com/dizzamer/DEMO2025/blob/main/fstab_hqcli.png)  
  ## Создаем точку монтирования и примонтируемся  
    mkdir -p /mnt/nfs  
    mount -a 
  ![mountdir_hqcli](https://github.com/dizzamer/DEMO2025/blob/main/nount_cli.png)  
  ## Проверка монтирования
      После этого при создании файла на клиенте, он должен появляться и на сервере
   •	Основные параметры сервера отметьте в отчёте  
  ## 4.	Настройте службу сетевого времени на базе сервиса chrony на маршрутизаторе ISP  
  ### Настройка проивзодится на ISP:  
  Вышестоящий сервер ntp на маршрутизаторе ISP - на выбор участника
  Стратум сервера - 5
      en  
      conf t  
      ntp server 172.16.4.1   
      ntp timezone UTC+3  
      end  
      wr mem  
  •	Стратум сервера - 5 
  •	В качестве клиентов ntp настройте: HQ-SRV, HQ-CLI, BR-RTR, BR-SRV.   
  
## 5.	Сконфигурируйте ansible на сервере BR-SRV  
  ### Настройка подключения по ssh BR-RTR | HQ-RTR
     Настройка производится на HQ-RTR:  
     en  
     conf  
     security-profile 1  
     rule 1 permit tcp any eq 22 any  
     end  
     wr mem  
     configure
     ip vrf vrf0  
     transport input ssh    
     security 1 vrf vrf0  
     end  
     wr mem  
     conf
     no security default  
     Настройка производится на BR-RTR: 
     conf  
     security-profile 1  
     rule 1 permit tcp any eq 22 any  
     end  
     wr mem  
     configure
     ip vrf vrf0  
     transport input ssh    
     security 1 vrf vrf0  
     end  
     wr mem  
     conf
     no security default
  ## •	Сформируйте файл инвентаря, в инвентарь должны входить HQ-SRV, HQ-CLI, HQ-RTR и BR-RTR  
   ### Настройка производится на BR-SRV:  
    • Рабочий каталог ansible должен располагаться в /etc/ansible  
      dnf install ansible -y  
      1) В файле можно прописывать как ip адреса так и имена хостов, сделаем следующим образом.  
      Так как у нас порт для покдлючения к серверам и клиентам 2024, укажим необходимые переменные для подключения  
      Для роутера так же указываем переменные, для подключения к роутерам будем использовать пользователя net_admin
      nano /etc/ansible/inventory.ini  
      [clients]
      hq-cli ansible_host=192.168.1.65
        
      [servers]
      hq-srv ansible_host=192.168.0.2
         
      [routers]
      hq-rtr ansible_host=192.168.0.62
      br-rtr ansible_host=172.16.5.1
         
      [clients:vars]
      ansible_port=2024
      ansible_user=sshuser
         
      [servers:vars]
      ansible_port=2024
      ansible_user=sshuser
 
      [routers:vars]
      ansible_user=net_admin
      ansible_password=P@$$word
   ![inventory](https://github.com/dizzamer/DEMO2025/blob/main/inventoryini.png) 
   ###  Настройка подключения по ключам на BR-SRV
    2) Подключение к хостам осуществляется по протоколу ssh с помощью rsa ключей.  
    Для начала переходим в ранее созданно пользователя sshuser командой:
    su sshuser
    Сгенерировать серверный ключ можно командой ниже. При её выполнении везде нажмите Enter.
    ssh-keygen
    3) Далее нужно распространить ключ на все подключенные хосты.  
    Распространить ключи на хосты можно командой:  
    На роутеры ключи пробрасывать не нужно, для них подключение по паролю!
    ssh-copy-id sshuser@{hq-srv, hq-cli} 
    где:  
    sshuser - это пользователь, от имени которого будут выполняться плейбуки;  
    server - IP-адрес хоста.  
  ### •	Все указанные машины должны без предупреждений и ошибок отвечать pong на команду ping в ansible посланную с BR-SRV  
    Пингуем удаленные хосты с помощью Ansible находясь в пользователе sshuser:  
    ansible -i /etc/ansible/inventory.ini all -m ping 
    В результате под каждым хостом должно быть написано "ping": "pong".  
![inventory](https://github.com/dizzamer/DEMO2025/blob/main/ansubleping.png)   

## 6.	Развертывание приложений в Docker на сервере BR-SRV. 
    Установка необходимых пакетов:  
    dnf install docker-ce docker-ce-cli docker-compose -y  
    systemctl enable docker --now
    Добавляем текущего пользователя в группу докер, текущий пользователь - student   
    usermod -aG docker $USER  
### •	Создайте в домашней директории пользователя файл wiki.yml для приложения MediaWiki.  
     • Средствами docker должен создаваться стек контейнеров с веб
     приложением и базой данных
     • Используйте образы site_latestи mariadb_latestрасполагающиеся в
     директории docker в образе Additional.iso
     • Основной контейнер testapp должен называться tespapp
     • Контейнер с базой данных должен называться db
     • Импортируйте образы в docker, укажите в yaml файле параметры
     подключения к СУБД, имя БД - testdb, пользователь testс паролем
     P@ssw0rd, порт приложения 8080, при необходимости другие
     параметры
     • Приложение должно быть доступно для внешних подключений через
     порт 8080
      Поднимаем стек контейнеров с помощью команды: 
      docker compose -f wiki.yml up -d  
 ![wikiyml](https://github.com/dizzamer/DEMO2025/blob/main/wikiyml.png)  
### Настройка mediawiki после успешного поднятия контейнеров  
  Переходим по доменному имени или адреса нашего сервера, должны увидеть такую картину:  
 ![wikistart](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki.png)  
  Далее настройка выглядит следуюшим образом:  
  На скриншоте ниже здесь этап проверки всего необходимого для работы MediaWiki, проверка должна пройти успешно  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki2.png)  
  Далее необходимо указать в хосте базы данных то, как у вас называется контнейнер на сервере, у меня mariadb  
  Проверить можно зайдя на сервер и выполнить команду docker ps:  
  ![hostwiki](https://github.com/dizzamer/DEMO2025/blob/main/hostwiki.png)  
  Указываем, все как по заданию и жмем далее:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki3.png)  
  Далее будет такое у вас как снизу, жмем далее:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki4.png)  
  Настройка дальше на скриншоте снизу:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki5.png)  
  На след скриншоте жмем далее:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki6.png)  
  Настройка базы данных должна быть выполнена успешно, как на скриншоте ниже:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki77.png)  
  Далее у вас должен скачаться файл LocalSettings.php автомтаически:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/mediawiki8.png)  
  Потом переходим в директории загрузки:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/localsettings.png)  
  Открывем директорию через терминал:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/localsettingsterm.png)  
  Перекидываем его через scp следующим образом:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/localsettingsterm2.png)  
  Переходим на сервер и убеждаем, что файл находится рядом с нашим wiki.yml:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/localsettingsterm3.png)  
  Далее раскоменчиваем строку в файле wiki.yml:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/localsettingswikiyml.png)   
  Запускаем стек контейнеров командой docker-compose -f wiki.yml up -d    
  Далее переходим по доменному имени или адреса нашего сервера и наблюдаем вот это:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/wikidemo.png)  
  Для проверки того, что все получилось входим под админской учеткой Wiki:WikiP@ssw0rd:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/wikidemologin.png)  
  Должно получиться вот так:  
  ![wikinext](https://github.com/dizzamer/DEMO2025/blob/main/wikiuser.png)  
  
## 7.	Разверните веб приложениена сервере HQ-SRV:   
### Подготовка   
 Переводим selinux в состояние Permissive:  
 setenforce 0    
 Проверяем:  
 getenforce  
 dnf install -y git httpd mariadb-server php php-cli php-common php-fpm php-gd php-intl php-json php-mbstring php-mysqlnd php-opcache php-pdo php-xml php-xmlrpc php-pecl-zip php-soap   
## •	Используйте веб-сервер apache  
    systemctl enable --now httpd  
    Создаем конфигурационный файл /etc/httpd/conf.d/moodle.conf:  
    nano /etc/httpd/conf.d/moodle.conf  
    <VirtualHost *:80>  
        DocumentRoot "/var/www/html/moodle"  
        ServerName HQ-SRV  
        <Directory "/var/www/html/moodle">  
            AllowOverride All  
            Require all granted  
        </Directory>  
        ErrorLog "/var/log/httpd/moodle_error.log"  
        CustomLog "/var/log/httpd/moodle_access.log" combined  
    </VirtualHost>  
     Перезапускаем Apache:  
     systemctl restart httpd   
## •	В качестве системы управления базами данных используйте mariadb   
     systemctl enable --now mariadb  
     mysql_secure_installation  
## •	Файлы веб приложения и дамп базы данных находятся в директории web образа Additional.iso  
     mysql -u root -p   
     CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;   
## •	Выполните импорт схемы и данных из файла dump.sql в базу данных webdb  

## •	Создайте пользователя webс паролем P@ssw0rd и предоставьте ему права доступа к этой базе данных   
     CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssw0rd';   
     GRANT ALL PRIVILEGES ON moodledb.* TO 'moodle'@'localhost';   
     FLUSH PRIVILEGES;   
     EXIT;   
     
     Создаем директории для нашего moodle  
     mkdir /opt/moodle  
     mkdir /usr/moodle_data  
     Далее переходим в директорию и клонируем  
     cd /opt/moodle  
     git clone git://git.moodle.org/moodle.git  
     cd /opt/moodle/moodle  
     cp config-dist.php config.php 
## •	Файлы index.php и директорию images скопируйте в каталог веб сервера apache  
## •	В файле index.php укажите правильные учётные данные для подключения к БД  

## •	Запустите веб сервер и убедитесь в работоспособности приложения  
## •	Основные параметры отметьте в отчёте  
•	На главной странице должен отражаться номер рабочего места в виде арабской цифры, других подписей делать не надо  
•	Основные параметры отметьте в отчёте  

## 8.	На маршрутизаторах сконфигурируйте статическую трансляцию портов  
### •	Пробросьте порт 80 в порт 8080 на BR-SRV на маршрутизаторе BR-RTR, для обеспечения работы сервиса wiki  
     Настройка производится на EcoRouter BR-RTR:  
     ip nat destination static tcp 192.168.1.2 80 192.168.1.65 8080
     ip nat destination static tcp 172.16.5.1 80 192.168.1.2 8080
### •	Пробросьте порт 2024 в порт 2024 на HQ-SRV на маршрутизаторе HQ-RTR  
     Настройка производится на EcoRouter HQ-RTR:  
     ip nat destination static tcp 192.168.0.2 2024 192.168.1.65 2024  
### •	Пробросьте порт 2024 в порт 2024 на BR-SRV на маршрутизаторе BR-RTR  
     Настройка производится на EcoRouter BR-RTR:  
     ip nat destination static tcp 192.168.1.2 2024 192.168.1.65 2024  
## 9.	Настройте веб-сервер nginx как обратный прокси-сервер на ISP  
  ### •	При обращении по доменному имени web.au-team.irpo у клиента должно открываться веб приложение на HQ-SRV  
  ### • При обращении по доменному имени docker.au-team.irpo клиента
должно открываться веб приложение testapp
## 10.	На маршрутизаторе ISP настройте web-based аутентификацию:  
  ### •	При обращении к сайту web.au-team.irpo клиенту должно быть предложено ввести аутентификационные данные
         В качестве логина для аутентификации выберите WEBс паролем P@ssw0rd
        • Выберите файл /etc/nginx/.htpasswd в качестве хранилища учётных записей
        • При успешной аутентификации клиент должен перейти на веб сайт.
      
     Настройка производится на ISP:  
     en  
  
•	При обращении к HQ-RTR по доменному имени wiki.au-team.irpo клиента должно перенаправлять на BR-SRV на порт, на сервис mediwiki  
## 11.	Удобным способом установите приложение Яндекс Браузер для организаций на HQ-CLI  
•	Установку браузера отметьте в отчёте  

# Модуль № 3:Эксплуатация объектов сетевой инфраструктуры    


1. Выполните импорт пользователей в домен au-team.irpo:
• В качестве файла источника выберите файл users.csv располагающийся
в образе Additional.iso
• Пользователи должны быть импортированы со своими паролями и
другими атрибутами
70
• Убедитесь, что импортированные пользователи могут войти на машину
HQ-CLI
2. Выполните настройку центра сертификации на базе HQ-SRV:
• Необходимо использовать отечественные алгоритмы шифрования
• Сертификаты выдаются на 30дней
• Обеспечьте доверие сертификату для HQ-CLI
• Выдайте сертификаты веб серверам
• Перенастройте ранее настроенный реверсивный прокси nginx на
протокол https
• При обращении к веб серверам https://web.au-team.irpo и
https://docker.au-team.irpo у браузера клиента не должно возникать
предупреждений.
3. Перенастройте ip-туннель с базового до уровня туннеля,
обеспечивающего шифрование трафика
• Настройте защищенный туннель между HQ-RTR и BR-RTR
• Внесите необходимые изменения в конфигурацию динамической
маршрутизации, протокол динамической маршрутизации должен
возобновить работу после перенастройки туннеля
• Выбранное программное обеспечение, обоснование его выбора и его
основные параметры, изменения в конфигурации динамической
маршрутизации отметьте в отчёте.
4. Настройте межсетевой экран на маршрутизаторах HQ-RTR и BR-RTR на
сеть в сторону ISP
• Обеспечьте работу протоколов http, https, dns, ntp, icmp или
дополнительных нужных протоколов
• Запретите остальные подключения из сети Интернет во внутреннюю
сеть.
5. Настройте принт-сервер cups на сервере HQ-SRV:
• Опубликуйте виртуальный pdf-принтер
71
• На клиенте HQ-CLI подключите виртуальный принтер как принтер по
умолчанию.
6. Реализуйте логирование при помощи rsyslog на устройствах HQ-RTR,
BR-RTR, BR-SRV:
• Сервер сбора логов расположен на HQ-SRV, убедитесь, что сервер не
является клиентом самому себе
• Приоритет сообщений должен быть не ниже warning
• Все журналы должны находиться в директории /opt. Для каждого
устройства должна выделяться своя поддиректория, которая совпадает с
именем машины
• Реализуйте ротацию собранных логов на сервере HQ-SRV:
• Ротируются все логи, находящиеся в директории и
поддиректориях /opt
• Ротация производится один раз в неделю
• Логи необходимо сжимать
• Минимальный размер логов для ротации – 10МБ.
7. Насервере HQ-SRV реализуйте мониторинг устройств с помощью
открытого программного обеспечения
• Обеспечьте доступность по URL - http://mon.au-team.irpo для сетей
офиса HQ, внесите изменения в инфраструктуру разрешения доменных
имён
• Мониторить нужно устройства HQ-SRV и BR-SRV
• В мониторинге должны визуально отображаться нагрузка на ЦП, объем
занятой ОП и основного накопителя
• Логин и пароль для службы мониторинга admin P@ssw0rd
• Организуйте доступ к мониторингу для HQ-CLI, без внешнего доступа
• Выбор программного обеспечения, основание выбора и основные
параметры с указанием порта, на котором работает мониторинг,
отметьте в отчёте
72
8. Реализуйте механизм инвентаризации машин HQ-SRV и HQ-CLI через
Ansible на BR-SRV:
• Плейбук должен собирать информацию о рабочих местах:
• Имя компьютера
• IP-адрес компьютера
• Плейбук, должен быть размещен в директории /etc/ansible, отчёты в
поддиректории PC-INFO, в формате .yml. Файлы должны называется
именем компьютера, который был инвентаризирован
• Файл плейбука располагается в образе Additional.iso в директории
playbook
9. На HQ-SRV настройте программное обеспечение fail2ban для защиты
ssh
• Укажите порт ssh
• При 3 неуспешных авторизациях адрес атакующего попадает в бан
• Бан производится на 1минуту
10.Настройка резервного копирования директории сервера HQ-SRV:
• На HQ-SRV развернуть программное обеспечение для резервного
копирования и восстановления данных с защитой от вирусовшифровальщиков
• В качестве решения рекомендуется использовать программное
обеспечение Кибер Бэкап версии 17.4 или аналог
• Настройте организацию irpo
• Настройте пользователя с правами администратора на сервере
HQ-SRV, имя пользователя irpoadmin с паролем P@ssw0rd
• Установите на HQ-CLI агент с функциями узла хранилища и
подключите его к серверу управления
• На узле хранилища HQ-CLI создайте директорию /backup и
выберите её в качестве устройства хранения
• Создайте два плана резервного копирования для сервера HQ-SRV
73
• план для резервного копирования директории /etc и всех её
поддиректорий
• план для резервного копирования базы данных webdb типа
mysql
• Выполните резервное копирование директории /etc и всех её
поддиректорий сервера HQ-SRV на узел хранения HQ-CLI
• Выполните резервное копирование базы данных webdb сервера
HQ-SRV на узел хранения HQ-CLI


Необходимые приложения:  
Приложение. Файл users.csv (в отдельном файле).  
 
