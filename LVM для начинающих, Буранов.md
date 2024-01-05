---
type: course
aliases:
  - $ LVM для начинающих, Буранов
status: todo
recommendedby: 
title: '"LVM для начинающих, Буранов"'
---
___ 
tags:: 
prev:: [[videos|назад в библиотеку]] 
category:: 
url:: 
children:: 
___ 

[LVM для начинающих // Демо-занятие курса «Administrator Linux» - YouTube](<iframe title="LVM для начинающих // Демо-занятие курса «Administrator Linux»" src="https://www.youtube.com/embed/QFMnZsl-GOM?feature=oembed" height="150" width="200" style="aspect-ratio: 1.33333 / 1; width: 100%; height: 100%;" allowfullscreen="" allow="fullscreen"></iframe>)

Лекция Буранова
[LVM для начинающих // Демо-занятие курса «Administrator Linux» - YouTube](https://www.youtube.com/watch?v=QFMnZsl-GOM)
В Linux можно обойтись без разделов диска, а работать с целым диском - это просто файл типа `b`. В Windows обязательны разделы диска.

==Если можно обойтись без разделов, то это лучше== - избегаем лишнего слоя абстракции.

если диск логический, то устройство имеет цифру в названии `sda1`, а вот диск без разбиения цифр в названии не имеет `sdb`.

`LVM` - это прослойка между файловой системой и дисками. 
`LVM` состоит из трех абстракций
- `pl`
	- это блочное устройство, на которое программный комплекс LVM записал метаданные для **присваивание** этого устройства, подчинение устройства себе
		- ? LVM - owner блочного устройства?
		- 
- `vg`
	- с физическим устройством непосредственно LVM **не может**[^1], обязательно нужно устройство включить в состав *группы*
	- аналогия: *volume group - это свободное, неразмеченное пространство на hdd*
		- `vg` заменяет собой 1-ый уровень шестиэтажки - `physical drive`, но
			- с `vg` как таковым мы работать не можем - нужно выделять на этом свободном пространстве логические тома
			- с той лишь разницей, что со вторым этажом - разделами, logical drives мы работать можем напрямую 
			- `vg` - это набор `экстентов`, кусочков в 4Мб, которые можно *аллоцировать* в `lv`
- `lv`
	- аналогия: `lv` - это логические диски - 2-ой уровень шестиэтажки, разделы hdd
	- на нем можно создать файловую систему[^2]

Получаем многоэтажку в 6 этажей [[Общий слой работы с блочными устройствами]]:
![](https://pittbits.files.wordpress.com/2017/01/lvm.png)


>[22:56](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=1376.292398)Последний раз я работал с Windows лет 12 назад

[24:58](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=1498.3817339370576) LVM совсем не уменьшает скорость. Влияние не замечено вообще.

[25:41](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=1541.852734087738) Лучше ставить ОС на LVM, а не потом переносить ОС на LVM.

- [ ] Поискать отдельный вебинар про MBR и производительность

```bash
$ sudo fdisk -l /dev/sdb
...
Device     Boot   Start      End Sectors  Size Id Type
/dev/sdb1          2048  1026047 1024000  500M 83 Linux
/dev/sdb2       1026048  2050047 1024000  500M 82 Linux swap / Solaris
/dev/sdb3       2050048  2664447  614400  300M 83 Linux
/dev/sdb4       2664448 10485759 7821312  3,7G  5 Extended
/dev/sdb5       2666496  3280895  614400  300M  c W95 FAT32 (LBA)
/dev/sdb6       3282944  4102143  819200  400M 8e Linux LVM

```

 Узнать список `pv` - short:
 ```bash
 $ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <13,25g  <3,25g
  /dev/sdb6  myvg0     lvm2 a--  396,00m 196,00m

```

[30:42](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=1842.455377)
Сделать устройство частью `pv`
```bash
$ sudo pvcreate /dev/sdb1
WARNING: ext4 signature detected on /dev/sdb1 at offset 1080. Wipe it? [y/n]: y
# обнаружена особенная файловая система устройства, удаляем?
  Wiping ext4 signature on /dev/sdb1.
  Physical volume "/dev/sdb1" successfully created.

$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <13,25g  <3,25g
  /dev/sdb1            lvm2 ---  500,00m 500,00m
  /dev/sdb6  myvg0     lvm2 a--  396,00m 196,00m
# sdb1 не входит ни в одну vg

$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  myvg0       1   1   0 wz--n- 396,00m 196,00m
  ubuntu-vg   1   1   0 wz--n- <13,25g  <3,25g
```

[34:31](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2071.491105)
>LVM и soft-raid - это разные вещи, хотя LVM и позволяет делать некоторые вещи типа raid-массива. Обычно, если мы хотим защитить данные мы создаем raid массив, и этот массив вводим в LVM.


[35:28](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2128.567996)
`vg` создаются посредством *отталкивания* от наличных `pv`:
```bash
$ sudo vgcreate myvg1 /dev/sdb1
  Volume group "myvg1" successfully created

$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <13,25g  <3,25g
  /dev/sdb1  myvg1     lvm2 a--  496,00m 496,00m
  /dev/sdb6  myvg0     lvm2 a--  396,00m 196,00m
```
 объем `pv /dev/sdb1` изменился с 500,00m до 496,00m - это запись с метаинформацией LVM о вхождении в `vg` заняла *экстент*, единицу объема LVM.

[36:47](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2207.360038)
Теперь, отталкиваясь от наличия `vg` можно создавать `lv` - сущности-пространства пользовательских имен.

```bash
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  myvg0       1   1   0 wz--n- 396,00m 196,00m
  myvg1       1   0   0 wz--n- 496,00m 496,00m
  ubuntu-vg   1   1   0 wz--n- <13,25g  <3,25g

$ sudo lvcreate myvg1 -n books -L 300M
  Logical volume "books" created.

$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <13,25g  <3,25g
  /dev/sdb1  myvg1     lvm2 a--  496,00m 196,00m
  /dev/sdb6  myvg0     lvm2 a--  396,00m 196,00m

```

[38:44](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2324.330758)
Свободный размер `/dev/sdb1` уменьшился с 496,00m до 196,00m - LVM ничего не знает о файловых системах устройств, его задача, чтобы созданный `lv` работал вне зависимости от того, есть там файловая система или нет - *у меня есть собственное пространство экстентов, за которым я и слежу*.

Создано устройство с пользовательским именем `/dev/mapper/myvg1-books`
```bash
$ ls /dev/mapper/myvg1* -l
lrwxrwxrwx 1 root root 7 ноя 22 02:46 /dev/mapper/myvg1-books -> ../dm-2
$ ls -l /dev/dm-2
brw-rw---- 1 root disk 253, 2 ноя 22 02:46 /dev/dm-2
```

[40:19](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2419.932249)
>Девайс мэппер - часть ОС Linux, которая отвечает за создание новых устройств в каталоге `/dev`


> [!NOTE] Слово `mapper` 
> в имени `/dev/mapper/myvg1-books` указывает на **разметку** - т.е. на то, что здесь имеет место **система координат**, по другому **пространство имен** пользователя - 
> 
> здесь* ОС оборотилась лицом к пользователю*, а задом к устройствам `../dm-2`

[40:50](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2450.981099)
В пространстве имен пользователя созданы и записи `vg`:
```bash
$ ls -l /dev/myvg0
total 0
lrwxrwxrwx 1 root root 7 ноя 21 08:41 music -> ../dm-1
administrator@u22srv:~$ ls -l /dev/myvg1
total 0
lrwxrwxrwx 1 root root 7 ноя 22 02:46 books -> ../dm-2

```

[39:09](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2349.801634)
```bash
$ sudo lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  music     myvg0     -wi-ao---- 200,00m
  books     myvg1     -wi-a----- 300,00m
  ubuntu-lv ubuntu-vg -wi-ao----  10,00g
```
Здесь нет даже понятия свободного объема - здесь сфера LVS, сфера экстентов, тогда как *свободный объем*- сфера имен пользователя. Вот только сейчас можно создавать систему координат для сущностей пользователя - особенную файловую систему блочного устройства.

```bash
$ sudo mkfs -t ext4 /dev/mapper/myvg1-books
[sudo] password for administrator:
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 76800 4k blocks and 76800 inodes
Filesystem UUID: 357e84df-4717-463d-a5ef-495695d7da56
Superblock backups stored on blocks:
        32768

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

```

[41:12](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2472.164838)
Наконец, ее можно подмонтировать к какому-либо каталогу
```bash
$ sudo mkdir /mnt/mybooks
$ sudo mount /dev/mapper/myvg1-books /mnt/mybooks

# df (аббревиатура от disk free) — утилита в UNIX и UNIX-подобных системах, показывает список всех файловых систем по именам устройств, сообщает их размер, занятое и свободное пространство и точки монтирования
~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              197M  1,1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  6,8G  2,5G  74% /
tmpfs                              983M     0  983M   0% /dev/shm
tmpfs                              5,0M     0  5,0M   0% /run/lock
/dev/sda2                          1,7G  251M  1,4G  16% /boot
/dev/mapper/myvg0-music            184M   24K  175M   1% /mnt/mymusic
tmpfs                              197M  4,0K  197M   1% /run/user/1000
/dev/mapper/myvg1-books            265M   24K  244M   1% /mnt/mybooks

```


[41:29](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2489.873026)
Из 300,00m пространства на диске `lv books` доступно 265M ведь файловая система ext4 - это тоже данные, записанные на `lv books`, которые занимают место.

```bash
~$ ls -l /mnt/mybooks/
total 16
drwx------ 2 root root 16384 ноя 22 03:19 lost+found

```

[41:37](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2497.617384)
в этом каталоге появилась папка `lost+found`, поэтому мы теперь туда можем что-либо записывать, чтобы убедиться, что у нас все работает.

```bash
$ sudo cp -r /var/log/* /mnt/mybooks/
$ ls -l /mnt/mybooks/
total 4248
-rw-r--r-- 1 root root  13850 ноя 22 03:41 alternatives.log
...
$ ls -l /mnt/mybooks/ | wc -l
42

# скопированное заняло 29% места lv, 70 мегабайт
$ df -h | grep books
/dev/mapper/myvg1-books            265M   70M  175M  29% /mnt/mybooks

```


[43:05](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2585.28947)
>? В какой последовательности занимаются `pv`, объединенные одной `vg`?
>! Это определяется настройками.


[46:40](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2800.854942)
создадим еще несколько логических томов
```bash
$ sudo lvcreate myvg1 -n mysoft -L 100M
  Logical volume "mysoft" created.

$ sudo lvcreate myvg1 -n myinbox -L 100M
  Volume group "myvg1" has insufficient free space (24 extents): 25 required.
# закончились свободные экстенты...
$ sudo lvcreate myvg1 -n myinbox -L 96M
  Logical volume "myinbox" created.

# этот vg заполнене полностью
$ sudo vgs | grep myvg1
  myvg1       1   3   0 wz--n- 496,00m      0

```

и файловые системы на них:
```bash 
$ ls -l /dev/mapper/myvg1*
lrwxrwxrwx 1 root root 7 ноя 22 03:19 /dev/mapper/myvg1-books -> ../dm-2
lrwxrwxrwx 1 root root 7 ноя 22 03:56 /dev/mapper/myvg1-myinbox -> ../dm-4
lrwxrwxrwx 1 root root 7 ноя 22 03:55 /dev/mapper/myvg1-mysoft -> ../dm-3

$ sudo mkfs -t ext4 /dev/mapper/myvg1-myinbox
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 24576 4k blocks and 24576 inodes
...

$ sudo mkfs -t ext4 /dev/mapper/myvg1-mysoft
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 25600 4k blocks and 25600 inodes
...

# монтируем блочне устройства lv в каталоги имен
$ sudo mkdir /mnt/mysoft
$ sudo mkdir /mnt/myinbox
$ sudo mount /dev/mapper/myvg1-mysoft /mnt/mysoft/
$ sudo mount /dev/mapper/myvg1-myinbox /mnt/myinbox/
```

запишем что-либо в эти каталоги
```bash
# создадим файлы и заполним их нулями
sudo dd if=/dev/zero of=/mnt/myinbox/tmp bs=1M count=83
83+0 records in
83+0 records out
87031808 bytes (87 MB, 83 MiB) copied, 0,785356 s, 111 MB/s
$ sudo dd if=/dev/zero of=/mnt/mysoft/tmp bs=1M count=86
...
$ sudo dd if=/dev/zero of=/mnt/mybooks/tmp bs=1M count=230

# пространство каталогов заполнилось на
$ df -h | grep my
/dev/mapper/myvg0-music            184M   24K  175M   1% /mnt/mymusic
/dev/mapper/myvg1-books            265M  231M   14M  95% /mnt/mybooks
/dev/mapper/myvg1-mysoft            90M   87M     0 100% /mnt/mysoft
/dev/mapper/myvg1-myinbox           86M   84M     0 100% /mnt/myinbox

```

[49:18](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=2958.867272)
Пусть у нас выявлено не занятое дисковое пространство`sdb2`[[найти свободное пространство на диске]], позволяющее расширить объем пространства группы логических дисков `myvg1`, в которой заканчивается место. Это можно сделать длинной последовательностью действий, выполненной выше, а можно одной командой:
```bash
$ sudo vgextend myvg1 /dev/sdb2
WARNING: swap signature detected on /dev/sdb2 at offset 4086. Wipe it? [y/n]: y
  Wiping swap signature on /dev/sdb2.
  Physical volume "/dev/sdb2" successfully created.
  Volume group "myvg1" successfully extended
```

> [!error] раздел `sdb2` здесь не форматируется
> - [ ] Природа ошибки не понята
> 
> Пришлось
> - добавить в `vg` новый раздел
> - удалить `sdb2` из из `vg` [[удалить pv из vg]]
> - переформатировать `sdb2` в утилитой `fdisk` в стандартный раздел Linux

Заход № 2 после исправления ошибки:

```bash 
$ sudo vgextend myvg1 /dev/sdb2
  Volume group "myvg1" successfully extended

# Также был добавлен sdb7
$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <13,25g  <3,25g
  /dev/sdb1  myvg1     lvm2 a--  496,00m      0
  /dev/sdb2  myvg1     lvm2 a--  496,00m 496,00m
  /dev/sdb6  myvg0     lvm2 a--  396,00m 196,00m
  /dev/sdb7  myvg1     lvm2 a--   <1,17g 700,00m

# в myvg1 теперь три диска (раздела дисков)
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  myvg0       1   1   0 wz--n- 396,00m 196,00m
  myvg1       3   4   0 wz--n-  <2,14g  <1,17g
  ubuntu-vg   1   1   0 wz--n- <13,25g  <3,25g

$ sudo lvs
[sudo] password for administrator:
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  music     myvg0     -wi-ao---- 200,00m
  books     myvg1     -wi-ao---- 300,00m
  mybckp    myvg1     -wi-a----- 496,00m
  myinbox   myvg1     -wi-ao----  96,00m
  mysoft    myvg1     -wi-ao---- 100,00m
  ubuntu-lv ubuntu-vg -wi-ao----  10,00g

```

тот же результат может быть достигнут командой `vgcreate`. Можно создать `lv` объемом 496M

[50:58](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=3058.121373)
>Это не raid, 
>1. создавать программный raid можно  в момент создания `lv`, а не `vg`.
>2. такое редко используется на серверах, обычно предпочитают специальные программы, в частности 
>	- программный `MDA`(?) или
>	- raid контроллер
>3. но и с помощью LVM можно сделать дубликацию данных, если это нас устроит

- [ ] можно ли скормить VirtualBox-у физический диск целиком, с тем, чтобы использовать Ubuntu как файловый сервер с LVM зеркалированием?

[53:23](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=3203.541886)
**здесь**
- [ ] ? Проблема что содержится в устройстве `/dev/mapper/myvg1-mybckp -> ../dm-5`
- [ ] ? Как отформатировано устройство `/dev/mapper/myvg1-mybckp -> ../dm-5`
- [ ] ? Проблема что содержится в устройстве `/dev/sdb2`
- [ ] ? Как отформатировано устройство `/dev/sdb2`
`
> [!Tip] идея дидактическая
> перевести это занятие на язык диаграмм, для того, чтобы
> - видеть конечный результат, цель
> - видеть текущее место настройки
> - видеть логическое пространство - сущности и их атрибуты текущей фразы лектора
> 
> кажется сами лекторы путаются и путают

![[прочитать суперблок файловой системы]]

```bash
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  myvg0       1   1   0 wz--n- 396,00m 196,00m

# на myvg1 есть 1.17g свободного пространства
  myvg1       3   4   0 wz--n-  <2,14g  <1,17g
  ubuntu-vg   1   1   0 wz--n- <13,25g  <3,25g

$  df -h | grep my
/dev/mapper/myvg0-music            184M   24K  175M   1% /mnt/mymusic
# и я хочу добавить 300M  в lv books
/dev/mapper/myvg1-books            265M  231M   14M  95% /mnt/mybooks
...

$ sudo lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
...
  books     myvg1     -wi-ao---- 300,00m
...

$ sudo lvextend /dev/mapper/myvg1-books -L +300M
  Size of logical volume myvg1/books changed from 300,00 MiB (75 extents) to 600,00 MiB (150 extents).
  Logical volume myvg1/books successfully resized.

# размер увеличен
$ sudo lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
...
  books     myvg1     -wi-ao---- 600,00m
...

# однако файловая система еще не знает об увеличении lv
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
...
/dev/mapper/myvg1-books            265M  231M   14M  95% /mnt/mybooks
...

# сообщаем файловой системе об изменении
$ sudo resize2fs /dev/mapper/myvg1-books
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/myvg1-books is mounted on /mnt/mybooks; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/myvg1-books is now 153600 (4k) blocks long.

# файловая система внесла изменения
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
...
/dev/mapper/myvg1-books            553M  231M  295M  44% /mnt/mybooks

# данные на месте
$ ls -l /mnt/mybooks
total 235520
-rw-r--r-- 1 root root 241172480 ноя 22 04:44 tmp
```

Проведем проверку сохранности данных на `lv` так - отмонтируем каталоги, и затем заново их примонтируем
```bash
$ sudo umount /mnt/mybooks/
$ sudo umount /mnt/mysoft
$ sudo umount /mnt/myinbox
$ sudo rm -r /mnt/mybooks/
$ sudo rm -r /mnt/mymusic/
rm: cannot remove '/mnt/mymusic/': Device or resource busy
$ sudo rm -r /mnt/myinbox/
$ sudo rm -r /mnt/mysoft/
$ sudo mkdir /mnt/inbox
$ sudo mkdir /mnt/books
$ sudo mkdir /mnt/soft
$ sudo mount /dev/mapper/myvg1-books /mnt/books
$ sudo mount /dev/mapper/myvg1-mysoft /mnt/soft
$ sudo mount /dev/mapper/myvg1-myinbox /mnt/inbox

# на ls-ах данные сохранились
$ ls -l /mnt/books/
total 235520
-rw-r--r-- 1 root root 241172480 ноя 22 04:44 tmp
$ ls -l /mnt/soft/
total 88064
-rw-r--r-- 1 root root 90177536 ноя 22 04:43 tmp
$ ls -l /mnt/inbox/
total 84996
-rw-r--r-- 1 root root 87031808 ноя 22 04:42 tmp

# т.о. lv был расширен, хотя он был создан первым 
# в ряду других lv, 
# такая операция была бы невозможна для разделов hdd
```

Т.е. порядок распределения экстентов на `lv` решает LVS, это не наша задача.


[01:05:49](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=3949.125384)
>В реальности на серверах не принято *аллоцировать* все свободное пространство, обычно создается необходимый минимум
>- root 4G
>- var 4G
>- остальное в неразмеченном пространстве
>
>потому что добавить - легко.

[01:06:37](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=3997.272618)
Сложнее убрать. ext4 не умеет уменьшаться на ходу. Чтобы уменьшить ext4 нужно отключить `lv`, отмонтировать его.
Способность уменьшаться или увеличиваться - это свойство файловой системы, которое не меняется от носителя - hdd, raid, LVM или еще что-то.
Вот xfs из мира RH вообще не имеет уменьшаться никоим образом.
NTFS из Windows вообще умеет всё - диск C можно уменьшить на ходу.

[01:09:50](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=4190.664415)
С помощью всего лишь одной команды увеличим размер `/dev/mapper/myvg1-mysoft` на 100M:
```bash
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/myvg1-mysoft            90M   87M     0 100% /mnt/soft

$ sudo lvextend /dev/mapper/myvg1-mysoft -L +100M -r
# ключ -r - ресайз: LVM сообщает файловой системе о
# изменении пространства lv
  Size of logical volume myvg1/mysoft changed from 100,00 MiB (25 extents) to 200,00 MiB (50 extents).
  Logical volume myvg1/mysoft successfully resized.
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/myvg1-mysoft is mounted on /mnt/soft; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/myvg1-mysoft is now 51200 (4k) blocks long.

$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/myvg1-mysoft           184M   87M   89M  50% /mnt/soft

```
 
Как уменьшить размер файловой системы?
```bash
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/myvg1-books            553M  231M  289M  45% /mnt/books

$ sudo resize2fs /dev/mapper/myvg1-books 450M
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/myvg1-books is mounted on /mnt/books; on-line resizing required
resize2fs: On-line shrinking not supported
# т.е. уменьшиться размер на лету файловая система ext4
# не способна, сначала нужно отмонтировать lv
# что повлечет за собой простой приложения
$ sudo resize2fs /dev/mapper/myvg1-books 450M
resize2fs 1.46.5 (30-Dec-2021)
Please run 'e2fsck -f /dev/mapper/myvg1-books' first.
# сначала нужно сделать проверку этой файловой системы

$ sudo e2fsck -f /dev/mapper/myvg1-books
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
/lost+found not found.  Create<y>? yes
Pass 4: Checking reference counts
Pass 5: Checking group summary information

/dev/mapper/myvg1-books: ***** FILE SYSTEM WAS MODIFIED *****
/dev/mapper/myvg1-books: 12/128000 files (0.0% non-contiguous), 71106/153600 blocks

# и вот только теперь можно сделать resize
$ sudo resize2fs /dev/mapper/myvg1-books 450M
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/mapper/myvg1-books to 115200 (4k) blocks.
The filesystem on /dev/mapper/myvg1-books is now 115200 (4k) blocks long.

# после этого можно спокойно делать mount
$ sudo mount /dev/mapper/myvg1-books /mnt/books/
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
# файловая система уменьшилась
/dev/mapper/myvg1-books            409M  231M  154M  60% /mnt/books

$ ls -l /mnt/books/
total 235524
drwx------ 2 root root      4096 ноя 23 03:11 lost+found
-rw-r--r-- 1 root root 241172480 ноя 22 04:44 tmp

```

[01:13:17](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=4397.629667)
>В реальности, если мы уменьшаем реальную файловую систему, которая раскидала данные по всему диску, то как правило что-то теряется. Обязательно. Root-овый раздел перестает загружаться.
>
>По факт это действие требует бэкапа.


[01:14:49](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=4489.187122)
```bash
$ sudo lvreduce /dev/myvg1/books -L 100M
# если бы делать это без размонтирования,то у нас бы
# после этой операции точно были бы потерянные данные
  WARNING: Reducing active and open logical volume to 100,00 MiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce myvg1/books? [y/n]: y
  Size of logical volume myvg1/books changed from 600,00 MiB (150 extents) to 100,00 MiB (25 extents).
  Logical volume myvg1/books successfully resized.

$ sudo umount /mnt/books
$ sudo mount /dev/mapper/myvg1-books /mnt/books/
mount: /mnt/books: wrong fs type, bad option, bad superblock on /dev/mapper/myvg1-books, missing codepage or helper program, or other error.
# увы, lv не смонтировался (в отличие от Буранова)
```

[01:15:47](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=4547.134514)
> Люди по незнанию строгают множество разделов на дисках. Новички из оставшегося пустого пространства hdd создают еще один раздел и добаляют его как `pv` в `vg`. Этого делать не надо.

Как увеличить размер `pv` sdb7?
```bash
$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  ...
  /dev/sdb7  myvg1     lvm2 a--   <1,17g 600,00m

# если мы увеличим размер раздела sdb7 pv не увелится...
$ sudo fdisk -l /dev/sdb
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
...
Device     Boot   Start      End Sectors  Size Id Type
...
/dev/sdb4       2664448 10485759 7821312  3,7G  5 Extended
/dev/sdb5       2666496  3280895  614400  300M  c W95 FAT32 (LBA)
/dev/sdb6       3282944  4102143  819200  400M 8e Linux LVM
/dev/sdb7       4104192  6561791 2457600  1,2G 83 Linux
```
Пусть нам нужно добавить в `vg` 1G. Идем в раздел sdb7, удаляем его и создаем его заново.

```bash
$ sudo fdisk /dev/sdb
...
# нужно обязательно сохранить начало раздела /dev/sdb7       4104192
# а вот то, где он заканчивается (6561791) нам нужно изменить 

Command (m for help): d
Partition number (1-7, default 7):

Partition 7 has been deleted.

Command (m for help): n
All primary partitions are in use.
Adding logical partition 7
First sector (4104192-10485759, default 4104192):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4104192-10485759, default 10485759): +2200M
# Размер задали в 2.2G

Created a new partition 7 of type 'Linux' and of size 2,1 GiB.

Command (m for help): p
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcbdf2cd0

Device     Boot   Start      End Sectors  Size Id Type
/dev/sdb1          2048  1026047 1024000  500M 83 Linux
/dev/sdb2       1026048  2050047 1024000  500M 83 Linux
/dev/sdb3       2050048  2664447  614400  300M 83 Linux
/dev/sdb4       2664448 10485759 7821312  3,7G  5 Extended
/dev/sdb5       2666496  3280895  614400  300M  c W95 FAT32 (LBA)
/dev/sdb6       3282944  4102143  819200  400M 8e Linux LVM
/dev/sdb7       4104192  8609791 4505600  2,1G 83 Linux
# начало раздела /dev/sdb7 тоже самое: 4104192
# конец раздела изменился: уже не 6561791

Command (m for help): w
The partition table has been altered.
Syncing disks.

```

[01:22:01](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=4921.40235)
В некоторых случаях (не в нашем) здесь может возникать ошибка `partition table failed`
```bash
$ cat /proc/partitions
major minor  #blocks  name
# размеры в блоках
   7        0      64972 loop0
   7        1     114636 loop1
   7        2      64988 loop2
   7        3      54536 loop3
   7        4      41836 loop4
  11        0    1048575 sr0
   8        0   15728640 sda
   8        1       1024 sda1
   8        2    1835008 sda2
   8        3   13890560 sda3
   8       16    5242880 sdb
   8       17     512000 sdb1
   8       18     512000 sdb2
   8       19     307200 sdb3
   8       20          0 sdb4
   8       21     307200 sdb5
   8       22     409600 sdb6
# здесь размер раздела в секторах в случае ошибки будет в два раза больше
   8       23    2252800 sdb7
 253        0   10485760 dm-0
 253        1     204800 dm-1
 253        2     102400 dm-2
 253        3     204800 dm-3
 253        4      98304 dm-4
 253        5     507904 dm-5

# таблица разделов ОС не перечиталась, 
# потому что устройство занято. Перечитать ее можно командой:
$ cat /proc/partitions
```

[01:23:34](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=5014.03859)
LVM еще не знает об изменении размера раздела:
```bash
$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sdb7  myvg1     lvm2 a--   <1,17g 600,00m

# сообщаем LVM об изменениях
$ sudo pvresize /dev/sdb7
  Physical volume "/dev/sdb7" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized

$ sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sdb7  myvg1     lvm2 a--    2,14g   1,56g

$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  myvg1       3   4   0 wz--n-   3,11g   2,24g
```
>Т.о. будучи в здравом уме вы не будете создавать новый раздел, чтобы добавить его в `vg`, вы будете изменять размер раздела. Лучше всего уменьшать количество `lv`

[01:26:23](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=5183.446659982834)
Эксперимент с ошибочным изъятием одного физического диска из двух.

Выводы:
1. [01:30:03](https://www.youtube.com/watch?v=QFMnZsl-GOM#t=5403.152236) ==Если мы используем диск как единое пространство, то LVM нем не нужен==. Если планируем сервер, где не понятно, сколько разделов у нас будет, то здесь LVM - спасение.
2. Уменьшить раздел сложнее, хотя есть некоторые файловые системы, которые это умеют.
3. В домашнем использовании Linux LVM, наверное лишний.


[^1]: В чем логика *не мощи*? Почему не может?
[^2]: *Файловая система логического диска* - это особенное во всеобщем
	- всеобщее - ОС Linux как древообразное пространство имен - *все есть файл*
	- особенное - fat32 файловая система логического диска
	
	древообразность имен вообще и древообразность имен конкретная
[^3]: Кажется, это хорошее указание на то, как затереть данные на диске быстро.