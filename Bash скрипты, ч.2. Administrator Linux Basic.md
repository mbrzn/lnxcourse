---
type: course
---

Вопросы и  ответы по Bash скриптам
[Вопрос-ответ по Bash](https://otus.ru/learning/261941/#)

- [[регулярные выражения языка Bash]]
- [[массивы в bash]]
#### Упражнения

- [ ] регулярные выражения внутри скрипта
- [ ] версия 2 скрипта
- [ ] определиться со стандартом рег выражений
- [ ] файловый сервер - подключить диск, разметить его и пр. 
	- по книге *внутренне устройство linux*
- [ ] Создание программных массивов
	- по книге *от новичка к профессионалу*
- [ ] скрипт по ужатию картинок в *md* файле 
- [ ] использовать такой тип регулярных выражений, который действует и в powershel и в bash


```shell
administrator@u22srv:~$ sed '
s/vim/VIM/g;
 s/jammy/JOAMMY/g
' log
```

```shell
administrator@u22srv:~$ sed 's/vim/VIM/g; s/jammy/JOAMMY/g'
log
```

```bash
sed - n 's/vim/VIM/g; s/jammy/JOAMMY/g'
log
```

```bash
sed -n 's/vim/VIM/p; s/jammy/JOAMMY/p'
log
```

 Первые 10 строк файла
```bash
sed -n '1,10s/vim/VIM/p; 1,10s/jammy/JOAMMY/p'
log
```

Удаление пустых строк
```bash
sed '/^$/=' log
```

Заменить в последних строках `vim` на `(vim)`:
```bash
sed '500,$s/vim/(&)/'  <  log
```

Пятерки букв инвертируются:
```bash
sed 's/\([a-z]\)\([a-z]\)\([a-z]\)\(
[a-z]\)\([a-z]\)/\5\4\3\2\1/g' < log
```

STDOUT в переменную:
```shell
administrator@u22srv:~$ my_var=$(find -L . -t
ype f -perm -a=x)
administrator@u22srv:~$ ls -la $my_var
-rwxrwxr-x 1 administrator administrator 128
окт 31 19:51 ./myLab/time_as_lines.bash
```

#### консультация 18.11.2023: Bash
[[Лавлинский Николай]]
- [ ] составить конспект по записям в тетради

##### bash - встроенные слова и команыд
- [ ] Чем `shell keyword` отличается от `shell builtin` в этом контексте:
```bash
$ type -a [[
[[ is a shell keyword
$ type -a test
test is a shell builtin
test is /usr/bin/test
test is /bin/test
$ type -a [
[ is a shell builtin
[ is /usr/bin/[
[ is /bin/[
```
?
bash
- ключевые слова
	- интерпретатор опознает действия, требуемые пользователем
	- `if`, `for`, `[[`
- встроенная команда
	- готовая подпрограмма
	- `test`

##### bash - не единственная оболочка ОС
управлять ОС можно из другой оболочки, например `sh`
```sh
administrator@u22srv:~/mylab$ ls -la /bin/sh
lrwxrwxrwx 1 root root 4 мар 23  2022 /bin/sh -> dash
administrator@u22srv:~/mylab$ /bin/sh
$
$ #это строка оболочки sh
$ ls | grep srv
srv.txt
$ exit
administrator@u22srv:~/mylab$ # это строка обочки bash
administrator@u22srv:~/mylab$
```

>Мне пришлось учиться `bash`, очень долгое время я обходился без него.


##### perl в администрировании Linux
>`perl` 
>- на порядок изощреннее `bash`
>- очень стабилен
>- в консоли можно сделать все что угодно
>- миллион модулей
>- работает с чем угодно
>- швейцарский нож
> 
>Недостатки
>- теряет популярность
>- учебных курсов мало, но есть
>- не рекомендует учить его - комьюнити мало

>`Python` рожден для Unix

##### гипервизор для Linux
>`KVM` 
>- хорош, стабилен, встроен в ядро Linux
>- не плохо для сервера виртуализации
>- есть графический Virtual Manager

##### надежность
>достигается благодаря 
>- кластеру, 
>- распределенной системе, 
>- дублированию
>- обязательному обновлению ядра

##### ![[Linux и деньги]]