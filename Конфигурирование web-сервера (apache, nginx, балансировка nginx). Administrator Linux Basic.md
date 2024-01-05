---
type: course
aliases:
  - $ Знакомство с Nginx Веб сервер на LInux
status: todo
recommendedby:
---
___ 
tags:: 
prev:: [[videos|назад в библиотеку]] 
category:: 
url:: 
children:: 
___ 

Лекция 2 декабря 2023

- [ ] установить apache;  
- [ ] установить nginx;  
- [ ] настроить apache на работу в кач-ве backend;  
- [ ] настройка портов, проверка работоспособности (curl); 
- [ ] настроить балансировку нагрузки в nginx:  
- [ ] что такое балансировка; 
- [ ] некоторые алгоритмы балансировок; 
- [ ] возможности nginx для балансировки;
- [ ] настройка upstream на бэкенд apache.

### Протокол HTTP
[10:30](https://youtu.be/rO7I3jhD_8o#t=630.2210439008179)

![[Знакомство с Nginx Веб сервер на LInux-5.jpg]]
Сервер - только отвечает, реагирует.
Подключение к серверу только по ip, но не по имени. Сервер анализирует заголовок запроса - узнает что именно нужно клиенту, какую страницу загружать.

![[Знакомство с Nginx Веб сервер на LInux-1.jpg]]

- Метод get
- url - адрес от корня сайта
- host - запрос конкретного сайта
- браузер сообщает свои возможности по чтению
	- формат картинок
	- сжатые пакеты

[18:55](https://youtu.be/rO7I3jhD_8o#t=1135.2310439256134)
![[Знакомство с Nginx Веб сервер на LInux-7.jpg]]
ответ сервера.

![[Знакомство с Nginx Веб сервер на LInux-8.jpg]]

### nginx VS apache, понятие балансировки

<iframe src="https://www.youtube.com/embed/Nh6cB8AlbIs?list=PLhgRAQ8BwWFa7ulOkX0qi5UfVizGD_-Rc" height="113" width="200" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;" allowfullscreen="" allow="fullscreen"></iframe>

[20:33](https://youtu.be/rO7I3jhD_8o#t=1233.6689539961853)
Nginx
- самый популярный
- создан для замены Apache 
	- много мелких запросов
	- высокая эффективность
- не может вырабатывать динамический контент
	- а вот Apach выполняет серверный код
		- ставят за Nginx

[24:32](https://youtu.be/rO7I3jhD_8o#t=1472.4349989523164)
Понятие балансировки
- nginx распределяет нагрузку между двумя apach
	- отказоустойчивость
![[Знакомство с Nginx Веб сервер на LInux-3.jpg]]

### установить сервер nginx
```bash
$ sudo apt install nginx
Чтение списков пакетов… Готово
...
Настраивается пакет nginx (1.18.0-6ubuntu14.4) …
Обрабатываются триггеры для man-db (2.10.2-1) …
Обрабатываются триггеры для ufw (0.36.1-4ubuntu0.1) …
Scanning processes...

```


### настройка сервера nginx
##### язык конфигурации nginx, понятия

[#8 Термины в настройках Nginx - YouTube](https://youtu.be/iGTcyvnXBmc?list=PLhgRAQ8BwWFa7ulOkX0qi5UfVizGD_-Rc)
![](https://img.youtube.com/vi/iGTcyvnXBmc/maxresdefault.jpg)

[26:02](https://youtu.be/rO7I3jhD_8o#t=1562.711387125885)
Конфигурация

![[Знакомство с Nginx Веб сервер на LInux-9.jpg]]

###### Блок директив server
[29:31](https://youtu.be/rO7I3jhD_8o#t=1771.097626)

![[Знакомство с Nginx Веб сервер на LInux-14.jpg]]

###### директива server_name
- доменные имена обращений
- можно обращаться по регулярным выражениям
###### директива listen
- директивы портов, которые суть службы сервера

###### Блок директив location
[30:04](https://youtu.be/rO7I3jhD_8o#t=1804.045715)
Различное поведение на различные адреса блок директив `location { }`
![[Знакомство с Nginx Веб сервер на LInux-15.jpg]]
nginx именно в таком порядке ищет *аллокейшн* в запросе клиента.
##### установочный конфигурационный файл nginx

```bash
/etc/nginx$ ls -l
...
-rw-r--r-- 1 root root 1447 мая 30  2023 nginx.conf
...
# бэкап конфига
$ sudo cp /etc/nginx/nginx.conf  /etc/nginx/nginx.conf.bckp
```

Установочный конфигурационный файл[^2]
```bash
$ nano nginx.conf
user www-data;

worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
# директива подключения дополнительных конфигов
# *.conf: любые файлы по этой маске будут подключены

events {
	worker_connections 768;
}
```
`worker_processes auto` см. документарцию[^1] на русском языке.


```bash
http {
	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
	# Директория, куда будем класть симлинки на наши сайты.
}

```



### включение сервера nginx в работу
[36:08](https://youtu.be/rO7I3jhD_8o#t=2168.6036020953675)
```bash
$ nginx -v
nginx version: nginx/1.18.0 (Ubuntu)
```
`18` - четное число указывает на стабильную версию, нечетное на бета версию.

[37:23](https://youtu.be/rO7I3jhD_8o#t=2243.540466)
Запущен?
```bash
# метод 1, от сервисов
$ sudo systemctl status nginx
...
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-12-03 23:29:00 UTC; 46min ago
...

# метод 2, от процессов
$ sudo ps afx | grep nginx
...
  13833 ?        S      0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
  13836 ?        S      0:00  \_ nginx: worker process

# метод 3, от сокетов
$ sudo ss -nltp
State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port  Process
LISTEN  0       511            0.0.0.0:80           0.0.0.0:*      users:(("nginx",pid=13836,fd=6),("nginx",pid=13833,fd=6))
# этот сервис слушает 80 порт
# * означает любые ip адреса
...
LISTEN  0       511               [::]:80              [::]:*      users:(("nginx",pid=13836,fd=7),("nginx",pid=13833,fd=7))
# этот сервис слушает 80 порт
...
```
nginx - однопоточный, процессов столько, сколько процессоров:
```bash
$ ps afx | grep nginx
  13833 ?        S      0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
  13836 ?        S      0:00  \_ nginx: worker process
# здесь один процесс worker
```

Утилита `curl` - *си ю-рэ-эл* - утилита загрузки какого-либо ресурса:
```bash
$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
....
```
![[Знакомство с Nginx Веб сервер на LInux-11.jpg]]

В браузере `f12` или `Ctrl+Shift+i`- средство разработчика:
![[Знакомство с Nginx Веб сервер на LInux-12.jpg]]

![[2023-12-04 11_15_26-Welcome to nginx!.jpg]]

Ответил `nginx/1.18.0 (Ubuntu)` в *заголовке ответа* сервера на запрос браузера (клиента) - указаны *заголовки запроса*. 

### Конфигурационный файл nginx.conf
[49:29](https://youtu.be/rO7I3jhD_8o#t=2969.621346)

```bash
$ nano nginx.conf
user www-data;
# пользователь, от чьего имени запускается сервер
# права ползователей, которые будут отдавать файлы
# нужно сопоставлять с правами сервера

```
действительно:
```bash
$ top -u www-data
13836 www-data  20   0   55856   5788   3880 S   0,0   0,3   0:00.02 nginx
```

```bash
# файл nginx.conf
#

worker_processes auto;
# директива о порядке процессов - по одному процессу на ядро
```

Может быть указано расположение файла логов сервера, в Ubuntu он по умолчанию папке `/var/log/nginx/`

```bash

pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
# директива подключения дополнительных конфигов
# *.conf: любые файлы по этой маске будут подключены

events {
	worker_connections 768;
	# число максимальныйх подключений на каждый worker процесс
	# если нужно, можно увеличивать
}

http {
	# блок для работы с web
	# nginx может работать и с почтой,
	#     просто с tcp трафиком
	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	# директива для типов контента
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;

	include /etc/nginx/conf.d/*.conf;
	# директива с конфигурацией нашего сайта
	# 
	include /etc/nginx/sites-enabled/*;
}

```

Разные директивы имеют разный контекст, директива `include` подключает модули - верхний уровень конфигурации - нулевая вложенность.

#### конфигурация установочного сайта /etc/nginx/sites-enabled

[58:37](https://youtu.be/rO7I3jhD_8o#t=3517.851536)
Конфигурация в директиве include /etc/nginx/sites-enabled:
```bash
$ cat /etc/nginx/sites-available/default
...
server {
        # этот блок определяет директиву сокет
        # listen
        listen 80 default_server;
        listen [::]:80 default_server;



        root /var/www/html;

		# директива определяет где находится корень сайта - 
        # где находится файл, который нужно отдать по запросу `/`
        # здесь корень находится в директории `/var/www/html`
...
        }
}
```
У нас:
```bash
$ ls -l /var/www/html
-rw-r--r-- 1 root root 612 дек  3 23:28 index.nginx-debian.html
```


[01:00:25](https://youtu.be/rO7I3jhD_8o#t=3625.188917)
- [ ] Вынести конфиг сайта в отдельный файл.
...

`30:00 лекция 20231202`

Директория, куда будем класть симлинки на наши сайты
- в директории `sites-enabled/` собраны ссылки на конфиги
	- это конфиги, включенные в работу
	- тогда как заготовленные конфиги на разные случаи, библиотеки конфигов хранят в другой папке `sites-available/`
- в директории `sites-available/` расположены библиотеки конфигов, настроенных пользователем под свои потребности
```bash
$ ls -l /etc/nginx/sites-enabled/
lrwxrwxrwx 1 root root 34 дек  3 23:28 default -> /etc/nginx/sites-available/default

```

- [x] создать собственный конфиг, дополнительный к установочному для применения его к установочному сайту nginx

```bash
$ sudo cp -r ~/sites/ /var/
# создали папку для сайтов с аттрибутами root
# под эту папку перепишем конфиг nginx

$ sudo nano /etc/nginx/nginx.conf
# это новый конфиг
user www-data;
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


        # Virtual Host Configs
        ##

        # установочный конфиг
        #include /etc/nginx/conf.d/*.conf;
        #include /etc/nginx/sites-enabled/*;
        server {
                listen 80;
                server_name 192.168.1.219;
                root /var/sites/linux_basic/;
        }
}

# проверяем работоспособность сервера
$ sudo nginx -t
$ sudo service nginx reload
$ curl localhost/index
```
- [ ] поупражняться с блоком `location` - настроить реакцию сервера на адресное выражение в запросе вот в таком духе: ![[Знакомство с Nginx Веб сервер на LInux-15.jpg]]
- [ ] 

> [!Tip] **?!** Идея сервера - понять клиента
> 
> $$\mathbf{Чего\ же\ ты\ хочешь?}$$
> 
>  - выслушать его птичий язык
> - сопоставить со своими возможностями
> - принять решение - Как реагировать на запрос?
> - отреагировать
> 
> т.о. http язык выражает **идею диалога немногословного господина (клиент) и слуги (сервера)**
> 
> $$\mathbf{Сервер\ суть\ искусственный\ интеллект}$$


- [x] создать собственный сайт, дополнительный к установочному сайту nginx
	- [x] создаем документ  для публикации в Obsidian`markdown` 
	- [x] конвертируем его в `html` средствами Obsidian
	- [x] отправляем его на web сервер средствами MobaXterm
		- [x] в папку `~/sites/linux_basic/`
		- [x] копируем этот файл в специальную папку ` /var/www/html/`
```bash
sudo cp file.html /var/www/html/doc.html
```
> [!Question] **?** Почему **?**
> nginx публикует файлы только непосредственно из папки `/var/www/html/`, и не публикует файлы по симлинкам во вне этой папки. 
>  
> Что крайне неудобно.

- [x] проверяем публикацию файла `$ curl localhost/doc.html | head`


- [ ] применить к этому новому сайту оба конфига - собственный и установочный

---

```bash
# файл /etc/nginx/sites-available/default
server {
	listen 80 default_server;
	# директива порта по умолчанию

	root /var/www/html;
	# директива корня сайта, отсюда будут браться файлы
	# для отдачи в статике

	index index.html index.htm index.nginx-debian.html;
	# какие индексные файлы будем показывать из корневой директории
	# порядок соответствует приоритету поиска файла

	server_name _;
        # директива определяет доменное имя
        #  доменное имя, на которое
        # сервер будет откликаться на запрос с заголовком
        # host
  # здесь серверное имя не важно
	# мы же будем ходить по ip адресу

 location / {
		# / означает, что это самый общий аллокейшен,
		# сюда попадают запросы, которые не попали в 
		# другие запрос

		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
		# ищем файл 
			# с запрашиваемым адресом, если это не получается, то 
			# попробуем обработать этот адрес как директорию, если и это не работает,то
			# вернем ошибку 404
		# это многуровневый обработчик для отдачи файлов
	}

}
```

Отредактировали файл и далее, проверим соответствие его правилам nginx
```bash
$ sudo vim default
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# все в порядке, перезапускаем службу
$ sudo systemctl reload nginx

# работает?
$ sudo systemctl reload nginx
administrator@u22srv:/etc/nginx/sites-enabled$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-12-03 23:29:00 UTC; 9h ago
...

$ sudo ss -ntlp | grep -P '80|8080'
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=106419,fd=6),("nginx",pid=13833,fd=6))
# да, 80 порт слушается nginx-ом

# консольный вызов url
$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
```
В браузере

Сервер запущен в конфигурации по умолчанию.
```bash
# здесь в настройках по умолчанию располагается файл для отдачи
$ cd /var/www/html
$ ls
index.nginx-debian.html
# его мы и видим в браузере
```
Т.е. в директивах мы
- указали корень с индексными файлами
- указали имена индексных файлов
- в `location` попытались найти файл с именем `/`
	- такого имени нет
	- обратился к папке корневой
	- нашел в ней индексный файл с именем `index.nginx-debian.html`
	- и выдал его в браузер

URL это - в браузере  `http://192.168.1.219/`
- `http:` - протокол
- `//` - **?** корень высшего уровня в адресации серверов
- `192.168.1.219` - адрес сервера
- `/` - адрес на сервере, внутри сервера


- [ ] сделать две разные папки с файлами для клиента
	- [ ] написать директиву серверу выдавать их
		- [ ] в порядке приоритета
			- [ ] или по шаблону в запросе
			- [ ] **?** сервер считывает у клиента именно имя нужного тому файла?
			- [ ] **?** или же просто кодовое слово, на которое реагирует?

Благодаря это директиве подключаются сайты:
```bash
# nginx.conf
http {
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```

41:00
в `/sites-enabled/` можно создать новый конфиг (вернее симлинк на него). Ссылки - для удобства, чтобы можно было быстро включать/выключать.

По nginx см. обучающие видео [[Лавлинский Николай]] на [плэйлист Nginx](https://www.youtube.com/watch?v=7XJoCZ4wsoc&list=PLc7C4rck3fYswuRgp9pI3NsAxehJwcNFD)
- [Nginx: настройка директивы location - YouTube](https://www.youtube.com/watch?v=7XJoCZ4wsoc&list=PLc7C4rck3fYswuRgp9pI3NsAxehJwcNFD)
- [Nginx: зачем нужен веб-сервер? - YouTube](https://www.youtube.com/watch?v=fo5KYjqPfWs&list=PLc7C4rck3fYswuRgp9pI3NsAxehJwcNFD&index=2)
- [Nginx multiserver: запускаем несколько сайтов на одном сервере - YouTube](https://www.youtube.com/watch?v=EuSTB84VB9E&list=PLc7C4rck3fYswuRgp9pI3NsAxehJwcNFD&index=7)
- [Настройка кэширования в Nginx - YouTube](https://www.youtube.com/watch?v=LD8m2IXQ73U&list=PLc7C4rck3fYswuRgp9pI3NsAxehJwcNFD&index=8)

### Apache
![[Знакомство с Nginx Веб сервер на LInux-17.jpg]]

Обычно выполняет серверный код на PHP, а перед ним будет стоять nginx, чтобы выдывать статические запросы и легкий код. Apache тратит много оперативной памяти на поддержку параллельных соединений. 
![[Знакомство с Nginx Веб сервер на LInux-18.jpg]]

```bash
$ sudo apt install apache2

$ sudo systemctl status apache2
× apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)

#    не смог запуститься 
	 Active: failed (Result: exit-code) since Mon 2023-12-04 10:38:26 UTC; 2min 16s ago
       Docs: https://httpd.apache.org/docs/2.4/
        CPU: 189ms

# есть ошибки
дек 04 10:38:26 u22srv apachectl[146132]: AH00558: apache2: Could not reliably determine the server's fully qualified domain >
дек 04 10:38:26 u22srv apachectl[146132]: (98)Address already in use: AH00072: make_sock: could not bind to address [::]:80
дек 04 10:38:26 u22srv apachectl[146132]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:80
# порты уже заняты другим сервисом
# не может быть запущен системный вызов bind
# один процесс - один сокет

дек 04 10:38:26 u22srv apachectl[146132]: no listening sockets available, shutting down
дек 04 10:38:26 u22srv apachectl[146132]: AH00015: Unable to open logs
дек 04 10:38:26 u22srv apachectl[146122]: Action 'start' failed.
дек 04 10:38:26 u22srv apachectl[146122]: The Apache error log may have more information.
дек 04 10:38:26 u22srv systemd[1]: apache2.service: Control process exited, code=exited, status=1/FAILURE
дек 04 10:38:26 u22srv systemd[1]: apache2.service: Failed with result 'exit-code'.
дек 04 10:38:26 u22srv systemd[1]: Failed to start The Apache HTTP Server.

```
Метод поиска нужного места в конфигах малоизвестных нам:
```bash
# пройти рекурсивно по директории /etc/apache2
# и найти все строчки про listen без учета регистра
$ grep -rni listen
ports.conf:5:Listen 80
# т.о. нужно идти в этот файл

ports.conf:8:   Listen 443
ports.conf:12:  Listen 443
apache2.conf:36:#   supposed to determine listening ports for incoming connections which can be
apache2.conf:149:# Include list of ports to listen on

$ sudo nano ports.conf
...
# устанавливаем другой порт прослушивания

Listen 8090
# Listen 80


<IfModule ssl_module>
#       Listen 443
# закрываем директиву на всякий случай, чтобы не 
# пересекаться с Nginx
</IfModule>

<IfModule mod_gnutls.c>
#       Listen 443
# тоже
</IfModule>

# проверяем синтаксис конф файлов в директории
$ apachectl -t
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK

```
Запускаем:
```bash
$ sudo systemctl restart apache2
$ sudo ps afx
...
 153414 ?        Ss     0:00 /usr/sbin/apache2 -k start
 153415 ?        Sl     0:00  \_ /usr/sbin/apache2 -k start
 153416 ?        Sl     0:00  \_ /usr/sbin/apache2 -k start

$ sudo ss -ntlp
State           Recv-Q           Send-Q                     Local Address:Port                     Peer Address:Port          Process
LISTEN          0                511                              0.0.0.0:80                            0.0.0.0:*              users:(("nginx",pid=106419,fd=6),("nginx",pid=13833,fd=6))
...
LISTEN          0                511                                    *:8090                                *:*              users:(("apache2",pid=153416,fd=4),("apache2",pid=153415,fd=4),("apache2",pid=153414,fd=4))
...


curl localhost:8090
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2022-03-22
...
```
В браузере:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx).jpg]]
... ответ пришел от сервера Apache.

Обновим страницу и получим:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-1.jpg]]
... здесь ответил Nginx. Разбираемся...
Дело в том, что одна и таже директория является корнем для обоих серверов:
```bash
/var/www/html$ ls -l
-rw-r--r-- 1 root root 10671 дек  4 10:38 index.html
-rw-r--r-- 1 root root   612 дек  3 23:28 index.nginx-debian.html
```
Туда apache дописал свой index, который и обрабатывает nginx, когда слышит вопрос на 80 порт.
А в конфигурации nginx есть строка
```bash
	index index.html index.htm index.nginx-debian.html;
	# какие индексные файлы будем показывать из корневой директории
	# порядок соответствует приоритету поиска файла
```
файл `index` имеет преимущество перед `index.nginx-debian.html`, поэтому nginx выгружает его клиенту.

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-7.jpg]]
Картинка имеет код 404 - не найден файл с ней.

### FrontEdn и BackEnd роли Reverse proxy
1:02:44
![[Pasted image 20231204211741.jpg]]
![[Pasted image 20231204211741.jpg]]

В закладке `Doc` запросы как правило динамические, 
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-4.jpg]]

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-30.jpg]]

В закладке `JS,CSS, Font, Img` запросы как правило статические - это файл на диске, 
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-3.jpg]]

Переходим на другой сервер, где уже все настроено:

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-5.jpg]]
- nginx слушает 80 порт
- apache слушает 4 порта 8080-8083

1:09:38
Новые директивы nginx:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-6.jpg]]
```bash
server {
		listen 80 default_server; 
		listen [::]:80 default.server;

		root /var/wmv/html;

		# Add index.php to the list if you are using PHP 
		index index.html index.htm index.nginx-debian.html;

		server.name site.ru mvw.site.ru;

		location / {
			# все что попадет на этот локейшен, попадет на Apache
			
			#proxy_pass http://backend; 
			proxy_pass http://localhost:8080;

			# для того, чтобы прокси сервер понимал, откуда пришел запров
			# добавляется несколько заголовков из практики проксирования
			proxy_set_header Host $host;

			proxy_set_header X-Foryjarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Real-IP $renote_addr;
			# First attempt to serve request as file, then
			# as directory, then fall back to displaying a 404. 
			#try_files $uri $uri/ @apache;
		}


```
На этой машине nginx нашел недостающую картинку:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-8.jpg]]
nginx здесь переправил запрос на Apache. 

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-9.jpg]]
Картинка хранится в папке `/icons/`. Манипуляция
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-10.jpg]]
обрабатываем nginx этот локайтион, но не подставляем root. Этот локейшен будет иметь приоритет для `icons` над локайшном с проксированием на apache. А так как директива root закоментарена, то вклчится директива root уровня высшего:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-11.jpg]]
Тот же сервер, но картинка пропала:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-12.jpg]]

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-13.jpg]]

