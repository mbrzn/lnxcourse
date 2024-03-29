---
type: course
title: '"Bash крипты, Administrator Linux Basic"'
---

Bash крипты, лекция курса Administrator Linux Basic
#### Термины
- `bourne again shell` - заново рожденный shell
- `#` - октотор
- `^` - циркум флекс
- $`$ - бэк тик
- `#!` - шебанг
#### Что такое bash?
> [!Tip] bash 
> \- язык программирования

Если `язык программирования` - это разговор человека с машиной на человеческом языке, то 
> [!Tip] bash - язык автоматизации командования машиной
> 
> **Команды-программы** выражают **единичные** управленческие потребности человека, сложившиеся в истории машиностроения.
> **Автоматизация** выражает **общие** управленческие потребности человека.

>bash - стандарт автоматизации Linux, его нужно знать обязательно.
>
>• Bourne-again shell
>
>• Интерактивная оболочка
>• Язык программирования
>• Стандартный вариант
>• Другие варианты
>>о zsh
>>о csh
>>о ksh
>>о sh

![[Otus. Administrator Linux. Basic-1.jpg]]

>Суть `bash` - консерватизм. Скрипт написанный 20 лет назад будет работать сейчас.

`bash` - чистота языка автоматизации
подобно тому, как перевод библии с латыни на немецкий - протестантская ересь, так и 


>Bash-скрипт
>• Текстовый файл
>• Состоит из команд и операторов bash
>• Расширение файла — обычно нет, можно .sh
>• Разделитель команд — новая строка или точка с запятой
>• Не компилируется в машинный код
>• Исполняется интерпретатором - bash
>• Как система определяет тип скрипта?

система определяет тип скрипта по первой строке текстового файла `#!/bin/bash` - полное имя интерпретатора команд

![[Otus. Administrator Linux. Basic-2.jpg]]

Типы команд
Command types
```shell
$ type -a cd
cd is a shell builtin
$ type -a fgnep fgnep is /bin/fgrep
$ type -a Is
Is is aliased to 'Is —colon=auto'
Is is /bin/ls
$ type -a for
for is a shell keyword
$ which passwd /usr/bin/passwd
```

Методы запуска
```shell
# 1 - без прав 
bash count_files.sh

# 2 - с правом на исполнение 
chmod +х count_files.sh

# Относительный путь 
./count_files.sh

# Абсолютный путь 
/home/db/count_files.sh

# 3 -из директории для исполняемых файлов 
echo $РАТН
/usr/local/bin
ср ./count_files.sh /usn/local/bin/
```

Позиционные параметры
•	`$1` - первый параметр
•	`$2` - второй
•	`$#` - количество
•	`${10}` - 10-й
•	`${11}` - 11-й

Использование параметров
```bash
# count_files. sh 
find $1 -type f | wc -l

bash count_fiT.es.sh /usr 
bash count_files.sh /etc 
bash count_fiT.es.sh
```

#### Переменные
• Временные значения
• Видны в рамках одного процесса (скрипта)
• Пользовательские переменные `$var`
• Системные переменные (константы) `$РАТН`
• Могут наследоваться через `export`

`export` наследует переменные в *дочерний* процесс

Использование переменных
```bash
################ Переменные
tnaining=Python
TRAINING=Linux

echo $TRAINING 
echo $training 
echo ${TRAINING> 
echo ${training}

# count_files.sh 
directory=$1
find $directory -type f | wc -I
```

Что это такое?
```bash
$(command)

output=$(cat /etc/passwd)

output= `cat /etc/passwd`
```

Результат команды в переменную
```shell
# count_fil.es.sh
############ Возврат значений из команд
directory=$l
num_of_files=$(find $directony -type f | wc -1) 
echo $num_of_files
```

> [!Attention]
> `bash`: вывод команды - всегда текст, **строка**

```bash
############ Специальные переменные

echo $HOSTNAME - имя хоста
echo $HOME # домашняя директория
echo $PWD # текущая директория
echo $UID # идентификатор пользователя
echo $$ # идентификатор процесса
echo $PS1 # вид командной строки
echo $? # код возврата
```

Запрос ввода пользователя, сохранение в переменную
User input
```bash
1 #//bin/bash
2
3 echo "Enter the filename:"
4 read filename
5
6 echo $filename
```
[[переменные bash]]
#### Подстановки
Подстановки (expansions)
```bash
echo {0..10} 
mkdir test{a..e} 
echo {{a..e},z} 
mkdir test{001..100} 
echo ~ 
echo ~+ 
echo ~-

```

Код возврата (exit code)
```bash
ls /etc/; echo $?

ls /abc/; echo $?
```

• Выставляется в результате любой команды
• `0` — команда выполнена успешно
• `1..255` — команда выполнилась с ошибкой (код ошибки)
• Код возврата доступен через переменную `$?`
• Позволяет организовывать условия в программе
#### Условия (if)
if and test
```shell
if [ -f $filename ] 
then
	echo "True!" 
else
	echo "False!"
fi

type -a test [ [[
test -f /etc/ ; echo $? # существует ли файл? 
test -d /etc/ ; echo $? # существует ли директория
```

Команда `test` и `[` или `[[`

- Если условие успешно, возвращает `exit code 0`, иначе ` 0`
- ` ==` - равенство.
- `! =` — неравенство.
- `-lt` — меньше.
- `-gt` — больше.
- `-lte` — меньше или	равно.
- `-gte` — больше или	равно.
- `-f` -файл.
- `-d` -директория
- `help test; help	[; help [[`
- `man test` # для внешней команды

