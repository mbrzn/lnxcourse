---
type: course
aliases:
  - "{ GIT. Otus. Administrator Linux. Basic"
  - "{ GIT Linux Basic"
---

___ 
tags:: 
prev:: [[courses|назад в библиотеку]] 
category:: 
url:: 
children:: 
___ 
## введение
### Преподаватель

[[Лавлинский Николай]]
### Цели занятия

объяснить, что такое система управления версиями;  
рассмотреть особенности git — распределенная система контроля версий (каждый клиент имеет полную копию репозитория);  
поговорить об основных понятиях git;  
установить и поработать с git (установка, создание репозитория, добавление файлов в репозиторий, коммит).
![[GIT-4.jpg]]

![[GIT-5.jpg]]

![[GIT-6.jpg]]

### Краткое содержание

базовая работа с системой контроля версий git;  
репозиторий (repository, repo);  
рабочий каталог (working directory);  
область подготовленных файлов (staged area);  
ревизия (revision);  
коммит (commit).  

![[GIT-7.jpg]]

Расределенная - полная локальная копия проекта.

![[Pasted image 20231218024750.jpg]]
![[GIT-26.jpg]]
Базовые термины
- Репозиторий (repository, repo)
- Рабочий каталог (working directory)
- Коммит (commit)
	- фиксация имененения, навсегда
- Область подготовленных файлов (staged area)
	- она же - индекс
	- промежуточная стадия между рабочей и фиксированной

## Локальный репозиторий
### cоздать репозиторий
```bash
gitinit	-	□	X

# Создаём каталог
mkdir repo
cd repo

# Создаём репозиторий 
git init
...
Initialized empty Git repository in /home/administrator/repo/.git/


~/repo$ ll
...
drwxrwxr-x  7 administrator administrator 4096 дек 17 17:17 .git/
# появилась директория git

########################
# права доступа к директории repo и
# вложенной git должны быть одинаковы - один владелец!



# Проверяем 
git status
On branch master

No commits yet

# удалить репозиторий - удалить папку .git
# папка repo становится рядовой

# Свойства пользователя
# изменения будут маркироваться этим пользователем
git config --global user.name "first_name last_name"
git config --global user.email email@domain

git config --global user.name "Mihail Berezin"
git config --global user.email potrebitelberezin@gmail.com


# Редактор (необязательно)
git config --global core.editor nano

```

Содержимое каталога `.git`
- Конфигурация
- Объекты (Commit, Tree, Blob) 
- Ссылки на коммиты и ветки 
- Скрипты-хуки

![[GIT-10.jpg]]

```bash
git add
# сообщить системе git о том, что файлы директории подлежат
# наблюдению
# файлы получают статус staging area

git commit
# файлы приобретают статус законченных
# и ранее проведенные изменения стадии staging 
# запоминаются как законная история продукта
```

```bash
# Добавление в индекс (stage)
git add testfile

# Просмотр проиндексированных изменений
git diff --cached

# Удаление из индекса (stage)
git rm --cached testfile

# Удаление файла (совсем)
git rm testfile
```


Оформление сообщения о коммите
- [x] Автор, email
- [x] Описание причин и сути изменений
- [ ] Ссылка на задачу в трекере
- [ ] Дата

```bash
# пример
cat testfile
This file for commit!

1. eof

git status
On branch master
# ветка мастер
No commits yet
# комитов еще не делалось
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        testfile
# обнаружен не учтенный ранее файл
nothing added to commit but untracked files present (use "git add" to track)
git add testfile
$ git status
...
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   testfile
# этот файл будет комититься
# также его можно отключить от наблюдения


# указать смысл комита
git commit -m 'First commit!'
[master (root-commit) 6513a64] First commit!
#                     кусочек id коммита
 1 file changed, 3 insertions(+)
 create mode 100644 testfile
git status
On branch master
nothing to commit, working tree clean
# рабочая директория не содержит наблюдаемых объектов


cat file2

Второй файл проекта.

1. Конец этого файла.
git status
git add file2
git status
git commit -m 'Second commit!'
[master eb93b87] Second commit!
 1 file changed, 4 insertions(+)
# один файл поменялся
#                четыре строки изменены
#                т.к. file 2 состоит именно из четырех строк
 create mode 100644 file2
# фиксируются, запоминаются права доступа на этот файл

# пусть добавлена новая строка
cat >> testfile
4. eof
git commit -m 'Test commit'
...
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   testfile
# это изменение не вносится в истории, не коммитится
# поскольку этот файл не помечен как наблюдаемый

# git может восстановить состояние файла
# на состояние последнего коммита
$ git restore testfile
$ cat testfile
This file for commit!

1. eof
2. eof
# файл восстановлен!

```

```bash
# Добавить всё
git add -A

# Список коммитов
git log

git status

# Список коммитов
git log

# Информация о последнем коммите
git show

# Информация о конкретном коммите
git show [commit_id]

# Вернуть состояние на [commit_id]
git checkout [commit_id]

# Загрузить состояние последнего коммита ветки master
git checkout master
```

