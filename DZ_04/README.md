Домашнее задание 4 "работа с ZFS"

1. Определить алгоритм с наилучшим сжатием:
- Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
- создать 4 файловых системы на каждой применить свой алгоритм сжатия;
- для сжатия использовать либо текстовый файл, либо группу файлов.

2. Определить настройки пула:
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
- размер хранилища;
- тип pool;
- значение recordsize;
- какое сжатие используется;
- какая контрольная сумма используется.

3. Работа со снапшотами:
- скопировать файл из удаленной директории;
- восстановить файл локально. zfs receive;
- найти зашифрованное сообщение в файле secret_message.



подготовил виртуалку, система на sda, sdb-sdi 512M для выполнения заданий, установка zfsutils:
```
root@coudzzfs:~# lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   25G  0 disk 
├─sda1   8:1    0    1M  0 part 
├─sda2   8:2    0    2G  0 part /boot
└─sda3   8:3    0   10G  0 part /
sdb      8:16   0  512M  0 disk 
sdc      8:32   0  512M  0 disk 
sdd      8:48   0  512M  0 disk 
sde      8:64   0  512M  0 disk 
sdf      8:80   0  512M  0 disk 
sdg      8:96   0  512M  0 disk 
sdh      8:112  0  512M  0 disk 
sdi      8:128  0  512M  0 disk 
sr0     11:0    1 1024M  1 rom  
root@coudzzfs:~# sudo apt install zfsutils-linux 
```

1 Степень сжатия:
для тестирования методов зжатия готовим 4 пула по 2 диска:
```
root@coudzzfs:~# zpool create zfsdz1 mirror /dev/sdb /dev/sdc
root@coudzzfs:~# zpool create zfsdz2 mirror /dev/sdd /dev/sde
root@coudzzfs:~# zpool create zfsdz3 mirror /dev/sdf /dev/sdg
root@coudzzfs:~# zpool create zfsdz4 mirror /dev/sdh /dev/sdi
root@coudzzfs:~# zpool list 
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zfsdz1   480M   128K   480M        -         -     6%     0%  1.00x    ONLINE  -
zfsdz2   480M   135K   480M        -         -     5%     0%  1.00x    ONLINE  -
zfsdz3   480M   129K   480M        -         -     6%     0%  1.00x    ONLINE  -
zfsdz4   480M   162K   480M        -         -     5%     0%  1.00x    ONLINE  -
```

на каждом из пулов задаем свой метод зжатия:
```
root@coudzzfs:~# zfs set compression=lzjb zfsdz1
root@coudzzfs:~# zfs set compression=lz4 zfsdz2
root@coudzzfs:~# zfs set compression=gzip-9 zfsdz3
root@coudzzfs:~# zfs set compression=zle zfsdz4

root@coudzzfs:~# zfs get all | grep compression
zfsdz1  compression             lzjb                    local
zfsdz2  compression             lz4                     local
zfsdz3  compression             gzip-9                  local
zfsdz4  compression             zle                     local
```

на каждый скачиваем файл pg2600.converter.log из примера в методичке:
```
root@coudzzfs:~# ls -l /zfsdz*
/zfsdz1:
total 22133
-rw-r--r-- 1 root root 41273476 Oct  2 07:31 pg2600.converter.log

/zfsdz2:
total 18023
-rw-r--r-- 1 root root 41273476 Oct  2 07:31 pg2600.converter.log

/zfsdz3:
total 10973
-rw-r--r-- 1 root root 41273476 Oct  2 07:31 pg2600.converter.log

/zfsdz4:
total 40348
-rw-r--r-- 1 root root 41273476 Oct  2 07:31 pg2600.converter.log
```

смотрим занимаемое место и степень сжатия:
```
root@coudzzfs:~# zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
zfsdz1          21.8M   330M  21.6M  /zfsdz1
zfsdz2          17.7M   334M  17.6M  /zfsdz2
zfsdz3          10.9M   341M  10.7M  /zfsdz3
zfsdz4          39.5M   312M  39.4M  /zfsdz4

root@coudzzfs:~# zfs get all | grep compressratio | grep -v ref
zfsdz1           compressratio           1.82x                     -
zfsdz2           compressratio           2.23x                     -
zfsdz3           compressratio           3.66x                     -
zfsdz4           compressratio           1.00x                     -
```
zfsdz3 - gzip-9 лучше других жмет текстовые данные.

2 Определение настроек пула:
скачал и распаковал файл из примера
```
root@coudzzfs:~# ls
archive.tar.gz
root@coudzzfs:~# tar -xzvf archive.tar.gz 
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

смотрим есть ли там пул:
```
root@coudzzfs:~# zpool import -d zpoolexport/
  pool: otus
    id: 6554193320433390805
 state: ONLINE
status: Some supported features are not enabled on the pool.
	(Note that they may be intentionally disabled if the
	'compatibility' property is set.)
action: The pool can be imported using its name or numeric identifier, though
	some features will not be available without an explicit 'zpool upgrade'.
config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE
```

пул есть, импортируем, смотрим статус:
```
root@coudzzfs:~# zpool import -d zpoolexport/ otus
root@coudzzfs:~# zpool status 
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
config:

	NAME                         STATE     READ WRITE CKSUM
	otus                         ONLINE       0     0     0
	  mirror-0                   ONLINE       0     0     0
	    /root/zpoolexport/filea  ONLINE       0     0     0
	    /root/zpoolexport/fileb  ONLINE       0     0     0
errors: No known data errors
```

пробую выборочно смотреть параметры импортированой ФС:
```
root@coudzzfs:~# zfs get snapshot_count otus
NAME  PROPERTY        VALUE    SOURCE
otus  snapshot_count  none     default
root@coudzzfs:~# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
root@coudzzfs:~# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
root@coudzzfs:~# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
root@coudzzfs:~# zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
root@coudzzfs:~# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

3 работа со снапшотом:
качаю файл из примера
```
root@coudzzfs:~# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
```

воссатанавливаю ФС из файла:
```
root@coudzzfs:~# zfs receive otus/test@today < otus_task2.file
```

поиск и вывод содержимого файла "secret_message":
```
root@coudzzfs:/# ls -la /otus/test/
total 2586
drwxr-xr-x 3 root root      11 May 15  2020 .
drwxr-xr-x 4 root root       4 Oct 23 16:56 ..
-rw-r--r-- 1 root root       0 May 15  2020 10M.file
-rw-r--r-- 1 root root  309987 May 15  2020 Limbo.txt
-rw-r--r-- 1 root root  509836 May 15  2020 Moby_Dick.txt
-rw-r--r-- 1 root root 1209374 May  6  2016 War_and_Peace.txt
-rw-r--r-- 1 root root  727040 May 15  2020 cinderella.tar
-rw-r--r-- 1 root root      65 May 15  2020 for_examaple.txt
-rw-r--r-- 1 root root       0 May 15  2020 homework4.txt
drwxr-xr-x 3 cou  cou        4 Dec 18  2017 task1
-rw-r--r-- 1 root root  398635 May 15  2020 world.sql

root@coudzzfs:~# find /otus/test/ -name "secret_message"
/otus/test/task1/file_mess/secret_message

root@coudzzfs:/# cat /otus/test/task1/file_mess/secret_message 
https://otus.ru/lessons/linux-hl/
```
