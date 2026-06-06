---
layout: post
title: linux+win双系统grub修复记录
date: 2026-02-09 08:03:37
tags:
  - grub
  - windows
  - linux
categories: 
  - linux
  - grub
author: dyf189
toc: true
---

## 背景
电脑上装了lubuntu和windows7的双系统，但是因为几次操作，导致开机直接进入lubuntu，Grub菜单消失  
重启后虽然grub菜单出来了，但是windows引导项消失。

## 修复过程

### 第一次出现  

这一次出现是因为安装了一个旧内核，reboot重启后直接进入lubuntu，Grub菜单消失。  

进入系统，打开终端后，第一次运行 `sudo update-grub` ，

成功检测到了系统的所有linux内核，但没有检测到windows.  

参照deepseek回答，进入 `/etc/default/grub` 文件，添加 `GRUB_DISABLE_OS_PROBER=false`项，

再次运行 `sudo update-grub` ，成功检测到了windows引导项。  

### 第二次出现

第二次出现的原因未知，但这一次直接运行 `sudo update-grub` 无法直接修复引导，

直接运行os-prober，也无法修复引导。

通过输入 `sudo fdisk -l`输出如下

```bash
sda                                                                             
├─sda1 ntfs                 3E409C6B409C2C21                       28.9G    56% /media/dyf189/3E409C6B409C2C21
├─sda2                                                                          
├─sda5 ntfs           软件  3914F32BFE858648                         71G    49% /media/dyf189/软件
├─sda6 ntfs           文档  5FDE84A8C15EA476                       73.2G    47% /media/dyf189/文档
└─sda7 ext4     1.0         e891be7a-5931-475f-a4e5-acb8cfdfce1d   68.8G    37% /
```

可以看到，sda7是lubuntu的根目录，sda1是windows的引导分区。而sda1被正常挂载。

参照deepseek建议，运行 `sudo fdisk -l /dev/sda`输出如下

```bash
Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1            2048 137556295 137554248  65.6G  7 HPFS/NTFS/exFAT
/dev/sda2       137558281 976768064 839209784 400.2G  f W95 Ext'd (LBA)
/dev/sda5       137558344 429064519 291506176   139G  7 HPFS/NTFS/exFAT
/dev/sda6       429066568 720572743 291506176   139G  7 HPFS/NTFS/exFAT
/dev/sda7  *    720582984 976768064 256185081 122.2G 83 Linux

Partition 2 does not start on physical sector boundary
```
sda1不是活动分区（但通过之后的操作，无法检测到windows7与sda1不是活动分区无关）

我先按照deepseek建议，将sda1设为了活动分区，出现了如下情况

```bash
/dev/sda1  *      2048 137556295 137554248  65.6G  7 HPFS/NTFS/exFAT
/dev/sda7  *    720582984 976768064 256185081 122.2G 83 Linux
```

sda1和sda7同时被设置成了活动分区，此时我还没有察觉到事情的不对劲，继续操作，自定义windows7引导项

重启后，直接进入了windows7，Grub菜单也没有出现。

无法进入lubuntu，幸运的是我的U盘里还保留着lubuntu的iso文件，同时U盘已经安装了ventoy。

通过U盘启动进入lubuntu系统，参照deepseek回答，

```bash
# 挂载您的 Ubuntu 分区
sudo mount /dev/sda7 /mnt

# 重新安装 GRUB 到 MBR
sudo grub-install --boot-directory=/mnt/boot /dev/sda

# 如果上面命令报错，尝试：
sudo grub-install --root-directory=/mnt /dev/sda

# 更新 GRUB 配置
sudo mount --bind /proc /mnt/proc
sudo mount --bind /dev /mnt/dev
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt update-grub

# 退出并重启
exit
sudo umount /mnt/proc /mnt/dev /mnt/sys
sudo umount /mnt
sudo reboot
```
重启后，成功进入lubuntu系统，Grub菜单也成功恢复了。

## 总结

经验：不可以盲信AI回答 

~~写这篇文章时可能是vscode装了Trae插件原因，出现了AI补全，其实效果并不好~~ 

第二次修复过程虽然修复成功了，但是里面还有一些问题没有得到解答。

1.Windows由正常激活状态转为未激活状态，使用激活工具后还是出现这种情况

2.理论来讲只能设置一个活动分区 ~~也不知道是Bug还是啥问题~~

## 后言

这是我博客的第一篇文章，~~虽然也不知道有多少人看，能帮到多少人~~，如果你觉得对你有用，可以在github给我点个star，谢谢。