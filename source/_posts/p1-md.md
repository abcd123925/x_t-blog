---
title: p1.md
date: 2022-09-13 03:59:55
tags:
---
# Raid (ubuntu 16.04)

（raid 0 ,raid 1）

## 准备工作：

 

## 最少需要两块盘

- 这里使用的是/dev/sdb和/dev/sdc

- 通过lsblk命令查看。

 

## 然后开始配置：
```
 1，sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc (raid 1只需要将l--evel=1)
```
###  2,上条命令成功后可以查看/proc/mdstat文件。（可选）
```
cat /proc/mdstat
```
(raid 1可以在这一步等待一下，使二者同步（会有进度条）)

### 3,格式化md0并挂载：
```

-   sudo mkfs.ext4 -F /dev/md0

-   sudo mkdir -p /mnt/md0

-   sudo mount /dev/md0 /mnt/md0
```
### 4,查看挂载是否成功（可选）
```
df -h -x devtmpfs -x tmpfs
```
### 5,保存Array设置
```
sudo mdadm --detail --scan \| sudo tee -a /etc/mdadm/mdadm.conf

sudo update-initramfs -u
```
使挂载后重启有效：
```
echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' \| sudo tee -a /etc/fstab
```
\---------------------------------------------------------------------------------------------------------------------------

### 删除上述设置，并恢复两块盘：
- 1,sudo umount /dev/md0 (根据设置的设备名，可以cat /proc/mdstat查看)
- 2，
```
sudo mdadm --stop /dev/md0

sudo mdadm --remove /dev/md0
```

-   3,查看两块盘的信息（可选）
```
lsblk
```

-   4,清空盘的superblock
```
sudo mdadm --zero-superblock /dev/sdc

sudo mdadm --zero-superblock /dev/sdb
```
-   5,注释配置信息。
```
（1）sudo vim /etc/fstab

. . .

/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0

1.  sudo vim /etc/mdadm/mdadm.conf

. . .

\# ARRAY /dev/md0 metadata=1.2 name=mdadmwrite:0 UUID=7261fb9c:976d0d97:30bc63ce:85e76e91

 

\- 6,sudo update-initramfs -u

\---------------------------------------------------------------------------------------------------------------------------
```
 

### raid5设置方法也一样。

只是最少需要三块盘
```
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
```
其他无差别。

### raid6至少需要四块盘
```
sudo mdadm --create --verbose /dev/md0 --level=6 --raid-devices=4 /dev/sda /dev/sdb /dev/sdc /dev/sdd
```
### raid10至少需要三块盘

可以使用--layout指示布局，不过也可以不用。那么配置方法就与上述的一样。

 

\---------------------------------------------------------------------------------------------------------------------------

 

 

 

*参考：How To Create RAID Arrays with mdadm on Ubuntu 16.04 \| DigitalOcean*