nginx здесь сам обрабатывает запрос на директиву location и делает теперь так:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-14.jpg]]

именно это и написано в логе:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-15.jpg]]

А вот в apache есть свой location
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-16.jpg]]
со своим правилом обработки. Здесь переопределяется корень для запроса `/icons`
Если это правило включить в nginx, то мы увидим картинку.
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-17.jpg]]
мы ему объяснили, где лежат `icons`
### Балансировка Nginx
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-18.jpg]]
здесь работает три сервера, каждый из них обслуживает свой сайт - свои директории.
1:20:08
Нужно взять группу apache серверов и отправить на них nginx
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-19.jpg]]

Изменяем так:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-20.jpg]]
`proxy_pass` ссылается теперь на `backend`, а не на `localhost:8080`
Список `backend` - это правило чередования ответов - по очереди.

>Добавили всего пару строчек в конфигурацию, добавив всего пару строчек. Это - здорово!

Если в `backend`адреса разных машин, то получаем
- балансировку
- отказоустойчивость

три наших сервера `backend` - это и есть кластер (если они на разных машинах, конечно). 

Обычно отказоустойчивость реализуют так:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-21.jpg]]

Конфиг apache:
![[2023-12-04 22_21_21-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
здесь все в одном конфиге, это не правильно - разные сайты нужно разносить в разные файлы.
В директивах `listen` пишем виртуальные хосты:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-23.jpg]]
на nginx мы это делали в блоках server.

