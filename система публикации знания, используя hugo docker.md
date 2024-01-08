---
title: система публикации знания с использованием hugo docker
---

- Часть 1. Как [[git/lnxcourse/опубликовать сайт в Hugo контейнере]]
## отказоустойчивость публикации знания
Катастрофа знания имеет и техническое измерение - машина или люди, стоящие за машинами, может перестать соответствовать потребностям знания:
![[Отказоустойчивость сайта  Хабр#Реальные способы обеспечения отказоустойчивости]]

### элементы системы публикации знания
В нашем упражнении техническая отказоустойчивость сайта (публикации знания) будет состоять из:
- [x] многих hugo-http-серверов, резервирующих друг друга
- [x] одного nginx-http-сервера, ищущего для читателя знания наилучший hugo-сервер с публикацией знания
- [x] одного депозитория с командами для nginx-http-сервера. Этот депозиторий подчинен `git`-технологии.
- [x] linux сервер для выполнения программы `docker` - технического основания hugo-конейнеров и nginx-контейнеров
- [x] одного депозитория со знаниями в форме `.md`-заметок, ссылающихся друг на друга. Этот депозиторий подчинен `git`-технологии.

### включение резервирующего hugo-контейнера
```bash
cd /opt/git/amethyst/
sudo docker run --name hugo-slave -p 1314:1313 \
        -v ${PWD}:/src \
        ghcr.io/hugomods/hugo:go-git-0.121.2  \
        hugo server --bind 0.0.0.0
```
Если [[git/lnxcourse/система публикации знания, используя hugo docker#задействовать hugo контейнер|master-контейнер]] настроен на порт 1313, то slave-контейнер на порт 1314.

### установка репозитория, управляющего nginx-сервером
Репозиторий включает в себе множество конфигов и скриптов для управления nginx-сервером. В этом упражнении используются:
- [ ] nginx.conf
- [ ] trafic.sh
- [ ] whererafic.sh

Конфиг и скрипты предполагается совершенствовать посредством *метода контроля версий git*, поэтому репозиторий с ними следует разместить в локальной директории  linux-хост-сервера, содержащей в себе git-репозитории, используемые на хост-сервере, у нас:
```bash
mkdir /opt/git/nginx_balancer/
cd /opt/git/nginx_balancer/
git clone git@github.com:mbrzn/nginx_balancer.git
```

```text
# nginx.conf
user nginx;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
}
http {
    # Logging Settings
    ##
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # для кирилических знаков
    charset UTF-8;

    upstream backend {                      # распределить запросы:  
                                            # очередной запрос - следующему серверу

        server 192.168.1.69:1314;           # hugo slave-контейнер резервирующий, сайт lnxcourse

        server 192.168.1.69:1313;           # hugo master-контейнер, сайт lnxcourse
    }

    server {                                # собственно сервер http запросов
                location / {
                proxy_pass http://backend;  # режим proxy, перенаправлят запросы
                                            # в распределитель backend
                }
    }
}
```
### установка балансирующего nginx-сервера

```bash
sudo docker run -d \
    --name nginx-blns \
    -p 80:80 \
    -v  /opt/git/nginx_balancer/:/etc/nginx/ \
    nginx:1.25.3
```

Теперь к hugo-контент-серверам можно обращаться отказоустойчиво - посредством nginx-балансирующего сервера:
![[git/lnxcourse/files/hugo docker.png]]

## совершенствование управления системой публикации знаний
Каждый [[git/lnxcourse/система публикации знания, используя hugo docker#элементы системы публикации знания| элемент системы публикации знаний]] должен совершенствоваться - ведь познание суть совершенствование. 

Пусть совершенствование управления состоит в
- [ ] разворачиваний дополнительного резервирующего hugo-контент сервера, предназначенного для включения только тогда, когда и master, и slave-контент сервера выйдут из строя

Изменяем конфиг nginx-балансирующего сервера:
```text
user nginx;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
}

http {
    # Logging Settings
    ##
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # для кирилических знаков
    charset UTF-8;

    upstream backend {                      # распределить запросы:  
                                            # очередной запрос - следующему серверу

        server 192.168.1.69:1315 backup;    # hugo backup контейнер резервирующий, сайт lnxcourse

        server 192.168.1.69:1314;           # hugo slave контейнер, резервирующий публикацию сайта lnxcourse

        server 192.168.1.69:1313;           # hugo master контейнер, сайт lnxcourse
    }
 
    server {                                # собственно сервер http запросов
                location / {
                proxy_pass http://backend;  # режим proxy, перенаправлят запросы
                                            # в распределитель backend
                }
    }
}
```
Репозиторий синхронизируем с git-сервером (у нас - `github`).

Обновляем репозиторий команд nginx сервера на linux-хост сервере:
```bash
cd /opt/git/nginx_balancer/
git fetch
git pull
```

Включаем в работу резервный hugo-контент-сервер:
```bash
cd /opt/git/amethyst/
sudo docker run --name hugo-backup -p 1315:1313 \
        -v ${PWD}:/src \
        ghcr.io/hugomods/hugo:go-git-0.121.2  \
        hugo server --bind 0.0.0.0
# порт 1313: master hugo-контент сервер
# порт 1314: slave hugo-контент сервер
# порт 1315: backup hugo-контент сервер
```


Перезапускаем контейнер с nginx-балансирующим сервером:
```bash
sudo docker container restart nginx-blns
```

Проверяем действительность управления последовательно выключая из работы master и slave-контент сервера, предполагая автоматического переключения трафика на backup-контент сервер:
```shell
curl -s localhost | grep -P '<title.+Welcome'
<title>💜 Welcome to Amethyst! | 📓 Amethyst</title>

# отлючаем последовательно hugo-контент сервера
# и проверяем доступность контента
sudo docker stop hugo-master
hugo-master
curl -s localhost | grep -P '<title.+Welcome'
<title>💜 Welcome to Amethyst! | 📓 Amethyst</title>
# здесь действует hugo-slave сервер

sudo docker stop hugo-slave
hugo-slave

curl -s localhost | grep -P '<title.+Welcome'
<title>💜 Welcome to Amethyst! | 📓 Amethyst</title>
# здесь действует hugo-backup сервер
# значит конфиг nginx-балансировщика действительно обновился -
# в работу включился hugo-backup сервер через порт 1315
# не задействованный в предыдущем конфиге

sudo docker stop hugo-backup
hugo-backup
administrator@s2:~$ curl -s localhost | grep -P '<title.+Welcome'
# здесь вывода команды curl нет - значит 
# нет действующих hugo-контент сервереров
administrator@s2:~$

sudo docker start hugo-master
hugo-master
curl -s localhost | grep -P '<title.+Welcome'
<title>💜 Welcome to Amethyst! | 📓 Amethyst</title>
# доступность контента восстановлена посредством
# восстановления работоспособности hugo-master-контент сервера
```

## Что  необходимо улучшить?
### непосредственное наблюдение за портами контейнеров
Интерпретировать природу доступности/не доступности контента hugo-контент  сервером можно многими образами. А вот прямое наблюдение за портами исключает множественность интерпретаций. 
- [ ] необходимо освоить непосредственное наблюдение за портами

Каким средством воспользоваться?

###### Docker stats
- [docker stats | Docker Docs](https://docs.docker.com/engine/reference/commandline/stats/)

>Монитор `docker stats` запущенных контейнеров.

- [docker stats | Docker Docs](https://docs.docker.com/engine/reference/commandline/stats/#format)
```bash
sudo docker stats --format "{{.Container}}: {{.NetIO}}"
sudo docker stats --format "table {{.Name}}:\t {{.NetIO}}"
# потоковый вывод сетевого тарфика

sudo docker stats --format "table {{.Name}}:\t {{.NetIO}}" --no-stream
# не потоковый вывод, но единичное, одномоментное значение
```
- [x] Здесь предлагается измерять трафик в Mbit, что очень грубая единица измерения, 
	- [x] см. [wheretrafic.sh](https://github.com/mbrzn/nginx_balancer/blob/2d22e3678831b2ad9a1356fb53b6dd24f6f8f7b1/wheretrafic.sh)
		- [x] и синтаксический анализатор [trafic.sh](https://github.com/mbrzn/nginx_balancer/blob/2d22e3678831b2ad9a1356fb53b6dd24f6f8f7b1/trafic.sh)
- [ ] нужен другой, более чувствительный инструмент


###### Docker API метрики
- [How to Collect Docker Metrics | Datadog](https://www.datadoghq.com/blog/how-to-collect-docker-metrics/)

```bash
$ echo -ne "GET /containers/1c39ff0fcbc7/stats HTTP/1.1\r\n\r\n" | sudo nc -U /var/run/docker.sock
echo -ne "GET /containers/d9a357236073/stats HTTP/1.1\r\n\r\n" | sudo nc -U /var/run/docker.sock
HTTP/1.1 400 Bad Request: missing required Host header
Content-Type: text/plain; charset=utf-8
Connection: close
```
Эти команды не сработали, нужно с ними разбираться
- [ ] кажется, перспективный инструмент
	- [ ] нуждается в освоении
	- [ ] ориентирован на программистов на языке `Golang`

###### ss 
 - [10 Examples of Linux ss Command - Monitor Network Connections - BinaryTides](https://www.binarytides.com/linux-ss-command/)
>- Here is an example

`$ watch -n 1 "ss -t4 state syn-sent"`
>After running the above command, try opening some website in a browser or download something from some url. Immediately you should see socket connections appearing in the output, but for a very short while.

- [ ] `sgl` промелькивает, а `http` - нет. Как настроить?
- [ ] кажется, перспективный инструмент, нуждается в освоении
###### в iptables
- [networking - Traffic stats per network port - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/41765/traffic-stats-per-network-port)
>iptables может дать вам статистику о том, сколько было запущено каждое правило, поэтому вы можете добавить правила LOG на интересующих портах (скажем, порт 20 и порт 80):

```
iptables -A INPUT -p tcp --dport 22
iptables -A INPUT -p tcp --dport 80
```

>и тогда

```
iptables -n -L -v
```

>предоставит вам количество пакетов и байтов, отправленных через эти порты. Конечно, вам придется проанализировать из вывода порты, которые вас интересуют.

>Если вам нужны точные значения, добавьте a -x:

```
iptables -n -L -v -x
```

- [ ] кажется, перспективный инструмент, нуждается в освоении
