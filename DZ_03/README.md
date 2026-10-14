PV остались от предыдущей практики, делаю новые  VG и LV:
```
root@dz2:~# pvs
  PV         VG Fmt  Attr PSize  PFree 
  /dev/sdb      lvm2 ---  25.00g 25.00g
  /dev/sdc      lvm2 ---  25.00g 25.00g

root@dz2:~# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created

root@dz2:~# vgs
  VG       #PV #LV #SN Attr   VSize   VFree  
  vg_root   1   0   0 wz--n- <25.00g <25.00g

root@dz2:~# lvcreate -n lv_root -l+100%FREE /dev/vg_root
root@dz2:~# mkfs.ext4 /dev/vg_root/lv_root
```

монтирую lv_root в /mnt, копирую туда корень ФС
```
root@dz2:~# mount /dev/vg_root/lv_root /mnt
root@dz2:~# rsync -avxHAX --progress / /mnt/
...
sent 6,928,095,865 bytes  received 1,385,050 bytes  251,981,124.18 bytes/sec
total size is 8,526,371,983  speedup is 1.23

root@dz2:~# ls -la /mnt/
total 4003940
drwxr-xr-x 21 root root       4096 Oct 14 12:56 .
drwxr-xr-x 21 root root       4096 Oct 14 12:56 ..
lrwxrwxrwx  1 root root          7 Apr 20  2026 bin -> usr/bin
drwxr-xr-x  3 root root       4096 Oct 15 10:27 boot
dr-xr-xr-x  2 root root       4096 Aug 26 05:03 cdrom
drwxr-xr-x  2 root root       4096 Oct 20 13:00 dev
drwxr-xr-x 87 root root       4096 Oct 15 10:27 etc
drwxr-xr-x  3 root root       4096 Oct 12 10:46 home
lrwxrwxrwx  1 root root          7 Apr 20  2026 lib -> usr/lib
lrwxrwxrwx  1 root root          9 Apr 20  2026 lib64 -> usr/lib64
drwx------  2 root root      16384 Oct 12 10:43 lost+found
drwxr-xr-x  2 root root       4096 Aug 26 04:45 media
drwxr-xr-x  2 root root       4096 Oct 20 13:01 mnt
drwxr-xr-x  2 root root       4096 Aug 26 04:45 opt
dr-xr-xr-x  2 root root       4096 Oct 20 12:49 proc
drwxr-xr-x  7 root root       4096 Oct 14 12:56 raid
drwx------  7 root root       4096 Oct 13 12:48 root
drwxr-xr-x  2 root root       4096 Oct 20 13:22 run
lrwxrwxrwx  1 root root          8 Apr 20  2026 sbin -> usr/sbin
drwxr-xr-x  6 root root       4096 Aug 26 04:47 snap
drwxr-xr-x  2 root root       4096 Aug 26 04:45 srv
-rw-------  1 root root 4099932160 Oct 12 10:45 swap.img
dr-xr-xr-x  2 root root       4096 Oct 20 12:49 sys
drwxrwxrwt  2 root root       4096 Oct 20 13:22 tmp
drwxr-xr-x 12 root root       4096 Aug 26 04:45 usr
drwxr-xr-x 13 root root       4096 Oct 12 10:56 var
```

монтирование и изменение 
```
root@dz2:~# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done

root@dz2:~# chroot /mnt/

root@dz2:/# grub-mkconfig -o /boot/grub/grub.cfg 
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/kdump-tools.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-7.0.0-38-generic
Found initrd image: /boot/initrd.img-7.0.0-38-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

```
root@dz2:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-7.0.0-38-generic
root@dz2:/# exit
root@dz2:~# reboot
```

корневой раздел переехал в LVM "lv_root"
```
cou@dz2:~$ lsblk 
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0               7:0    0 66.8M  1 loop /snap/core24/1643
loop1               7:1    0 66.8M  1 loop /snap/core24/2124
loop2               7:2    0  1.5M  1 loop /snap/hwctl/131
loop3               7:3    0 50.1M  1 loop /snap/snapd/27710
loop4               7:4    0  1.5M  1 loop /snap/hwctl/123
loop5               7:5    0 44.7M  1 loop /snap/snapd/28254
sda                 8:0    0   25G  0 disk 
├─sda1              8:1    0    1M  0 part 
└─sda2              8:2    0   25G  0 part 
sdb                 8:16   0   25G  0 disk 
└─vg_root-lv_root 252:0    0   25G  0 lvm  /
sdc                 8:32   0   25G  0 disk 
sdd                 8:48   0   25G  0 disk 
sde                 8:64   0   25G  0 disk 
sr0                11:0    1 1024M  1 rom 
```

изначально сда2 у меня был без ЛВМ-а, создаю PV,VG,LV, ФС, монтирую, копирую "/"
```
root@dz2:~# wipefs -a /dev/sda2
/dev/sda2: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
root@dz2:~# pvcreate /dev/sda2
  Physical volume "/dev/sda2" successfully created.