![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-24.jpg]]
В этих директориях разные индексные файлы.

Это отвечает nginx
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-25.jpg]]
мы просто обновляем его страницу `F5`, а директива `backend` балансирует - перебирает сервера apache, у каждого из которых своя страница.

### Упражнения
- [ ] отключить один из хостов на стороне apache, и посмотреть что будет на стороне nginx. Должна быть демонстрация отказоустойчивости - пока есть рабочие apache, ответы будут приходить.
	- см. ![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-26.jpg]]
- [ ] ![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-27.jpg]]

Вариант отказоустойчивости:
![[2023-12-04 22_34_19-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
- второй сервер включается, если умер первый
- третий не используется вообще, он - в холодном резерве

Способ проверки в консоли - страница постоянно меняется:
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-29.jpg]]
### Домашнее задание

Настроить веб-сервер с балансировкой. FrontEnd — nginx, BackEnd — apache

Цель:

В результате выполнения ДЗ вы создадите базовый скелет web-сервера с балансировкой нагрузки.  
В данном задании тренируются навыки:

- декомпозиции предметной области
- установка ПО на сервер, работа с файлами конфигурации
- построения элементарной архитектуры FrontEnd/BackEnd web-сервера с балансировкой нагрузки 

Описание/Пошаговая инструкция выполнения домашнего задания:

