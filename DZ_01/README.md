Занятие 1. Обновление ядра системы
Цель домашнего задания
Научиться обновлять ядро в ОС Linux.



cou@linadm01:~$ uname -a
Linux linadm01 7.0.0-38-generic #38-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep  4 09:10:14 UTC 2026 x86_64 GNU/Linux

cou@linadm01:~$ cd kernel/
cou@linadm01:~/kernel$ ls -lah

total 199M
drwxrwxr-x 2 cou cou 4.0K Oct  3 09:31 .
drwxr-x--- 5 cou cou 4.0K Oct  3 09:42 ..
-rw-rw-r-- 1 cou cou 4.0M Sep 14 15:39 linux-headers-7.2.6-070206-generic_7.2.6-070206.202609141300_amd64.deb
-rw-rw-r-- 1 cou cou  15M Sep 14 15:39 linux-headers-7.2.6-070206_7.2.6-070206.202609141300_all.deb
-rw-rw-r-- 1 cou cou  17M Sep 14 15:38 linux-image-unsigned-7.2.6-070206-generic_7.2.6-070206.202609141300_amd64.deb
-rw-rw-r-- 1 cou cou 164M Sep 14 15:38 linux-modules-7.2.6-070206-generic_7.2.6-070206.202609141300_amd64.deb

cou@linadm01:~/kernel$ sudo dpkg -i *
...
cou@linadm01:~/kernel$ sudo update-grub
...
cou@linadm01:~/kernel$ sudo grub-set-default 0
...
cou@linadm01:~$ uname -a
Linux linadm01 7.2.6-070206-generic #202609141300 SMP PREEMPT_DYNAMIC Mon Sep 14 15:22:52 UTC 2026 x86_64 GNU/Linux
