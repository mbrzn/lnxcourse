## виртуальная ip-сеть Linux

Посредством программы Proxmox VE виртуализации устройств созданы две ip-сети и исследовано их ip-отношение.

Последовательно рассмотрены отношения сетей:
- красная <-> синяя
- синяя <-> зеленая
- синяя <-> Inet
![[git/lnxcourse/files/Linux маршрутизация.jpg]]
## ip-сеть как две простые противоположности
Две сети, отношение которых подлежит исследованию,  организованы вокруг двух виртуальных концентраторов *Linux bridge* и одного виртуального Ubuntu-маршрутизатора, их соединяющего.

Каждая из частных виртуальных сетей организована вокруг собственного виртуального концентратора. См. пример в публикации:
![[git/lnxcourse/Настраиваем сеть в Proxmox Virtual Environment#Частная сеть]]

Сеть `192.168.101.0/24` организована на основании виртуального концентратора *Linux bridge* `vmbr101`
![[git/lnxcourse/files/Ubuntu router Lab-6.png]]

Две виртуальные Ubuntu-машины ассоциируются[^1] с этим виртуальным концентратором посредством виртуальных NIC.  Внутренний узел сети имеет один NIC:
![[git/lnxcourse/files/Ubuntu router Lab-7.jpg]]

Пограничный узел сети - Ubuntu маршрутизатор - имеет два NIC:
![[git/lnxcourse/files/Ubuntu router Lab-8.jpg]]
![[Ubuntu router Lab-10.jpg]]


Настройки узлов соседствующих сетей
- [x] `192.168.101.0/24` 
- [x] `192.168.102.0/24`

#### первая противоположность, сеть `192.168.101.0/24` 
```yaml
# узел @s105
# /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    enp6s18:
      #dhcp4: true
      dhcp4: no
      addresses: [192.168.101.105/24]
      # net associated with vmbr101 PVE
      routes:
        #- to: 0.0.0.0/0
        - to: default
          via: 192.168.101.104
  version: 2
```
```bash
# узел s105
# 
sudo netplan apply

ip a
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:3a:f7:27 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.105/24 brd 192.168.101.255 scope global enp6s18
       valid_lft forever preferred_lft forever

berezin@s105:~$ ip route show
default via 192.168.101.104 dev enp6s18 proto static
192.168.101.0/24 dev enp6s18 proto kernel scope link src 192.168.101.105
```
#### граница противоположностей, маршрутизатор `192.168.101.0/24 <-> 192.168.102.0/24` 
```bash
# узел @s104rt
# nano /etc/netplan/00-installer-config.yaml
#
network:
  ethernets:
    enp6s21:
    # мост vmbr101 частной сети PVE
      dhcp4: no
      addresses: [192.168.101.104/24]
      # сеть, ассоциированная с ВМ, подключенными к vmbr101
    enp6s20:
    # мост vmbr102 частной сети PVE
      dhcp4: no
      addresses: [192.168.102.104/24]
  version: 2
```
```bash
# узел @s104rt
#
sudo netplan apply


4: enp6s20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:d0:01:5e brd ff:ff:ff:ff:ff:ff
    inet 192.168.102.104/24 brd 192.168.102.255 scope global enp6s20
       valid_lft forever preferred_lft forever
5: enp6s21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:f5:f6:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.104/24 brd 192.168.101.255 scope global enp6s21
       valid_lft forever preferred_lft forever
```

#### вторая противоположность, сеть `192.168.102.0/24` 
```bash
# узел @s106
# nano /etc/netplan/00-installer-config.yaml
#
network:
  ethernets:
    enp6s18:
      #dhcp4: true
      dhcp4: no
      addresses: [192.168.102.106/24]
      # net associated with vmbr102 PVE
      routes:
        #- to: 0.0.0.0/0
        - to: default
          via: 192.168.102.104
  version: 2
```
```bash
# узел @s106
# 
sudo netplan apply

ip a
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:b3:fd:cb brd ff:ff:ff:ff:ff:ff
    inet 192.168.102.106/24 brd 192.168.102.255 scope global enp6s18
       valid_lft forever preferred_lft forever
```

#### единство противоположностей, мониторинг связанности сетей `192.168.102.0/24 <-> 192.168.101.0/24`
```bash
# узел @s105
#
ping -n -4 -c 3 192.168.102.106
PING 192.168.102.106 (192.168.102.106) 56(84) bytes of data.

--- 192.168.102.106 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2039ms

# узел @s106
#
ping -n -4 -c 3 192.168.101.105
PING 192.168.101.105 (192.168.101.105) 56(84) bytes of data.

--- 192.168.101.105 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2030ms
```
- [x] ip-пакеты не маршрутизируются - на Ubuntu-маршрутизаторе отключен т.н. ip-`forwarding`
```bash
# @s104rt
#
sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

sudo sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

##################################################
# @s105
#
ping -n -4 -c 3 192.168.102.106
PING 192.168.102.106 (192.168.102.106) 56(84) bytes of data.
64 bytes from 192.168.102.106: icmp_seq=1 ttl=63 time=0.634 ms
64 bytes from 192.168.102.106: icmp_seq=2 ttl=63 time=0.482 ms
64 bytes from 192.168.102.106: icmp_seq=3 ttl=63 time=0.563 ms

--- 192.168.102.106 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2040ms
rtt min/avg/max/mdev = 0.482/0.559/0.634/0.062 ms

##################################################
# @s106
# 
ping -n -4 -c 3 192.168.101.105
PING 192.168.101.105 (192.168.101.105) 56(84) bytes of data.
64 bytes from 192.168.101.105: icmp_seq=1 ttl=63 time=0.571 ms
64 bytes from 192.168.101.105: icmp_seq=2 ttl=63 time=0.529 ms
64 bytes from 192.168.101.105: icmp_seq=3 ttl=63 time=0.589 ms

--- 192.168.101.105 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2041ms
rtt min/avg/max/mdev = 0.529/0.563/0.589/0.025 ms

```

Связанность ip-сетей Linux `192.168.102.0/24` и `192.168.101.0/24` установлена.
## ip-сеть как два субъекта, пограничный брэндмауэр

Здесь рассматривается отношение зеленой и синей сетей: ![[git/lnxcourse/files/Linux маршрутизация.jpg]]
Частная сеть `192.168.1.0/24`, граничащая с сетями гипервизора Proxmox представлена узлами
- `192.168.1.5` рабочая станция Windows 7 PC
- `192.168.1.1` маршрутизатор OpenWRT Zyxel Kinetic II
- концентратором
- `192.168.1.2` сервер виртуализации Proxmox VE

См. пример в публикации:
![[git/lnxcourse/Настраиваем сеть в Proxmox Virtual Environment#Внешняя сеть]]

Граница частных сетей `192.168.1.0/24` и `192.168.101.0/24` управляется посредством брэндмауэра Proxmox VE. Ubuntu-маршрутизатор `s104rt` здесь дополнен виртуальным NIC, подключенным к виртуальному концентратору *Linux bridge*, ассоциированному с сетью `192.168.1.0/24`:
![[git/lnxcourse/files/Ubuntu router Lab-3.png]]
![[git/lnxcourse/files/Ubuntu router Lab-11.jpg]]
Именно посредством концентратора `vmpr0` гипервизор Proxmox VE соединен с концетратором аппаратной части сети `192.168.1.0/24`.

Виртуальный ethernet NIC Ubuntu-маршрутизатора в сети `192.168.1.0/24` настроен так:
```yaml
# узел @s104rt
# nano /etc/netplan/00-installer-config.yaml
# секция   ethernets дополнена:
#
    enp6s18:
    # мост vmbr0 общий с PVE, подлючен к коммутатору LAN
      dhcp4: no
      addresses: [192.168.1.104/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

```

```bash
# узел @s104rt
#
sudo netplan apply

ip -4 a show up
...
# сеть, пограничная с виртуальными сетями Proxmox
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.1.104/24 brd 192.168.1.255 scope global enp6s18
       valid_lft forever preferred_lft forever

# виртуальная сеть Proxmox
4: enp6s20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.102.104/24 brd 192.168.102.255 scope global enp6s20

# виртуальная сеть Proxmox
	   valid_lft forever preferred_lft forever
5: enp6s21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.101.104/24 brd 192.168.101.255 scope global enp6s21
       valid_lft forever preferred_lft forever


ip route show
default via 192.168.1.1 dev enp6s18 proto static
192.168.1.0/24 dev enp6s18 proto kernel scope link src 192.168.1.104
192.168.101.0/24 dev enp6s21 proto kernel scope link src 192.168.101.104
192.168.102.0/24 dev enp6s20 proto kernel scope link src 192.168.102.104

```

Исследуем связанность узлов `192.168.1.5` и `192.168.101.105`.
#### Сигнализация из аппаратной сети в виртуальную сеть Proxmox VE
Аппаратный узел `192.168.1.5`(Windows 7 PC) настроен на маршрут в виртуальную сеть `192.168.101.0/24`:
```cmd
# маршрут в частную сеть 
C:\Windows\system32>ROUTE add 192.168.101.0 mask 255.255.255.0 192.168.0.104 if 20
# где if - это интерфейс lan1:

C:\Windows\system32>ipconfig
Ethernet adapter lan1:

   DNS-суффикс подключения . . . . . : lan
   IPv4-адрес. . . . . . . . . . . . : 192.168.1.5
   Маска подсети . . . . . . . . . . : 255.255.255.0
   Основной шлюз. . . . . . . . . : 192.168.1.1
```

Сигнализация в `distination` сеть `192.168.101.0/24`:
```cmd
C:\Windows\system32>ping 192.168.101.105

Обмен пакетами с 192.168.101.105 по с 32 байтами данных:
Превышен интервал ожидания для запроса.
Превышен интервал ожидания для запроса.

Статистика Ping для 192.168.101.105:
    Пакетов: отправлено = 2, получено = 0, потеряно = 2
    (100% потерь)
```
Мониторинг входящих сигналов на целевом узле `192.168.101.105`: 
```bash
ip a | grep 192
    inet 192.168.101.105/24 brd 192.168.101.255 scope global enp6s18

berezin@s105:~$ sudo tcpdump -i enp6s18 -n src 192.168.1.5
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp6s18, link-type EN10MB (Ethernet), snapshot length 262144 bytes
...
```
Входящих сигналов нет, по-видимому, они блокируются брэндмауэром Proxmox VE виртульного *Linux bridge* `vbmr0`.
#### Сигнализация из виртуальной сети Proxmox VE во вне, в аппаратную сеть
```bash
berezin@s105:~$ ping 192.168.1.5
PING 192.168.1.5 (192.168.1.5) 56(84) bytes of data.
^C
--- 192.168.1.5 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2040ms
```
Мониторинг соединения на стороне `distination`узла `192.168.1.5`:
![[git/lnxcourse/files/Ubuntu router Lab-5.jpg]]

Т.о. целевой узел реагирует на входящие сигналы. Реактивные сигналы, по видимому блокируются брэндмауэром Proxmox VE виртульного *Linux bridge* `vbmr0`.
#### вопросы для исследования
- [ ] проверить гипотезу о блокировании входящих сигналов брэндмауэром Proxmox VE виртульного *Linux bridge* `vbmr0`
	- [ ] отключить брэндмауэр
	- [ ] перезапустить 
		- [ ] PVE гипервизор
		- [ ] s105 узел `192.168.101.105`
		- [ ] s104rt узел `192.168.1.104 <-> 192.168.101.105`
- [ ] исследовать правила фильтрации трафика через виртуальный *Linux bridge* `vbmr0`.

## ip-сети как множество субъктов, единство частных и общественных сетей
Здесь рассматривается отношение синей сети и Inet.

![[git/lnxcourse/files/Linux маршрутизация.jpg]]

В технологии ip частная сеть не относится к публичной - они друг для друга суть пустые абстракции:
```bash
# узел s105
# 

ip -4 a show up
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.101.105/24 brd 192.168.101.255 scope global enp6s18
       valid_lft forever preferred_lft forever

# 192.168.101.105 --> 8.8.8.8
ping -n -4 -c 2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1005ms

# общественное узел с имеенем 8.8.8.8 ингнорирует узел с именем 192.168.101.105
```

Чтобы частная сеть относилась к публичной, она обязан предстать на публике как член этой самой публики, т.е. иметь общественное имя. Это общественное имя частная сеть берет на прокат у того, кто это имя имеет от рождения - у некоторого узла общественной сети. 

>Маршрутизатор теперь является посредником между внутренним хостом и пунктом назначения. Адресат ничего не знает о внутреннем хосте — соединение на удаленном хосте выглядит так, как будто оно пришло от маршрутизатора.[^2]

>Обычная IP-маршрутизация знает только IP-адреса источника и назначения на сетевом интернет-уровне. Но если бы маршрутизатор работал только с интернет-уровнем, каждый хост во внутренней сети мог бы одновременно устанавливать только одно соединение с одним пунктом назначения (среди прочих ограничений), поскольку в части пакета интернет-уровня нет информации, позволяющей различать несколько запросов от одного и того же хоста к одному и тому же пункту назначения. Следовательно, NAT должен выходить за рамки интернет-уровня и анализировать пакеты, чтобы извлекать больше идентифицирующей информации, в частности номера портов UDP и TCP транспортных уровней.[^3]

Это заимствование, аренда общественного  имени осуществляется посредством технологии `NAT`. Последняя реализована в Linux-программе `netfilter` и утилиты `iptables`.

>Чтобы настроить компьютер Linux для работы в качестве маршрутизатора NAT, вы должны активировать следующие функции в конфигурации ядра: фильтрацию сетевых пакетов (поддержку брандмауэра), отслеживание соединений, поддержку iptables, полную поддержку NAT и целевую поддержку MASQUERADE. [^4]

```bash
# узел @s104rt
#
ip -4 a show up

2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.1.104/24 brd 192.168.1.255 scope global enp6s18
       valid_lft forever preferred_lft forever
4: enp6s20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.102.104/24 brd 192.168.102.255 scope global enp6s20
       valid_lft forever preferred_lft forever
5: enp6s21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.101.104/24 brd 192.168.101.255 scope global enp6s21
       valid_lft forever preferred_lft forever


# MASQUERADE 
#
sudo sysctl -w net.ipv4.ip_forward=1 
sudo iptables -P FORWARD DROP
# арендовать общественное имя для исходящего трафика
# из сети 192.168.101.0/24 в Inet
# через интерфейс enp6s18
#
sudo iptables -t nat -A POSTROUTING -o enp6s18 -j MASQUERADE 

# ответный трафик (реакция на исходящий трафик)
# из сети Inet в сеть 192.168.101.0/24 будет входить
# через интерфейс enp6s18
# и выходить через интерфейс enp6s21 
sudo iptables -A FORWARD -i enp6s18 -o enp6s21 -m state --state ESTABLISHED,RELATED -j ACCEPT 
sudo iptables -A FORWARD -i enp6s21 -o enp6s18 -j ACCEPT

# таблица правил
sudo iptables -L -v --line-numbers
```

```bash
# узел s105
# 192.168.101.105 <--> 8.8.8.8
#
ping -n -4 -c 2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=105 time=127 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=105 time=127 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 126.625/126.737/126.850/0.112 ms
```

- [x] Связанность частной и общественной ip-сетей установлена.
- [x] [[Ubuntu Router 18.04#Сохранение правил netfilter]]

---
[^1]: Назначение виртуальной машине eth NIC и назначает ему MAC адреса гипревизором:
	![[git/lnxcourse/files/Ubuntu router Lab-1.png]] 
	Назначение ВМ-е новпого NIC требует  перезагрузки PVE - ВМ запускаться не будет с сообщением об ошибке `proxmox VM Start bridge 'vmbr' does not exist`.
[^2]: [Уорд. 2022. *Внутреннее устройство Linux*](zotero://select/items/_8PYIQ765) с. 307
[^3]: [Там же,  с. 307](zotero://select/items/_8PYIQ765)
[^4]: [Там же,  с. 307](zotero://select/items/_8PYIQ765)