Необходимо:

- [x] установить nginx и apache
- [x] настройить работу apache на порты отличные от порта 80
- [x] настройть рабту nginx на порт 80
- [x] настроить upstrean в nginx для BackEnd apache
- [x] настроить перенаправление обращения ngin на upstream.

Критерии оценки:

- [x] установлены nginx и apache, они работают на разных портах - 3 балла
- [x] настроен upstream в nginx на 3 разных порта apache - 4 балла
- [x] в nginx для сайта на порту 80 прописан proxy_pass на upstream - 3 балла  
	- Статус "Принято" ставится от 6 баллов.

*Если вы хотите получить более сложное задание, обратитесь, пожалуйста, к ментору.
  
Компетенции:

- умения работать с сервисами Linux
    - - умение работать с веб сервером nginx
    - - умение работать с веб сервером apache

Рекомендуем сдать до: 05.12.2023

Прислать:
- скрин консоли с `curl`
- конфиг nginx-а
- apache конфиг не нужен

- [x] [[настроить nginx сервер статических html файлов]]
- [x] [[[настроить nginx как простой прокси-сервер]]]
- [x] [[настроить nginx как http балансировщик]]
### веб сервер как идея диалога
Идея веб сервера - это диалог в платоновском смысле. Человек беседует со своим вторым я - ящиком для заметок, Zettelkasten:
- клиент - это человек
- сервер - ящик для заметок