Пример
```bash
$ cat > file3
Еще один файл,третий.

1. это его конец
$ git add file3
$ git commit -m 'Этот файл добавлен,чтобы быть удаленным'
[master 50c779c] Этот файл добавлен,чтобы быть удаленным
 1 file changed, 3 insertions(+)
 create mode 100644 file3


$ git log
commit 50c779c03918088fdc6f38e25020ac9c4e968f29 (HEAD -> master)
...
    Этот файл добавлен,чтобы быть удаленным
...
commit 5d6297911f44fb1db9b51f296729f219695242e0
...
    Test commit
...
commit eb93b87c41c1ec8641c8bfec675536da6b3d044f
...
    Second commit!
...
commit 6513a6486f3c0ded8303dfe1c509e68695b12cf8
...
    First commit!
# восстановим состояние файлов на коммит с таким id
$ git checkout 5d6297911f4
Note: switching to '5d6297911f4'.

# мы провалились в прошлое:
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
# предлагают создать новую ветку - начать вести параллельную историю

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 5d62979 Test commit
# мы вернулись на состояние директории помеченное 
# коммитом 'Test commit'

ll
...
-rw-rw-r--  1 administrator administrator   77 дек 17 17:53 file2
drwxrwxr-x  8 administrator administrator 4096 дек 17 18:29 .git/
-rw-rw-r--  1 administrator administrator   37 дек 17 18:10 testfile
# file3 удален

# если мы передумали, и решили продолжить историю с 
# commit 50c779c0 - когда file3 считался законным, то
$ git checkout master
# вернуться на ветку master
# в ее последнюю точку
Previous HEAD position was 5d62979 Test commit
Switched to branch 'master'

$ ll
# file3 вернулся с директорию
-rw-rw-r--  1 administrator administrator   77 дек 17 17:53 file2
-rw-rw-r--  1 administrator administrator   68 дек 17 18:43 file3
drwxrwxr-x  8 administrator administrator 4096 дек 17 18:43 .git/
-rw-rw-r--  1 administrator administrator   37 дек 17 18:10 testfile

# состояние истории на последний коммит
git show
commit 50c779c03918088fdc6f38e25020ac9c4e968f29 (HEAD -> master)
...

    Этот файл добавлен,чтобы быть удаленным

# вывод команды diff:
# различия версий файла file3 предыдущего комита и текущего
diff --git a/file3 b/file3
new file mode 100644
index 0000000..5f424cb
--- /dev/null
+++ b/file3
@@ -0,0 +1,3 @@
# так как файла file3 в предыдущем выводе не было, то
# вывод команыд diff показыват все строки, добавленные в 
# файл fil3
+Еще один файл,третий.
+
+1. это его конец
# это вывод в формате patch файла
# применение команды patch восстанавливает предыдущее состояние


```


Этапы комита:
```bash
# Редактируем файл
echo "Test new git commit" >> file.txt

# Смотрим, что изменилось
git diff

# Проиндексированные изменения
git diff --cached

# Фиксируем изменения
git commit -am "String added!"

# Смотрим историю
git log
```

## GitHub репозиторий

### регистрация в GitHub

Регистрация на github.com
- potrebitelberezin@gmail.com
- user : mbrzn

Название репозитория должно быть уникальным в рамках аккаунта, например `git_test` - т.к. это имя uri `https://github.com/mbrzn/git_test`

