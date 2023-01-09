# TP 4 Real services


# Partie 1 : Partitionnement du serveur de stockage

🌞 **Partitionner le disque à l'aide de LVM**

- créer un *physical volume (PV)* :
```
[joel@storage ~]$ sudo pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created.
[joel@storage ~]$ sudo pvs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VB4ad623ac-468657af_ PVID AW0wbOdqm5TwkZiu4c6AaevkayX4gHxh last seen on /dev/sda2 not found.
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb      lvm2 ---  2.00g 2.00g
```


- créer un nouveau *volume group (VG)*
```
[joel@storage ~]$ sudo vgcreate storage /dev/sdb
  Volume group "storage" successfully created
[joel@storage ~]$ sudo vgs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VB4ad623ac-468657af_ PVID AW0wbOdqm5TwkZiu4c6AaevkayX4gHxh last seen on /dev/sda2 not found.
  VG      #PV #LV #SN Attr   VSize  VFree 
  storage   1   0   0 wz--n- <2.00g <2.00g
```
  
- créer un nouveau *logical volume (LV)* : 
```
[joel@storage ~]$ sudo lvcreate -l 100%FREE storage -n  last-storage
  Logical volume "last-storage" created.
[joel@storage ~]$ sudo lvs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VB4ad623ac-468657af_ PVID AW0wbOdqm5TwkZiu4c6AaevkayX4gHxh last seen on /dev/sda2 not found.
  LV           VG      Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  last-storage storage -wi-a----- <2.00g
``` 

🌞 **Formater la partition**

- vous formaterez la partition en ext4 (avec 
une commande `mkfs`)
```
[joel@storage ~]$ mkfs -t ext4 /dev/storage/last-storage 
mke2fs 1.46.5 (30-Dec-2021)
mkfs.ext4: Permission denied while trying to determine filesystem size
[joel@storage ~]$ sudo !!
sudo mkfs -t ext4 /dev/storage/last-storage 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 523264 4k blocks and 130816 inodes
Filesystem UUID: ebdbcb12-1563-4a68-8957-5a2b3cef86b2
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done 
Writing inode tables: done 
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 
```


🌞 **Monter la partition**

- montage de la partition (avec la commande `mount`)
  - [joel@storage ~]$ mkdir /mnt/storage1
mkdir: cannot create directory ‘/mnt/storage1’: Permission denied
[joel@storage ~]$ sudo !!
sudo mkdir /mnt/storage1
[joel@storage ~]$ sudo mount /dev/storage/last-storage  /mnt/storage1/


[joel@storage ~]$ df -h | grep mnt
/dev/mapper/storage-last--storage  2.0G   24K  1.9G   1% /mnt/storage1 
