*смотрим какие есть диски:*

```
cou@dz2:~$ lsblk 
sda      8:0    0   25G  0 disk 
├─sda1   8:1    0    1M  0 part 
└─sda2   8:2    0   25G  0 part /
sdb      8:16   0   25G  0 disk 
sdc      8:32   0   25G  0 disk 
sdd      8:48   0   25G  0 disk 
sde      8:64   0   25G  0 disk 
sr0     11:0    1 1024M  1 rom  
```
*трем суперблоки, создаем массив:*

```
cou@dz2:~$ mdadm --zero-superblock /dev/sd{b,c,d,e}
cou@dz2:~$ sudo mdadm --create /dev/md10 -l 10 -n 4 /dev/sd{b,c,d,e}
```

*смотрим результат:*

```
cou@dz2:~$ lsblk 
sda      8:0    0   25G  0 disk   
├─sda1   8:1    0    1M  0 part   
└─sda2   8:2    0   25G  0 part   /
sdb      8:16   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sdc      8:32   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sdd      8:48   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sde      8:64   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sr0     11:0    1 1024M  1 rom    

cou@dz2:~$ cat /proc/mdstat 
Personalities : [raid4] [raid5] [raid6] [raid10] 
md10 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      52393984 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
      [==========>..........]  resync = 51.2% (26843648/52393984) finish=2.1min speed=200556K/sec
      
unused devices: <none>

cou@dz2:~$ sudo mdadm -D /dev/md10 
/dev/md10:
           Version : 1.2
     Creation Time : Wed Oct 14 10:56:44 2026
        Raid Level : raid10
        Array Size : 52393984 (49.97 GiB 53.65 GB)
     Used Dev Size : 26196992 (24.98 GiB 26.83 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Oct 14 11:01:06 2026
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : dz2:10  (local to host dz2)
              UUID : dd41d5b1:ef231aa0:ccd073b8:2feb3bdc
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```
*фейлим один диск:*

```
cou@dz2:~$ mdadm /dev/md10 --fail /dev/sdc
cou@dz2:~$ sudo mdadm -D /dev/md10 

/dev/md10:
           Version : 1.2
     Creation Time : Wed Oct 14 10:56:44 2026
        Raid Level : raid10
        Array Size : 52393984 (49.97 GiB 53.65 GB)
     Used Dev Size : 26196992 (24.98 GiB 26.83 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Oct 14 11:03:47 2026
             State : clean, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : dz2:10  (local to host dz2)
              UUID : dd41d5b1:ef231aa0:ccd073b8:2feb3bdc
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       -       0        0        1      removed
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       1       8       32        -      faulty   /dev/sdc
       
```
*убираем диск из массива:*

```
cou@dz2:~$ sudo mdadm /dev/md10 --remove /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md10

cou@dz2:~$ lsblk
sda      8:0    0   25G  0 disk   
├─sda1   8:1    0    1M  0 part   
└─sda2   8:2    0   25G  0 part   /
sdb      8:16   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sdc      8:32   0   25G  0 disk   
sdd      8:48   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sde      8:64   0   25G  0 disk   
└─md10   9:10   0   50G  0 raid10 
sr0     11:0    1 1024M  1 rom    
```

*"чиним" диск, подключаем новый/исправленый:*

```
cou@dz2:~$ sudo mdadm --zero-superblock /dev/sdc

cou@dz2:~$ sudo mdadm /dev/md10 --add /dev/sdc
mdadm: added /dev/sdc

cou@dz2:~$ sudo mdadm -D /dev/md10 
/dev/md10:
           Version : 1.2
     Creation Time : Wed Oct 14 10:56:44 2026
        Raid Level : raid10
        Array Size : 52393984 (49.97 GiB 53.65 GB)
     Used Dev Size : 26196992 (24.98 GiB 26.83 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Oct 14 11:08:56 2026
             State : clean, degraded, recovering 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 5% complete

              Name : dz2:10  (local to host dz2)
              UUID : dd41d5b1:ef231aa0:ccd073b8:2feb3bdc
            Events : 22

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       4       8       32        1      spare rebuilding   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```

*ждем завершения восстановления.*
```
cou@dz2:~$ sudo mdadm -D /dev/md10 
/dev/md10:
           Version : 1.2
     Creation Time : Wed Oct 14 10:56:44 2026
        Raid Level : raid10
        Array Size : 52393984 (49.97 GiB 53.65 GB)
     Used Dev Size : 26196992 (24.98 GiB 26.83 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Oct 14 11:11:07 2026
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : dz2:10  (local to host dz2)
              UUID : dd41d5b1:ef231aa0:ccd073b8:2feb3bdc
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       4       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```
*делаю GPT таблицу и 5 разделов:*