root@dz2:~# vgcreate vg_ubuntu /dev/sda2
  Volume group "vg_ubuntu" successfully created
root@dz2:~# lvcreate -n lv_ubuntu -L 8G /dev/vg_ubuntu
  Logical volume "lv_ubuntu" created.

root@dz2:~# lsblk 
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   25G  0 disk 
├─sda1                    8:1    0    1M  0 part 
└─sda2                    8:2    0   25G  0 part 
  └─vg_ubuntu-lv_ubuntu 252:1    0    8G  0 lvm  
sdb                       8:16   0   25G  0 disk 
└─vg_root-lv_root       252:0    0   25G  0 lvm  /
sdc                       8:32   0   25G  0 disk 
sdd                       8:48   0   25G  0 disk 
sde                       8:64   0   25G  0 disk 
sr0                      11:0    1 1024M  1 rom  

root@dz2:~# mkfs.ext4 /dev/vg_ubuntu/lv_ubuntu 
root@dz2:~# mount /dev/vg_ubuntu/lv_ubuntu /mnt
root@dz2:~# rsync -avxHAX --progress / /mnt/
```

монтирование, чрут, переустановка grub 
```
root@dz2:~# for d in dev sys proc run boot; do mount --bind /$d /mnt/$d; done
root@dz2:~# chroot /mnt/
root@dz2:/# grub-install /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
root@dz2:/# update-grub
...
root@dz2:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-7.0.0-38-generic
```

зеркало для var
```
root@dz2:/# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
root@dz2:/# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
root@dz2:/# lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  No input from event server.
  WARNING: Failed to monitor vg_var/lv_var.
  Logical volume "lv_var" created.
root@dz2:/# mkfs.ext4 /dev/vg_var/lv_var
...
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@dz2:/# mount /dev/vg_var/lv_var /mnt
root@dz2:/# cp -aR /var/* /mnt/
root@dz2:/# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
root@dz2:/# umount /mnt
root@dz2:/# mount /dev/vg_var/lv_var /var
root@dz2:/# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
root@dz2:/# echo "`blkid | grep lv_ubuntu: | awk '{print $2}'` / ext4 defaults 0 1" >> /etc/fstab
root@dz2:/# cat /etc/fstab 

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/swap.img	none	swap	sw	0	0
UUID="87104eda-1465-4619-9b38-c108ba941fa9" /var ext4 defaults 0 0
UUID="ebf609ad-3ab5-4988-896e-f55af2133c7c" / ext4 defaults 0 1
```
 
 удаляем временный "рут"
```
root@dz2:~# lvremove /dev/vg_root/lv_root 
Do you really want to remove and DISCARD active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed.
root@dz2:~# vgremove vg_root 
  Volume group "vg_root" successfully removed
root@dz2:~# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
```

отдельный раздел для "home"
```
root@dz2:~# lvcreate -n LV_Home -L 2G /dev/vg_ubuntu
  Logical volume "LV_Home" created.

root@dz2:~# mkfs.ext4 /dev/vg_ubuntu/LV_Home 
...
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@dz2:~# mount /dev/vg_ubuntu/LV_Home /mnt
root@dz2:~# cp -aR /home/* /mnt/
root@dz2:~# rm -rf /home/*
root@dz2:~# umount /mnt 
root@dz2:~# mount /dev/vg_ubuntu/LV_Home /home

root@dz2:~# echo "`blkid | grep LV_Home | awk '{print $2}'` /home ext4 defaults 0 2" >> /etc/fstab
```

практика создания снапшота и отката изменений:

```
root@dz2:~# touch /home/file{1..20}
root@dz2:~# lvcreate -L 100MB -s -n home_snap /dev/vg_ubuntu/LV_Home
  Logical volume "home_snap" created.

root@dz2:~# rm -f /home/file{5..15}
root@dz2:~# ls /home/
cou  file1  file16  file17  file18  file19  file2  file20  file3  file4  lost+found

root@dz2:~# umount /home 
root@dz2:~# lvconvert --merge /dev/vg_ubuntu/home_snap 
  Merging of volume vg_ubuntu/home_snap started.
  vg_ubuntu/LV_Home: Merged: 100.00%

root@dz2:~# mount /dev/vg_ubuntu/LV_Home /home

root@dz2:~# ls /home/
cou    file10  file12  file14  file16  file18  file2   file3  file5  file7  file9
file1  file11  file13  file15  file17  file19  file20  file4  file6  file8  lost+found
```