Google и Yandex выступают таким ящиком для заметок. Ящик реагирует на запросы человека, пытаясь наилучшим образом ответить на них.

Кажется, nginx, будучи детищем Rambler, как раз и предполагал технологию поиска карточки-файла в ящике-картотеке. Специфика технологии - не один вопрошающий у ящика, а многие. 
Кажется, эта технология опирается на форму виртуальной файловой системы ОС Liunux, на алгоритмы поиска файлов и строк текста в файлах. Вот те сущности, с которыми оперирует всеобщий ящик-сервер
- дерево имен и узлов (Linux сущность)
- файл (Linux сущность)
- директория (Linux сущность)
- символьная строка, текстовая строка (Bash сущность)
- текст как единство текстовых строк (Perl сущность)
- регулярное выражение (Perl сущность)

Чем же web-сервер отличен от Linux компьютера вообще - многопоточной и многопольовательской машины? Кажется, web сервер это такой Linux компьютер, который
- относится к другому компьютеру,
	- это компьютер в составе хаотической сети
- разговаривает с другими компьютерами (членами сети) на языке HTTP
- предназначен именно для разговора с человеком на человеко подобном языке - тексте, гипертексте HTTP

Хаотическая сеть - среда обитания web сервера, а значит он должен обладать средствами противостояния хаосу - доступность и отказоустойчивость. Отказоустойчивость - обязательный атрибут индивида в такой среде.
![[Конфигурирование web-сервера (apache, nginx, балансировка nginx)-18.jpg]]

