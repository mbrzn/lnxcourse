---
title: сайт в Hugo контейнере
---

## Hugo контейнеры, обзор
- [[Hugo How To]]
### обзор сборок
**Github**
>Hugo Docker Images
- [hugomods/hugo - Docker Image | Docker Hub](https://hub.docker.com/r/hugomods/hugo)
- [Package hugo · GitHub](https://github.com/hugomods/docker/pkgs/container/hugo)

#### hugo+git+go
**Docker репозиторий**
>The minimal, automated and up-to-date Hugo Docker images for building static sites.
- [Hugo Docker Images - Docs - HugoMods](https://hugomods.com/docs/docker/)

#### hugo+git+go+node+npm
>Image Tags[§](https://hugomods.com/docs/docker/#image-tags)
- [Hugo Docker Images - Docs - HugoMods](https://hugomods.com/docs/docker/)
```bash
# hugo+git+go+node+npm
docker pull ghcr.io/hugomods/hugo:exts-0.121.2
```
### как собрать hugo docker самостоятельно

![[install hugo in docker#^bddf89]]

## Установить hugo+git+go
###### скачать контейнер hugo
На hdd сервера должно быть достаточно места. Сколько? 2-3 gb. При недостатке места образы контейнеров попросту не собираются:
```bash
# hugo+git+go
# 
# server s0
# на нем мало места
docker pull ghcr.io/hugomods/hugo:go-git-0.121.2
sudo docker pull ghcr.io/hugomods/hugo:go-git-0.121.2
failed to register layer: archive/tar: invalid tar header
# Скорее всего на s0 недостаточно места на дисках


# server s2
# чистый сервер, с 5 gb hdd
sudo docker pull ghcr.io/hugomods/hugo:go-git-0.121.2
Digest: sha256:6f67ea4824293ea52d97fcc15ec09352552c5f77fd1f946c9fdd80f5b20029ed
Status: Downloaded newer image for ghcr.io/hugomods/hugo:go-git-0.121.2
ghcr.io/hugomods/hugo:go-git-0.121.2

sudo docker images
REPOSITORY              TAG              IMAGE ID       CREATED        SIZE
ghcr.io/hugomods/hugo   go-git-0.121.2   d38fb8226f0f   35 hours ago   491MB
hello-world             latest           d2c94e258dcb   8 months ago   13.3kB

```

## Материал для публикации - контент сайта
Готовые конфигураторы сайтов суть выражение особенной формы конкретных знаний. Какие знания, такие и конфигурации
- шаблон для документации [[Делаем документацию здорового человека в Git на примере Docs Ozon]]
- шаблон для индивидуальной базы знаний [[git/zkshell/Amethyst template|Amethyst template]]
- и пр.

### конфигурировать сайт для публикации базы знаний
- [[git/zkshell/Amethyst template|Amethyst template]]
```bash
pwd
/opt/git
# директория на сервере для хранения локальных копий
# git репозиториев
# именно git методология совершенствования, развития знаний
# служит технической основой методологии знаний вообще
# Zettelkasten / Obsidian

# Диерктория для Amethyst шаблона сайта
git clone git@github.com:mbrzn/amethyst.git
cd amethyst/
pwd
/opt/git/amethyst
# Собственно знания будут включены в директорию content
# директории Amethyst
# В эту директорию авторы Amethyst поместили докуменацию
# о конфигурировании шабона, эта докуменация и будет 
# начальным содержанием сайта базы знаний
```
## задействовать hugo контейнер
```bash
# 1313: в документации hugo-http-сервера используется этот порт 
# /src: в документации сборки hugo-контейнера используется эта директория 
# hugo-master: имя, назначенное запускаемому контейнеру

cd /opt/git/amethyst/
sudo docker run --name hugo-master -p 1313:1313 \
        -v ${PWD}:/src \
        ghcr.io/hugomods/hugo:go-git-0.121.2  \
        hugo server --bind 0.0.0.0
[sudo] password for administrator:
Watching for changes in /src/{archetypes,assets,content,data,i18n,layouts,static}
Watching for config changes in /src/config.yaml, /src/go.mod
Start building sites …
hugo v0.121.2-6d5b44305eaa9d0a157946492a6f319da38de154+extended linux/amd64 BuildDate=2024-01-05T12:21:15Z

WARN  Expand shortcode is deprecated. Use 'details' instead.

                   | EN
-------------------+-----
  Pages            | 49
  Paginator pages  |  0
  Non-page files   |  4
  Static files     | 79
  Processed images |  0
  Aliases          |  5
  Sitemaps         |  1
  Cleaned          |  0

# лог самотестирования hugo-http-сервера об адекватности
# настроек Amethysth директории требованиям hugo сервера

Built in 1597 ms
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 0.0.0.0)
Press Ctrl+C to stop

```
Представление публикации контента директории `/opt/git/amethyst/content/` в браузере:
```bash
ip a | grep 192.168.1.69
    inet 192.168.1.69/24 metric 100 brd 192.168.1.255 scope global dynamic enp0s3
```
![[git/lnxcourse/files/hugo-go-git docker.png]]

## публикация знаний посредством hugo контейнера
Авторы hugo-http-сервера предполагали такое отношение технического средства публикации и самого содержания публикации, которое подчиняется специальному пространству имен:
- корневое имя - имя каталога, здесь `amethyst`
	- директории для машины
		- archetypes/
		- assets/
		- .git/
		- resources/
		- ... прочие
	- директории для собственно знаний
		- content/

```bash
# пространство имен hugo-http-сервера
ll /opt/git/amethyst/ | grep -P '^d' | perl -pe 's/(^d)(.+\s)(\.?\w+\/$)/\3/'
archetypes/
assets/
content/
data/
.git/
.github/
i18n/
images/
layouts/
resources/
static/
```
Т.о. знания следует размещать в специальной директории `content`. Знание как нечто целое для человека следует представить как нечто целое и для машины. Такая машинная целокупность осуществляется в машинных понятиях `git`-технологии. То, что человек мыслит как целокупность надлежит представить машине как `git`-выражение. Такое выражение включает в себя возможность взаимодействия единичных машин посредством `ip`-сети, а значит и возможность *распределенного отношения единичных и взаимодействующих людей*. Следуя технологии *git* в директории `content` следует создать директорию со знанием-целокупностью, подчиняемую `git`-технологии. 
Так как человеческое знание раздроблено, представляет собой множество отдельных единиц, связанных кое-как неким смутным клеем представления, то единицы попросту рядоположены, что на машинном языке выражается рядом отдельных директорий. Значит, знание в машинной системе категорий имеет необходимую форму:
- директория отдельных единиц знания, эту директорию следует именовать `git`, поскольку она содержит отдельные машинные сущности-единицы, подчиненные `git` технологии
	- директория особенной еденицы знания, например знания об администрировании Linux сервера
		- эта директория подчиняется `git`-технологии
	- директория другой особенной еденицы знания, например знания об администрировании собственного мышления
		- она тоже подчиняется `git`-технологии
	- директория третьей особенной еденицы знания, например знания о природе
		- и эта тоже подчиняется `git`-технологии
	- и т.д.
		- тоже подчиняется `git`-технологии, поскольку директория этого особенного знания является целокупностью, единицей

```bash
# директория для знаний, машинно организованных в форме 
# git-единиц
# 
mkdir /opt/git/amethyst/content/git
cd /opt/git/amethyst/content/git
pwd
/opt/git/amethyst/content/git

# создаем директории единичных знаний, каждое из которых
# представляет собой единичную машинную сущность
# git-единицу
# 
# здесь git-единицей является lnxxourse - заметки по
# курсу "Linux администратор Basic", машинно организованные
# в git репозиторий
# машинное согласовние git репозитория amethyst и git репозитория
# lnxcourse достигается методом модульного включения
# технологии git
#
git submodule add git@github.com:mbrzn/lnxcourse.git
Cloning into '/opt/git/amethyst/content/git/lnxcourse'...
remote: Enumerating objects: 105, done.
remote: Counting objects: 100% (105/105), done.
remote: Compressing objects: 100% (91/91), done.
remote: Total 105 (delta 26), reused 86 (delta 12), pack-reused 0
Receiving objects: 100% (105/105), 1.01 MiB | 597.00 KiB/s, done.
Resolving deltas: 100% (26/26), done.

git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   ../../.gitmodules
        new file:   lnxcourse

git commit -m 'Добавлен модуль lnxcourse'
...
nothing to commit, working tree clean

```
Т.о. *человеческое знание* об управлении Linux машинами представлено как *машинный контент сайта*, пригодный для публикации посредством особенной машины - hugo-http-сервера:
![[git/lnxcourse/files/hugo-go-git docker-1.png]]