Домашнее задание "Работа с NFS"

- запустить 2 виртуальных машины (сервер NFS и клиента);
- на сервере NFS должна быть подготовлена и экспортирована директория; 
- в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё; 
- экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);
- монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.

---
сервер-192.168.43.177
клиент-192.168.43.242
на сервере устанавливаю НФС-сервер, в конфиге разрешаю версию 3 и UDP, проверяю порты:
```
root@nfss:~# apt install nfs-kernel-server
root@nfss:~# vi /etc/nfs.conf
root@nfss:~# systemctl restart nfs
root@nfss:~# ss -tnplu | grep -E '2049|111'
udp   UNCONN 0      0                                0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=2735,fd=5),("systemd",pid=1,fd=120))
udp   UNCONN 0      0                                0.0.0.0:2049       0.0.0.0:*                                                              
udp   UNCONN 0      0                                   [::]:111           [::]:*    users:(("rpcbind",pid=2735,fd=7),("systemd",pid=1,fd=123))
udp   UNCONN 0      0                                   [::]:2049          [::]:*                                                              
tcp   LISTEN 0      4096                             0.0.0.0:2049       0.0.0.0:*                                                              
tcp   LISTEN 0      4096                             0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=2735,fd=4),("systemd",pid=1,fd=119))
tcp   LISTEN 0      4096                                [::]:2049          [::]:*                                                              
tcp   LISTEN 0      4096                                [::]:111           [::]:*    users:(("rpcbind",pid=2735,fd=6),("systemd",pid=1,fd=121))
```

директория и права:
```
root@nfss:/# mkdir -p /srv/share/upload
root@nfss:/# chown -R nobody:nogroup /srv/share/
root@nfss:/# chmod 0777 /srv/share/upload/
```
добавил "/srv/share 192.168.43.242/32(rw,sync,root_squash)" в /etc/exports

```
root@nfss:/# exportfs -r
root@nfss:/# exportfs -s
/srv/share  192.168.43.242/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

```
root@nfsc:~# cat /etc/fstab 

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/c0139c1a-f790-4769-9231-39862d07431f / ext4 defaults 0 1
/swap.img	none	swap	sw	0	0
192.168.43.177:/srv/share/ /mnt nfs vers=3,noaute,x-systemd.automount 0 0

root@nfsc:~# systemctl  daemon-reload 
root@nfsc:~# systemctl  restart remote-fs.target 
root@nfsc:/mnt# mount | grep mnt

systemd-1 on /mnt type autofs (rw,relatime,fd=73,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=22940)
192.168.43.177:/srv/share on /mnt type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,fatal_neterrors=none,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.43.177,mountvers=3,mountport=47797,mountproto=udp,local_lock=none,addr=192.168.43.177)
```

проверки

делаю файл на сервере:
```
root@nfss:/# touch /srv/share/upload/FILE_FROM_SERVER
```
смотрю на клиенте наличие файла в "шаре", делаю файл с клиента:
```
root@nfsc:/# ls -la /mnt/upload/
total 8
drwxrwxrwx 2 nobody nogroup 4096 Oct 28 18:19 .
drwxr-xr-x 3 nobody nogroup 4096 Oct 28 17:42 ..
-rw-r--r-- 1 root   root       0 Oct 28 18:19 FILE_FROM_SERVER
root@nfsc:/# touch /mnt/upload/FILE_FROM_CLIENT
```
проверяю на сервере:
```
root@nfss:/# ls -la /srv/share/upload/
total 8
drwxrwxrwx 2 nobody nogroup 4096 Oct 28 18:20 .
drwxr-xr-x 3 nobody nogroup 4096 Oct 28 17:42 ..
-rw-r--r-- 1 nobody nogroup    0 Oct 28 18:20 FILE_FROM_CLIENT
-rw-r--r-- 1 root   root       0 Oct 28 18:19 FILE_FROM_SERVER
```

перезагружаю обе виртуалки:
```
root@nfsc:/# reboot
root@nfsc:/# Read from remote host 192.168.43.242: Connection reset by peer

root@nfss:/# reboot
root@nfss:/# Connection to 192.168.43.177 closed by remote host.
```

после ребута проверяю клиент:
```
cou@cou-PC:~$ ssh cou@192.168.43.242
cou@nfsc:~$ ls /mnt/upload/
FILE_FROM_CLIENT  FILE_FROM_SERVER

cou@nfsc:~$ showmount -a 192.168.43.177
All mount points on 192.168.43.177:
192.168.43.242:/srv/share

cou@nfsc:~$ touch /mnt/upload/FINAL_TEST_from_client
```

и сервер:
```
root@nfss:~# exportfs -s
/srv/share  192.168.43.242(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
root@nfss:~# showmount -a 
All mount points on nfss:
192.168.43.242:/srv/share
root@nfss:~# ls /srv/share/upload/
FILE_FROM_CLIENT  FILE_FROM_SERVER  FINAL_TEST_from_client
```