Доступность и отказоустойчивость в WWW реализуется как:
- клиент имеет много линий связи с порталом ящика, шлюзом ящика
- шлюз, портал ящика очень быстр
	- именно скорость его определяющее качество
	- вдумчивое, медленное исследование вопроса ему запрещено, ему не положено
		- этим будет заниматься специальный дотошный слуга
- дотошный слуга, специалист библиотекарь,
	- предназначенный для зарывания в книжки, для копания в них глубоко

Распределение ролей быстрого и медленного библиотекарей, служащих одному лишь человеку - это роли отказоустойчивости устройств, включенных в персональный Zettelkasten внутренний и внешний мозг одного человека. Здесь доступность и отказоустойчивость могут пониматься так:
- множество терминалов множества типов
	- планшет
	- рабочая станция
	- тетрадь и карандаш
- быстрый библиотекарь - секретарь
	- файловый сервер[^3], помещающий заметку в должную папку
	- сервер связи, шлюз в Интернет и телефонные сети
- медленный библиотекарь
	- сервер Obsidian приложений - плагинов
	- сервера других приложений
		- смысловых карт
		- книжные шкафы и Zotero
- техники - слуги
	- бэкапы
	- репликации
	- сборщики логов

Упражнение для рефлексии
- папка с файлами базы знаний Obsidian на файловом сервере
- быстрый библиотекарь
	- nginx файловый сервер
		- ищет файл по запросу и отдает его браузеру
			- nginx просто передает запрос приложению Obsidian, расположенному на сервере,
			- и публикует ответ в браузере
		- включая поиск строки в файле
