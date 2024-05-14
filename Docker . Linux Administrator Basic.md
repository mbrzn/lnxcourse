---
type: course
title: '"Docker . Otus Linux Basic"'
---
## Предисловие
###### Цели занятия

объяснить что такое контейнер;  
объяснить для каких задач были созданы контейнеры;  
объяснить когда не нужно использовать контейнер;  
поработать с контейнерами docker:  
- архитектура (рассмотреть сетевое взаимодействие контейнеров);  
- dockerhub;  
- образы (+ слои);  
- контейнеры;  
- запуск контейнеры из образа;  
- подключение к контейнеру;  
- создание образа на основании контейнера;  
- почему в образе контейнера много мусора (удаление данных в образе не уменьшает размер образа).

###### Краткое содержание

базовые понятия docker (образы, контейнеры);  
правильное применение контейнеров (когда использовать контейнер хорошо, а когда он не нужен).

###### Результаты

конспект занятия: базовые знания о docker;  
поиск образа в dockerhub, скачиOвание его на сервер и на основании образа создаст контейнер.

**Преподаватель** [[Лавлинский Николай]]

**Компетенции**
- управление безопасностью и мониторинг
    - - работать с Docker: создание образов, запуск контейнеров

Дата и время: 13 декабря, среда в 20:00

![[Docker . Otus Linux Basic.jpg]]

## Контейнеризация
![[Docker . Otus Linux Basic-1.jpg]]
>Контейниразация - виртуализация процессов.

Внутри контейнера нет ядра. А вот ядро всецело управляет аппаратными ресурсами. 

- LXC ближе к чистому Linux-у, она более открытая, Open Source-ная
- OpenVZ - отечественная Open Source, кажется увял.
	- Web хостинг - всесто VM некий контейнер