Написать скрипт который проверяет, что пользователь передал аргумент. Если аргумента нет — выдать сообщение об ошибке и завершить программу
Задача. Conditional exit
```bash
1 #!/bin/bash
2
3 # count_files. sh
4
5 dinectony="$l"
6 find "$directony" -type f | wc -l
```

```bash
1 # Вариант 1
2 if [ -z "$directory" ]
3 then
4    echo Please specify directory
5    exit 1
6 fi
7
8 # Вариант 2
9 if [ $# -eq 0 ]
10 then
11    echo Please specify directory
12    exit 1
13 fi
14
15 # Вариант 3
16 directory=$l
17 if [ -n "$directory" ]
18 then
19    find "$directory" -type f I wc -I
20 else
21    echo Please specify directory
22    exit 1
23 fi
```

#### Циклы
Цикл for
```bash
1 #!/bin/bash
2
3 for h in {01..24}
4 do
5	echo $h
6 done
7
8 for i in *
9 do
10	echo $i
11 done
12
13 for (( c=l; c<=5; c++ ))
14 do
15    echo "Попытка номер $c"
16 done
```

Цикл while
```bash
1 #!/bin/bash
2
3 c=10
4
5 while [ $c -ge 0 ]
6 do
7	echo "Test"
8	let "с = с - 1"
9 done
```

[[слово let в программировании#магия bash]]
> [!NOTE] `let`: текст - основа bash
> команда числовой математики распознается машиной из произнесенного человеком *математикообразного текста*

#### Как отлавливать ошибки определения переменных?

Настройка bash[^1]
```bash
##### SELinux
sestatus

getenforce

# Отключить
sudo setenforce 0

# Конфиг: /etc/selinux/config поставить SELINUX в disabled

# Включить
sudo setenforce 1
```
#### Литература
1.[[Как писать bash-скрипты надежно и безопасно минимальный шаблон  Хабр]]
2. [Исскуство написания Bash-скриптов](https://www.opennet.ru/docs/RUS/bash_scripting_guide/)
3. 

[^1]: [Исскуство написания Bash-скриптов](https://www.opennet.ru/docs/RUS/bash_scripting_guide/)
#### Домашнее задание

Вопросы для проверки
1. Какие бывают типы переменных?
2. Какие типы команд вы знаете и как их узнавать? 
3. Что делает команда test?


[OTUS - Онлайн-образование](https://otus.ru/learning/261941/)

Bash-скрипт

Цель:

Напишете скрипт, который очищает папку от временных файлов. Дополнительно: скрипт для управления DNS настройками.

Описание/Пошаговая инструкция выполнения домашнего задания:

##### Задание 1.  
Необходимо написать скрипт, который очищает указанную папку от временных файлов.

1. Первым параметром получает директорию.
2. Проверяет, что указанная директория существует. Если нет – пишет ошибку и завершается с кодом 2.
3. Удаляет все файлы `*.bak`, `*.backup`, `*.tmp`. Для каждого типа выводит сообщение об удалении или об отсутствии таких файлов.
4. Прислать исходный код скрипта в кодировке UTF-8 в текстовом файле.  

```bash
# прочитать параметры и проверить их
#    параметры
#    - ключ справки
#    - имя папки
#    - расширение файла
#    - 
# проверить существование директории
# проверить каждый файл на соответствие маске
```

- [x] ![[очистить директорию]]

 ###### Задание 2*. Дополнительное.  
Необходимо написать скрипт, который управляет настройками DNS в /etc/resolv.conf.  
Прислать исходный код скрипта в кодировке UTF-8 в текстовом файле.
1. Получает на вход список из двух DNS-серверов. Если на входе меньше или больше - ошибка.
2. Проверяет текущую конфигурацию в /etc/resolv.conf (соответствует ли настройка).
2. Проверяет, есть ли права у пользователя на редактирование.
4. Если конфигурация не совпадает, то выводит об этом сообщение и меняет настройки в файле из параметров.
5. У скрипта должен быть режим справочника (помощи). В справке показывается синтаксис и описывается использование скрипта.

Материалы к Заданию 2:
- [Resolve IP адресов в Linux: понятное и детальное описание / Хабр](https://habr.com/ru/articles/352300/)
- [## Как указать DNS сервер](https://dzen.ru/a/YMvAw13zpFQckjS2)
- [dns в linux - настраиваем клиентскую часть системы dns | HippoLab - блог системного администратора](https://www.hippolab.ru/nastroyka-dns-v-linux)
- [resolv.conf - Debian Wiki](https://wiki.debian.org/resolv.conf)
- 
###### атомы домашнего задания
- [ ] скрипт получает строку параметров
	- [ ] ? всегда ли вывод команды строка?
	- [ ] ? как строка разбивается на элементы: слова, предложения, абзацы? 
		- [ ] какие это элементы в bash?
- [x] проверка существования файлового объекта и его свойств
	- [x] какие свойства файловой системы нужно проверять?
- [x] как параметризовать ввод скрипта на *такие-то типы файлов*?
	- [x] как документировать скрипт?
		- [x] параметры
		- [x] примеры
	- [x] создать каркас скрипта вокруг возможных ключей и аргументов. Образец см. в *Идиомы bash*