- медленный библиотекарь
	- apache сервер с искусственным интеллектом
		- написанном на Perl
		- или на Python
		- или на Prolog
	- и еще наводит порядок в базе Obsidian
		- так, как это делает Лавлинский - сжимает картинки и прочее

### Упражнение: один front nginx / два back apache
- [ ] front nginx 
	- [ ] принимает все входящие запросы и сортирует их
		- [ ] непонятые вопросы
		- [ ] понятые вопросы
			- [ ] справочный вопрос о файловых сервисах сервера
				- [ ] список и инструкции о файловых сервисах (html)
					- [ ] ftp
					- [ ] webdav
					- [ ] samba
			- [ ] webdav запросы
				- [ ] перенаправляет вопрос webdav серверу
				- [ ] балансирует запросы webdav серверам A и B
			- [ ] все прочие
				- [ ] обтекаемый ответ (html)
- [ ]  back webdav apache A
	- [ ] сортирует запросы на
		- [ ] не понятые
		- [ ] понятые
			- [ ] webdav сервис
				- [ ] совершает файловую операцию 
			- [ ] все прочие вопросы
				- [ ] обтекаемый ответ (html)
- [ ] back webdav apache B
	- [ ] делает ровно то же самое что  webdav apache A
	- [ ] расположен на другой машине

#### Лекции к семинару
- [Знакомство с Nginx. Веб-сервер на LInux // Демо-занятие курса «Специализация Administrator Linux» - YouTube](https://www.youtube.com/watch?v=HacbsPsdLuw)
- [Web-сервер + базовая работа в консоли (анализ логов, однострочные скрипты) - YouTube](https://www.youtube.com/live/KWueT7hiwwY?si=Ck3tpDYs90KgpqIK)


[^1]: [Основная функциональность](https://nginx.org/ru/docs/ngx_core_module.html#worker_processes)
[^2]: [Руководство для начинающих](https://nginx.org/ru/docs/beginners_guide.html)
[^3]: Идея для рефлексии
    > [!Tip] HTTP сервер как файловый?
    > Если HTTP - почти человеческий язык, то почему бы на нем не обращаться к файловому серверу?
    > 
    > Не таковы ли тэговые файловые менеджеры?