```
cou@dz2:~$ sudo fdisk /dev/md10 

Command (m for help): g
Created a new GPT disklabel (GUID: 0C653D5C-19F9-425A-AC40-36517EF614A6).

Command (m for help): F

Unpartitioned space /dev/md10: 49.97 GiB, 53650374144 bytes, 104785887 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

Start       End   Sectors Size
 2048 104787934 104785887  50G

Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-104787934, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-104787934, default 104785919): +10G

Created a new partition 1 of type 'Linux filesystem' and of size 10 GiB.
....
Command (m for help): n
Partition number (5-128, default 5): 
First sector (83888128-104787934, default 83888128): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (83888128-104787934, default 104785919): 

Created a new partition 5 of type 'Linux filesystem' and of size 10 GiB.

Command (m for help): F
Unpartitioned space /dev/md10: 0 B, 0 bytes, 0 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

Command (m for help): p
Disk /dev/md10: 49.97 GiB, 53651439616 bytes, 104787968 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 524288 bytes / 1048576 bytes
Disklabel type: gpt
Disk identifier: 0C653D5C-19F9-425A-AC40-36517EF614A6

Device         Start       End  Sectors Size Type
/dev/md10p1     2048  20973567 20971520  10G Linux filesystem
/dev/md10p2 20973568  41945087 20971520  10G Linux filesystem
/dev/md10p3 41945088  62916607 20971520  10G Linux filesystem
/dev/md10p4 62916608  83888127 20971520  10G Linux filesystem
/dev/md10p5 83888128 104785919 20897792  10G Linux filesystem

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

*создаю файловую систему:*

```
cou@dz2:~$ sudo mkfs.ext4 /dev/md10p1
cou@dz2:~$ sudo mkfs.ext4 /dev/md10p2
cou@dz2:~$ sudo mkfs.ext4 /dev/md10p3
cou@dz2:~$ sudo mkfs.ext4 /dev/md10p4
cou@dz2:~$ sudo mkfs.ext4 /dev/md10p5
```

*папки для монтирования:*

```
cou@dz2:~$ mkdir -p /raid/part{1,2,3,4,5}

cou@dz2:~$ ls /raid/
part1  part2  part3  part4  part5
```

*монтирую:*

```
cou@dz2:~$ for i in $(seq 1 5); do sudo mount /dev/md10p$i /raid/part$i; done
```

*результат:*

```
cou@dz2:~$ lsblk
sda          8:0    0   25G  0 disk   
├─sda1       8:1    0    1M  0 part   
└─sda2       8:2    0   25G  0 part   /
sdb          8:16   0   25G  0 disk   
└─md10       9:10   0   50G  0 raid10 
  ├─md10p1 259:0    0   10G  0 part   /raid/part1
  ├─md10p2 259:1    0   10G  0 part   /raid/part2
  ├─md10p3 259:2    0   10G  0 part   /raid/part3
  ├─md10p4 259:3    0   10G  0 part   /raid/part4
  └─md10p5 259:4    0   10G  0 part   /raid/part5
sdc          8:32   0   25G  0 disk   
└─md10       9:10   0   50G  0 raid10 
  ├─md10p1 259:0    0   10G  0 part   /raid/part1
  ├─md10p2 259:1    0   10G  0 part   /raid/part2
  ├─md10p3 259:2    0   10G  0 part   /raid/part3
  ├─md10p4 259:3    0   10G  0 part   /raid/part4
  └─md10p5 259:4    0   10G  0 part   /raid/part5
sdd          8:48   0   25G  0 disk   
└─md10       9:10   0   50G  0 raid10 
  ├─md10p1 259:0    0   10G  0 part   /raid/part1
  ├─md10p2 259:1    0   10G  0 part   /raid/part2
  ├─md10p3 259:2    0   10G  0 part   /raid/part3
  ├─md10p4 259:3    0   10G  0 part   /raid/part4
  └─md10p5 259:4    0   10G  0 part   /raid/part5
sde          8:64   0   25G  0 disk   
└─md10       9:10   0   50G  0 raid10 
  ├─md10p1 259:0    0   10G  0 part   /raid/part1
  ├─md10p2 259:1    0   10G  0 part   /raid/part2
  ├─md10p3 259:2    0   10G  0 part   /raid/part3
  ├─md10p4 259:3    0   10G  0 part   /raid/part4
  └─md10p5 259:4    0   10G  0 part   /raid/part5
sr0         11:0    1 1024M  1 rom    
cou@dz2:~$ 
```