![[Pasted image 20231215042818.jpg]]
![[2023-12-15 04_28_12-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
![[2023-12-15 04_28_53-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]

![[Pasted image 20231215042858.jpg]]
Ядро есть и в гипервизоре и в хостовой машине (виртуальной).
  
Контейнер - своя файловая система. В контейнера может использоваться только Linux. 

![[Docker . Otus Linux Basic-4.jpg]]
Приложение заказа такси: эволюция от монотлита на одном сервевере, к множеству малых приложений, свзязанных по HTTP (Rest).
На практике одно приложение - сотни микросервисов.

Докер - готовый к запуску дистрибутив, экономит силы на настройку приложения. 

![[Docker . Otus Linux Basic-5.jpg]]
- Пространство имен свое для каждого контейнера
	- свои PIDы 
	- свои пользователи
	- свои сети
- Cgroups
	- [ ] ? Распределение виртуальных ресурсов?
![[2023-12-15 04_39_05-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
![[Pasted image 20231215043913.jpg]]
- Виртуализация
	- возможно все
	- но тяжеловесно
- Контейнер
	- по своей идеологии - готов к работе
		- исключается процесс конфигурации приложений
		- предполагается,что конфигурирование было сделано на этапе сборки образа (контейнера)

![[Docker . Otus Linux Basic-7.jpg]]
Процессы, которыми управляется контейнер выделены цветом.
- будем использовать для управления утилиту docker, но есть и
- дргуой канал управления (remote API)

Внутри хоста
- images - по сути дистрибутивы контейнеров
- из образом разворачиваются конкретные контейнеры
	- например, из одного образа nginx может быть получено три контейнера - разных nginx-сервера ![[Docker . Otus Linux Basic-8.jpg]]

![[Docker . Otus Linux Basic-9.jpg]]
Docker - система низлежащих технологий. Это некий конструктор (как и все в мире Linux), который удачно собран, и по верх этого намазано удобным интерфейсом. 
![[Docker . Otus Linux Basic-10.jpg]]
Целое, Docker
- протокол
	- Rest API - это шаблон проектирования, когда исполуются
		- http методы (post, put  и т.д.)
			- используемые для управления каким-либо объектом
			- базируется на нижележащем http проткоколе
				- это технология взаимодействия приложений
- клиент
- сервер

![[2023-12-15 04_53_26-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
![[Pasted image 20231215045342.jpg]]
- [ ] ? Тома - это LVM тома в хостовой LVM?
	- [ ] Том - это папка в ФС хоста, открытая контейнеру
	- [ ] Лавлинский: том - это не lvs - это тома Docker-а
-  идея тома - избежать хранения ценных данных внутри контейнера
	- вытекает из идеи контейнера - одноразовой машины


![[Docker . Otus Linux Basic-12.jpg]]
- Образ
	- готовый к использованию экземпляр приложения
	- реестр образов [Docker](https://hub.docker.com/search?q=)
	- безопасны лишь ![[Docker . Otus Linux Basic-13.jpg]], ведь каждый может опубликовать здесь свой контейнер.
		- в теории может приехать что угодно
			- руткит
			- майнер
			- шифратор данных
	- Тэги
		- пример [nginx - Official Image | Docker Hub](https://hub.docker.com/_/nginx)
		- определяет различие версий
			- содержит описание содержания контейнера
			- оно - уникально

![[2023-12-15 05_12_15-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
![[Pasted image 20231215051227.jpg]]
- Образ собирается из docker файла - это некий скрипт на специальном языке, на его выходе -
	- готовый образ (image)
	- команды скрипта
		- add
			- добавить некие файлы внутрь файловой системы образа
		- ...
	- каждая команда создает свой отдельный слой
- Создание контейнера - это
	- извлечение слоев из образа в обратном порядке
	- и собираем текущее состояние файловой системы
	- дополнительно к слоям создается `runtime read-write layer`
		- это то место, в которое в контейнере могут записываться данные
			- создание файлов, например, будет происходить здесь
		- этот слой будет уничтожен при удалении контейнера
		- [ ] ? этот слой (вместе с данными) хранится столько, сколько существует контейнер?
			- ![[Docker . Otus Linux Basic-15.jpg]] так если туда записать конфиг, то он будет удален вместе с контейнером
				- в идеале этот слой должен быть пустым
			- если нужно что-то хранить, то создаем volume и подключаем его к контейнеру ![[Docker . Otus Linux Basic-16.jpg]]
			- [Use the OverlayFS storage driver | Docker Docs](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
			- контейнеры приходят и уходят, а volume-ы остаются
			- ![](https://docs.docker.com/storage/storagedriver/images/overlay_constructs.webp)
- каждый слой - это дельта между вашим текущим состоянием и предыдущим, например
	- есть файл txt
		- туда добавлена строчка, возник слой 1
			- добавили 2-ую строчку - это второй слой
				- добавили еще строку - новый слой
				- ![[Docker . Otus Linux Basi.jpg]]
- разворачивание контейнера - это обратный порядок
	- смотрим, что было в первом слое
		- добавляем к нему изменения из второго слоя
			- третьего 
				- и т.д. ![[Pasted image 20231215053106.jpg]]
	- и вот последнюю, четвертую версию файла txt вы должны показать внутри системы
- принцип эфемерности - любой контейнер должен быть готов к тому, чтобы идти на свалку

![[Docker . Otus Linux Basic-20.jpg]]

![[Pasted image 20231215053504.jpg]]

## Установка Docker
Нам сейчас не требуется фирменный репозиторий docker-а, ставим:
```bash
# Установка Docker

$ sudo apt install docker.io
Чтение списков пакетов… Готово
...
$ ps afx
    PID TTY      STAT   TIME COMMAND
...
# процесс контейнера - клиента?
  75022 ?        Ssl    0:08 /usr/bin/containerd

# процесс контейнера - сервера?
  75362 ?        Ssl    0:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

# внесены изменения в настройки сети:
$ ip a
# новый специальный интерфейс
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
# он - мост с этой вот сетью:
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
# зачача его - связать трафик внутри контейнера с внешним миром
# что-то вроде NAT режима

$ sudo iptables -nvL
...
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
...
# docker насоздавал кучу правил и цепочек
# теперь он управляет нашим сетевым фильтром
```

```bash
# Информацию о Docker
$ sudo docker info
...
 Server Version: 24.0.5
...
 Storage Driver: overlay2
  Backing Filesystem: extfs
...
 Cgroup Driver: systemd
...
 CPUs: 1
...
 Docker Root Dir: /var/lib/docker
...
```

```bash
# Запустим тестовый контейнер
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
# не могу найти версию с тэгом по умолчанию latest
...
719385e32844: Pull complete
# часть id 1-ого слоя образа (единственный слой в этом образе)
...
Hello from Docker!
...
# контейнер просто выводти текст в консоль
```
Команда `run` содержит в себе два этапа:
- создание контейнера
- запуск контейнера

Просто запуск контейнера - это `docker start`, как и в `systemd`

```bash
$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
# это наш образ
03e4f181a9c5   hello-world   "/hello"   7 minutes ago   Exited (0) 7 minutes ago             magical_saha
# "/hello" - команда, которая внутри запускается

```

## docker nginx
```bash
# Поищем nginx
$ sudo docker search nginx
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                              Official build of Nginx.                        19366     [OK]
unit                               Official build of NGINX Unit: Universal Web …   19        [OK]
nginxinc/nginx-unprivileged        Unprivileged NGINX Dockerfiles                  138
...
# список того, что лежит на docker.hub

# хорошая практика - указывать конкретный тэг
# чего же именно ты хочешь?
# Скачаем образ
sudo docker pull nginx:1.25.3
# идем на докер хаб и качаем слои образа
1.25.3: Pulling from library/nginx
1f7ce2fa46ab: Downloading [=======>                                           ]  4.126MB/29.15MB
9b16c94bb686: Downloading [==>                                                ]  1.701MB/41.38MB
...


1.25.3: Pulling from library/nginx
1f7ce2fa46ab: Pull complete   # слой
9b16c94bb686: Pull complete   # слой
9a59d19f9c5b: Pull complete   # слой
9ea27b074f71: Pull complete   # слой
c6edf33e2524: Pull complete   # слой
84b1ff10387b: Pull complete   # слой
517357831967: Pull complete   # слой
Digest: sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee
Status: Downloaded newer image for nginx:1.25.3
docker.io/library/nginx:1.25.3


# эти семь слоев составляют наш образ
# теперь он у нас локально лежит
```

```bash
# Список активных контейнеров
sudo docker ps
# Чтобы увидеть и активные, и неактивные контейнеры
sudo docker ps -a
```
![[2023-12-15 08_16_58-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
![[Pasted image 20231215081708.jpg]]

- `-d` режим работы в фоне, как сервис
- `nginx1` назначим удобное имя
- `-p 8090:80` - проброс портов из хоста в контейнер
	- порт 8090 будет вести в 80 порт контейнера
	- примерно тоже самое в VirtualBox, чтобы включить режим NAT, для получения режима SSH
	- здесь тоже определенный вариант NAT, только внутри машины
- `-v` мы не создаем новый том в докере
	- `/var/www/html` хостовой системы будет смонтирована внутрь
	- `/ust/share/nginx/html` внутри контейнера
- закончить команду именем образа с тэгом

```bash
# Запустим контейнер
$ sudo docker run -d --name nginx1 -p 80:80 -v /var/www/html:/usr/share/nginx/html nginx:1.25.3
...
Error starting userland proxy: listen tcp4 0.0.0.0:80: bind: address already in use.
# порт 80 занят...

$ sudo ss -ntlp
State   Recv-Q  Send-Q     Local Address:Port      Peer Address:Port  Process
LISTEN  0       511              0.0.0.0:80             0.0.0.0:*      users:(("nginx",pid=774,fd=6),("nginx",pid=772,fd=6))

LISTEN  0       4096           127.0.0.1:35399          0.0.0.0:*      users:(("containerd",pid=75022,fd=11))
LISTEN  0       511                    *:8090                 *:*      users:(("apache2",pid=30788,fd=4),("apache2",pid=30787,fd=4),("apache2",pid=30786,fd=4),("apache2",pid=30785,fd=4),("apache2",pid=30784,fd=4),("apache2",pid=30783,fd=4))

# Запустим контейнер, но на какой порт его направить?
$ sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS                      PORTS     NAMES
fd049e6b29ec   nginx:1.25.3   "/docker-entrypoint.…"   About a minute ago   Created                               nginx1
03e4f181a9c5   hello-world    "/hello"                 42 minutes ago       Exited (0) 42 minutes ago             magical_saha
# контейнер таки создан, но стартовать не смог...
# поэтому, чтобы запустить, должно остановить уже работающий сервер на 80 порту - освободить порт для контейнера

$ sudo service nginx stop
$ sudo ss -ntlp
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process
...
LISTEN    0         511                      *:8090                   *:*        users:(("apache2",pid=30788,fd=4),("apache2",pid=30787,fd=4),("apache2",pid=30786,fd=4),("apache2",pid=30785,fd=4),("apache2",pid=30784,fd=4),("apache2",pid=30783,fd=4))

# Остановка и запуск контейнеров
sudo docker stop nginx1
# либо по имени
sudo docker start fd049e6b29
# либо по id (или его части)
# на самом деле id велико: fd049e6b29ec873cc4d5a097eeffd2edb37661ddc36aa4dcc110ed974ad6242a
$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                               NAMES
fd049e6b29ec   nginx:1.25.3   "/docker-entrypoint.…"   11 minutes ago   Up 2 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx1
# контейнер запущен, проброс портов есть!

$ ps afx
    PID TTY      STAT   TIME COMMAND
...
  75022 ?        Ssl    0:13 /usr/bin/containerd


# докер прокси - два процесса - перенаправляют трафик через себя, 
# запущены для 80 порта:
  75362 ?        Ssl    0:19 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

#1) для ip 4
 140111 ?        Sl     0:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 80 -container-ip 172.17.0.2 -con
# 2) для ip 6
 140116 ?        Sl     0:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 80 -container-ip 172.17.0.2 -containe

# этот процесс-родитель породил nginx-мастер процесс и worker-ы
 140135 ?        Sl     0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id fd049e6b29ec873cc4d5a097eeffd2edb37661ddc36a
 140158 ?        Ss     0:00  \_ nginx: master process nginx -g daemon off;
 140211 ?        S      0:00      \_ nginx: worker process
# т.о. мы сможем видеть worker как обычный процесс в нашей системе
# но в определенной изоляции
# внутри конетейнера id процесса 140211 будет другим...
```
- [ ] различие id процесса `140211` во вне и внутри - это лишь способ маскировки процесса?
	- [ ] какова основа различия файлов процессов во вне и внутри?
		- [ ] чем они одинаковы?
		- [ ] чем они различны?
- [ ] 


```bash
sudo ss -ntlp
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process
LISTEN    0         4096               0.0.0.0:80               0.0.0.0:*        users:(("docker-proxy",pid=140111,fd=4))
# порт слушает не nginx, а docker-proxy

$ curl -s localhost | grep title
    <title>Apache2 Ubuntu Default Page: It works</title>
```

### внутри контейнера
```bash
# Зайдём в оболочку контейнера
$ sudo docker exec -ti nginx1 bash
# дай нам оболочку в контейнере nginx1
# t - терминал
# i - интерактивно
#    т.е. команда погрузить нас в консоль bash

root@fd049e6b29ec:/#
# мы внутри контейнера
# вошли как root
root@fd049e6b29ec:/etc/nginx# ls -l
total 32
drwxr-xr-x 1 root root 4096 Dec 14 22:50 conf.d
-rw-r--r-- 1 root root 1007 Oct 24 13:46 fastcgi_params
-rw-r--r-- 1 root root 5349 Oct 24 13:46 mime.types
lrwxrwxrwx 1 root root   22 Oct 24 16:10 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  648 Oct 24 16:10 nginx.conf
-rw-r--r-- 1 root root  636 Oct 24 13:46 scgi_params
-rw-r--r-- 1 root root  664 Oct 24 13:46 uwsgi_params

root@fd049e6b29ec:/etc/nginx# nano nginx.conf
bash: nano: command not found
# здесь нет даже nano, здесь делать нечего
# будем конфигурировать этот nginx снаружи...
root@fd049e6b29ec:/etc/nginx# exit
exit

```

### управление nginx извне контейнера
```bash
# Остановка и запуск контейнеров
sudo docker stop sharp_volhard
sudo docker start d9b100f2f636

# Перезагрузка
sudo docker restart nginx
```

Логи в контейнере обычно принято перенаправлять в `stderr` и `stdout`. Их перехватывает docker и записывает их в себя:
```bash
# Логи контейнера
sudo docker logs nginx1
...
2023/12/14 23:04:19 [error] 29#29: *3 open() "/usr/share/nginx/html/icons/ubuntu-logo.png" failed (2: No such file or directory), client: 192.168.1.5, server: localhost, request: "GET /icons/ubuntu-logo.png HTTP/1.1", host: "192.168.1.68", referrer: "http://192.168.1.68/"
...
# здесь же лежит и лог скрипта, запущенного докером
# /docker-entrypoint.sh запускающий nginx внутри
# там внутри нет никаких сервисов
# или systemd

sudo docker cp
# копирование файлов в и из контейнера
```

```bash
# Информация о контейнере
sudo docker inspect nginx1
# сведения о слоях и файловой системе...
...
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2",
# внутренняя сеть контейнера
```


```bash
$ ip a
...
# это новый виртуальный интерфейс контейнера
9: veth902a5ba@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether d6:59:e3:5b:38:8b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::d459:e3ff:fe5b:388b/64 scope link
       valid_lft forever preferred_lft forever

```

? Как управлять nginx в контейнере?
- либо монтируете внутрь контейнера
	- `/etc/nginx/conf.d `
	- `/etc/nginx/nginx.conf`
```bash
# примерно так
sudo docker run -d --name nginx -p 80:80      /
	-v /var/www/html:/usr/share/nginx/html    /
	   ~/for_nginx/conf.d:/etc/nginx/conf.d          /
	   ~/for_nginx/nginx.conf:/etc/nginx/nginx.conf  /
	 nginx  
```
Хоть сто разных конфигов для ста разных контейнеров на 100 разных портах
![[Docker . Otus Linux Basic-23.jpg]]

- либо ...

```bash
# Список образов
$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         1.25.3    a6bd71f48f68   3 weeks ago    187MB
hello-world   latest    9c7a54a9a43c   7 months ago   13.3kB

```


Как собирался образ nginx?
```bash
# это и есть набор слоев
$ sudo docker history nginx:1.25.3
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
# при создании образа выполнялась такая команда
a6bd71f48f68   3 weeks ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
# она слой не порождала

# при создании образа выполнялась и такая команда тоже
<missing>      3 weeks ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
# она слой не порождала тоже

<missing>      3 weeks ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B

# это слой - у него не нулевой размер
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:9e3b2b63db9f8fc7…   4.62kB
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:57846632accc8975…   3.02kB
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:3b1b9915b7dd898a…   298B
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:caec368f5a54f70a…   2.12kB

# это тоже слой - у него не нулевой размер
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:01e75c6dd0ce317d…   1.62kB
<missing>      3 weeks ago   /bin/sh -c set -x     && groupadd --system -…   112MB
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bookworm   0B
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.8.2        0B
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.25.3     0B
<missing>      3 weeks ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>      3 weeks ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      3 weeks ago   /bin/sh -c #(nop) ADD file:d261a6f6921593f1e…   74.8MB
# всего 7 слоев. Им мы и качали
```

```bash
# Удаление контейнера
$ sudo docker rm nginx1

# форсированое удаление, с принудительной остановкой
$ sudo docker rm -f nginx1
nginx1

$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED       STATUS                   PORTS     NAMES
03e4f181a9c5   hello-world   "/hello"   2 hours ago   Exited (0) 2 hours ago             magical_saha

$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         1.25.3    a6bd71f48f68   3 weeks ago    187MB
hello-world   latest    9c7a54a9a43c   7 months ago   13.3kB

# Удаление образа
sudo docker rmi nginx:1.25.3
# если тэг не указан, то считается, что latest

```

## идея докера
1. 
настраиваем один конкретный nginx сервер, который можно перемещать на самые разные хостовые машины, с самыми разными ОС - сервер в контейнера этого не почувствует даже.

2.
пусть есть очень древняя ОС, для которой уже не пишут обновлений. Если для нее есть docker, то можно поставить самый новый nginx в контейнере. И этот nginx будет использовать свежие библиотеки openssh для шифрования, которой и в помине нет на вашей старой ОС. Так решаютя проблемы с совместимостью.

3.
запуская контейнер с предварительно настроенным вашим конфигом, вы получаете сразу же рабочий веб сервер, а не просто установленный пакет nginx.


```bash
# --restart always запускать контейнер при перезапуске системы
sudo docker run -d --restart always --name nginx -p 80:80 -v /var/www/html:/usr/share/nginx/html nginx

```

## режимы сети docker
![[Docker . Otus Linux Basic-24.jpg]]
Мы использовали **режим bridge**
- docker0 - это наш виртуальный концентратор
- из хоста заходить в контейнер мы не сможем - NAT работает в одну сторону
- чтобы сервис работал в контейнере мы делали проброс портов ![[Docker . Otus Linux Basic-25.jpg]] за счет этого мы ходили в nginx

**ражим none** полная изоляция от сети

**режим host** снимается изоляция по сети
- никакие docker proxy
- никакие iptables

ему уже не нужны. Мы его сажаем в основную сеть Linux-а. 

**режим именнованного bridge-а** - несколько контейнеров одной именованной сети могут общаться друг с другом.

```bash
# Сеть с именем mynet
docker network create mynet
docker run -d --name nginx2 --network=mynet -p 80:80 nginx
# здесь можно создать еще один контейнер, скажем с apache
# они будут друг друга видеть

# здесь изоляции сети контейнера нет
docker run -d --name nginx2 --network=host -p 80:80 nginx
```
[Networking overview | Docker Docs](https://docs.docker.com/network/)

## Сборка из образа dockerfile
```bash

# Dockerfile
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html

docker build -t webserver .

```
Простейший пример - копируется файл html в папку сборки образа и запустив команду `docker build` создать образ. 
`.` - текущая директория сборки.
![[Pasted image 20231215104012.jpg]]
На выходе получите `nginx` с измененным индексным файлом.

![[Docker . Otus Linux Basic-27.jpg]]
Более сложный пример из практики лектора.
## два контейнера в одной сети
![[Docker . Otus Linux Basic-28.jpg]]
- запустить базу данных
	- чтобы затем запустить в этой же сети
- `phpwebadmin` 
	- т.о. они смогут ходить друг к другу по своему внутреннему интерфейсу
	- а `phpwebadmin` будет еще выдан на внешний веб интерфейс
	- это такая веб морда для работы с mysql
		- параметр `--link` даже не потребуется, достаточно кинуть их в одну сеть
- причем там они смогут ходить по DNS, а не по ip, т.к. ip - прозвольны
	- название контейнера будет названием хоста

## docker compose
средство автоматизации для множества контейнеров.
![[Docker . Otus Linux Basic-29.jpg]]
1:37:10
![[Docker . Otus Linux Basic-31.jpg]]
![[Docker . Otus Linux Basic-30.jpg]]
это по сути команды докера, но в виде текстового файла - это наш конфиг. Мы описываем все контейнеры 
- db
- wordpress
- nginx
- volumes с данными
- одна сеть app-network

т.е. одной командой запустится сразу три контейнера с готовым приложением.

```bash
# Установка нескольких контейнеров, соединённых сетью
docker network create some-network

docker run --rm --network some-network --name some-mariadb -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb:10.5
docker run -it --network some-network --rm mariadb:10.5 mysql -hsome-mariadb -uroot -p

docker run --name myadmin -d --network some-network --link some-mariadb:db -p 8080:80 phpmyadmin

# Установка контейнеров для работы WordPress с помощью docker-compose

https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose-ru

# Дополнительные команды по обслуживанию
docker system df
docker system prune
docker system info

```
## Домашнее задание

![[Docker . Otus Linux Basic-32.jpg]]
- создаем nginx конфиг, где у нас несколько upstream-ов -
	- apache (или несколько), который запущен не в контейнере
		- можно apache запустить во втором контейнере
		- но лучше - не в контейнере, а хостовый
- и показать, что все это работает

Работа в Docker

Цель:
В результате выполнения ДЗ вы получите базовые навыки работы с контейнерами docker.  
В данном задании тренируются навыки:

- понимание предметной области задания
- установка ПО на сервер, работа с файлами конфигурации
- базовая работа с контейнерами"

Описание/Пошаговая инструкция выполнения домашнего задания:

Необходимо:

- установить docker
- найти ==образ nginx== и скачать его
- запустить ==контейнер nginx== на базе образа nginx
- ==подключить конфигурационные файлы nginx== из ДЗ с web-сервером в контейнер nginx

Критерии оценки:
- ==контейнер nginx запущен и работает== - 4 балла
- ==в контейнер nginx проброшены файлы конфигурации nginx с хоста== - 6 балла  
    Статус "Принято" ставится от 6 баллов.

*Если вы хотите получить более сложное задание, обратитесь, пожалуйста, к ментору.

Компетенции:

- управление безопасностью и мониторинг
    - - использование docker-compose

Рекомендуем сдать до: 15.12.2023
#### Решение ДЗ

- [x] Восстановить конфигурацию с балансировкой из ДЗ по веб-серверам
- [x] подменить веб сервер хостовый контейнерным

![[заменить хостовый nginx сервер контейнерным]]

- [x] домашнее задание выполнено
## Литература
- [[Механизмы контейнеризации namespaces]]
- [50 вопросов по Docker, которые задают на собеседованиях, и ответы на них / Хабр](https://habr.com/ru/companies/slurm/articles/528206/)
- [Play with Docker Classroom](https://training.play-with-docker.com/)
	- [Play with Docker Classroom](https://training.play-with-docker.com/alacart/)
- [Play with Docker](https://labs.play-with-docker.com/)
- [[Установка WordPress с помощью Docker]]
- [[How to Install WordPress with Docker Compose (Step by Step)]]
- [Докеризация Wordpress с помощью Nginx и PHP-FPM в Ubuntu 16.04](https://ru.linux-console.net/?p=4585)
- [Развертывание WordPress с NGINX, PHP-FPM и MariaDB с помощью Docker Compose - Блог системного администратора](https://admin812.ru/razvertyvanie-wordpress-s-nginx-php-fpm-i-mariadb-s-pomoshhyu-docker-compose.html)
- [[Docker-compose идеальное рабочее окружение]]