- [README](https://github.com/mbrzn/git_test/new/main?readme=1)
	- md файл с описанием
- [LICENSE](https://github.com/mbrzn/git_test/new/main?filename=LICENSE.md)
	- тип лицензирования для вашего проекта
- [.gitignore](https://github.com/mbrzn/git_test/new/main?filename=.gitignore)
	- это файлы, которые будут игнорироваться командой `git`


### [[клонировать github репозиторий в локальную папку]]

```bash
# Справочник по командам git hub
# Новый репозиторий - подключить к GitHub
echo "# otus_test" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:Nickmob/otus_test.git
git push -u origin main

# Залить сщуествующий репозиторий в GitHub
git remote add origin git@github.com:Nickmob/otus_test.git
git branch -M main
git push -u origin main
```

## Работа с ветками
```bash
# Справочник распределения по репозиториям

# создаем ветку
git branch feature

# Создание и переход в ветку одной командой
git checkout -b new

git branch

# Переходим в неё
git checkout feature

#
git checkout master

git add .

git commit -m 'Add new file1'

# Выливка изменений на Github
git push origin master

git remote -v
```


```bash
~/git_test$  git remote -v
# текстовая метка origin указывает на место хранения исходника
origin  git@github.com:mbrzn/git_test.git (fetch)
# для получения изменений

origin  git@github.com:mbrzn/git_test.git (push)
# для отправки изменений

$ cat > file1
Это первый файл в репозитори.

1. конец
$ git add file1
$ git commit -m 'First step of project!'
[main d341055] First step of project!
 1 file changed, 3 insertions(+)
 create mode 100644 file1
$ git status
On branch main
# мы находимся на ветке main

Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
# наша ветка main опережает ветку origin/main на 1 коммит
# т.е. ветку на git hub
# чтобы изменения появились в origin/main нужно сделать 
# команду git push

nothing to commit, working tree clean

~/git_test$ git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 357 bytes | 71.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:mbrzn/git_test.git
   4964580..d341055  main -> main
# на gin hub выливается разница между двумя комитами

~/git_test$ git log
# вот комит добавления файла в репозиторий
commit d341055113da9c5a60dcd0c160b430e53915f5fe (HEAD -> main, origin/main, origin/HEAD)
Author: Mihail Berezin <potrebitelberezin@gmail.com>
Date:   Sun Dec 17 21:30:26 2023 +0000

    First step of project!

commit 4964580efbfcc14b0c1423136d8386e88eaeaa28
Author: mbrzn <154092005+mbrzn@users.noreply.github.com>
Date:   Mon Dec 18 06:36:05 2023 +1000

    Create README.md

# a вот этот файл и его комит на gin hub:
```

![[GIT-15.jpg]]
![[GIT-16.jpg]]

Суть гита - это возможность ответвлений и, далее, соединение веток вновь:
![[GIT-17.jpg]]

```bash
# делаем ответвление
~/git_test$ git branch feature
git status
On branch main

Your branch is up to date with 'origin/main'.
# текущее положение - origin/main
nothing to commit, working tree clean

# Переходим в новую ветку
git checkout feature
Switched to branch 'feature'

git status
On branch feature
# новое положение feature
nothing to commit, working tree clean

:~/git_test$ cat > file_feature
Это файл feature

1. Конец файл

git add file_feature
git commit -m 'New feature of project!'
[feature 02c600f] New feature of project!
 1 file changed, 3 insertions(+)
 create mode 100644 file_feature

git status
On branch feature
nothing to commit, working tree clean
# на ветке feature вся история зафиксирована
```
На `git hub` пока ничего не известно о новой ветке, там есть лишь `main`:
![[GIT-18.jpg]]

```bash
# заливаем ветку на git hub
$ git push -u origin feature
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 367 bytes | 73.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'feature' on GitHub by visiting:
remote:      https://github.com/mbrzn/git_test/pull/new/feature
remote:
To github.com:mbrzn/git_test.git
 * [new branch]      feature -> feature
# создается новая ветка на гит хабе 
# логично, чтобы ветки назывались одинаково локальная -> удаленная
Branch 'feature' set up to track remote branch 'feature' from 'origin'.
# теперь наши ветки (локальная и удаленная) связаны

```
На гит хабе  в дереве веток появилась `feature`: ![[GIT-20.jpg]]
Эту ветку можно передавать другим разработчикам. 

1:27:10
Также появилась и особенная возможность гит хаба `pull requst`(выделена желтым), сокращенно `pr`.  Гость может попросить владельца репозитория влить эти изменения в ветку `main`, тогда ветка `main` изменится и станет видна измененной всеми прочими разработчиками.

![[GIT-21.jpg]]
Гит хаб не обнаружил конфликтов. И предлагает слияние `merge`.
![[GIT-22.jpg]]
![[GIT-23.jpg]]

Т.о. владелец репозитория увидел изменения, которые были предложенены в ветке `feature`, согласился с ними, и влил их в ветку `main`. Теперь в ветке `main` есть `file_feature`:
![[GIT-24.jpg]]

```bash
# справочник по слиянию веток

# Сливаем измнения между ветками с конфликтом
git pull

git checkout feature

git merge master

nano test

git add .

git merge master

git checkout master

git merge feature

git push origin master

git checkout feature

# Выливаем ветку в Github
git push -u origin feature

# Откат изменений
git revert [commit ID 7 first chars]

# Создать ветку от кокретного коммента
git checkout -b feature2 [commit ID 7 first chars]

cd .git/hooks
cat > pre-commit
#!/bin/bash

echo "Hello buddy!!!"

chmod +x pre-commit

```

Можно выливать на гит хаб уже имеющийся локальный репозиторий:
![[GIT-25.jpg]]


## Результаты

- установка git;  
- создание собственного тестового проекта в git.

##### Компетенции

- управление безопасностью и мониторинг
- работа в Git

## [[GIT Linux Basic, домашнее задание]]

## Литература
1. [Explain Git with D3](https://onlywei.github.io/explain-git-with-d3/)
> Визуализация концепций Git с помощью D3. 
> Этот сайт предназначен для того, чтобы помочь вам понять некоторые основные концепции git визуально.
3. [Learn Git Branching](https://learngitbranching.js.org/?locale=ru_RU)
>Это приложение создано, чтобы помочь новичкам постичь мощные возможности ветвления и работы с git. Мы надеемся, что вам понравится эта игра и может вы что-то усвоите!

3. [Oh Shit, Git!?!](https://ohshitgit.com/)
>Итак, вот некоторые плохие ситуации, в которые я попал, и как я в конечном итоге выбрался из них _на простом английском_.

4. [[Git-flow, реальная практика]]