---
title: Windows & ArchLinux双系统安装
subtitle:
date: 2024-12-08T21:35:49+08:00
slug: 69724bb
draft: false
author:
  name: mzbswh
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - linux
  - arch
  - misc
categories:
  - linux
  - misc
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

> [!NOTE] 简介
> 本文主要内容为Windows环境下安装ArchLinux双系统, 以及桌面安装和简单的美化配置。</br>
> 建议参考[ArchLinux官方文档](https://wiki.archlinux.org/title/Installation_guide)进行安装。  
> 建议参考[ArchLinux安装](https://archlinuxstudio.github.io/ArchLinuxTutorial)。

## 1. 准备工作

1. 下载地址: [ArchLinux下载](https://www.archlinux.org/download/), 国内用户下滑找到China镜像源下载。
2. 制作启动盘: [Rufus](https://rufus.ie/), [Etcher](https://www.balena.io/etcher/), [Ventoy](https://www.ventoy.net/cn/index.html)等工具。
3. 制作启动盘后重启电脑, 进入BIOS设置U盘启动。

## 2. 安装ArchLinux

### 2.1 连接wifi
```bash
# 首先确保wifi未被禁用
rfkill unblock wifi

iwctl                       # 进入wifi交互式命令行
device list                 # 查看设备列表，一般都是叫wlan0
station wlan0 scan          # 扫描wifi
station wlan0 get-networks  # 查看可用wifi
station wlan0 connect YOUR-WIFI-NAME  # 连接wifi

exit                        # 退出iwctl

ping archlinux.org          # 测试网络连接
```

### 2.2 更新系统时间
```bash
# 同步时间
timedatectl set-ntp true
# 国内用户可以设置为中国时间
timedatectl set-timezone Asia/Shanghai
```

### 2.3 分区
固态硬盘名字一般为`/dev/nvmex`, 机械硬盘名字一般为`/dev/sda`。记得前面都要带`/dev/`

- EFI分区: `/efi` 建议1G内即可
- swap分区: `/swap` 一般为内存的2倍或者等于内存大小
- 根分区: `/` 50G以上, 一些系统相关的软件应用都会安装在这里, 建议多留些空间
- 用户主目录: `/home` 剩余空间都分配到这里

```bash
lsblk                   # 查看磁盘分区信息
cfdisk                  # 分区工具
```

使用`cfdisk`可以比较直观的进行分区, EFI分区选择`EFI System`, Swap分区选择`Linux swap`, 其他的选择`Linux filesystem`。
分区完成后根据提示保存分区结果, 记得输入`yes`确定保存。

### 2.4 格式化分区
```bash
mkfs.vfat -F32 /dev/nvmexxx       # 格式化EFI分区
mkfs.ext4 /dev/nvmexxx            # 格式化根分区和用户目录
mkswap /dev/nvmexxx               # 格式化swap分区
```

### 2.5 挂载分区
挂载时需要注意挂载的顺序, 先挂载根分区, 再挂载EFI分区, 最后挂载swap分区。

```bash
mount /dev/nvmexxx /mnt          # 挂载根分区
mkdir /mnt/efi                   # 创建EFI挂载点
mount /dev/nvmexxx /mnt/efi      # 挂载EFI分区
mkdir /mnt/home                  # 创建用户目录挂载点
mount /dev/nvmexxx /mnt/home     # 挂载用户目录
# swap分区不需要挂载
swapon /dev/nvmexxx              # 激活swap分区
swapon --show                    # 查看swap分区
```

### 2.6 设置镜像源
软件包显示根据``/etc/pacman.d/mirrorlist`里的镜像源进行下载, 可以通过编辑该文件将国内的镜像源放在前面。
国内用户可执行以下命令下载国内镜像站:
```bash
curl -L "https://archlinux.org/mirrorlist/?country=CN&protocol=https" -o /etc/pacman.d/mirrorlist
# 下载完成后编辑文件, 将需要的镜像站前面的注释去掉
vim /etc/pacman.d/mirrorlist
```

### 2.7 安装基本系统
```bash
pacstrap /mnt base base-devel linux linux-firmware vim os-prober ntfs-3g networkmanager iwd dhcpcd
# 双系统安装时需要安装os-prober, ntfs-3g用于读写NTFS分区
```

### 2.8 生成fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab    # 查看生成的fstab
```

### 2.9 进入新系统
```bash
arch-chroot /mnt
```

### 2.10 设置时区
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc   # 当前的正确 UTC 时间写入硬件时间
```

### 2.11 设置区域和本地化
首先使用vim编辑`/etc/locale.gen`文件, 找到`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`两行, 去掉前面的注释`#`。
```bash
locale-gen # 生成本地化信息
vim /etc/locale.conf # 设置本地化 LANG=en_US.UTF-8
```

### 2.12 设置主机名
```bash
vim /etc/hostname # 设置主机名 myhostname
# 设置hosts
vim /etc/hosts
# 添加如下内容
127.0.0.1 localhost
::1       localhost
127.0.0.1 myhostname(填写你的主机名称)
```

### 2.13 设置root密码
```bash
passwd
```

### 2.14 添加非root用户
桌面环境安装时建议添加一个非root用户
```bash
# 添加用户 注意一定要将用户添加到wheel组，不然无法使用sudo
useradd -m -g users -G wheel -s /bin/bash username
# 编辑sudoers文件
vim /etc/sudoers # 找到 %wheel ALL=(ALL) ALL 去掉前面的注释
```

### 2.15 安装微码
用于修复 CPU 的已知问题或漏洞，并优化性能和功能。
```bash
# 根据处理器类型选择安装
pacman -S intel-ucode # intel处理器
pacman -S amd-ucode   # amd处理器
```

### 2.16 安装引导

> [!WARNING]
> 这是安装的最后一步也是关键的一步，请点击上述链接并按指引正确安装好引导加载程序后再重新启动。否则计算机重新启动后将无法正常进入 Arch Linux 系统。

```bash
pacman -S grub efibootmgr  # grub是引导程序, efibootmgr是UEFI引导管理工具
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB

# 编辑grub配置文件，去掉 #GRUB_DISABLE_OS_PROBER=false 前面的注释
vim /etc/default/grub

# 生成grub配置文件
grub-mkconfig -o /boot/grub/grub.cfg

# 如果是双系统, 这一步理论上能看到 windows boot manager相关的输出
# 如果没有，可以先将window引导所在的硬盘挂载到/mnt/efi/window下，然后重新执行grub-mkconfig，在将window引导所在的硬盘卸载
```

### 2.17 完成安装
```bash
exit            # 退出chroot
umount -R /mnt  # 卸载被挂载的分区
reboot          # 重启，拔掉U盘
```

## 3. Gnome桌面安装与美化

```bash
sudo pacman -Syu      # 更新系统
sudo pacman -S gnome  # 安装Gnome桌面
sudo pacman -S gnome-extra  # 安装Gnome附加组件(可选，可能会安装许多不需要的应用)

sudo pacman -S gdm    # 安装GDM登录管理器
sudo systemctl enable gdm.service  # 设置GDM开机启动

reboot               # 重启
```

### 3.1 输入法

推荐安装`fcitx5`输入法框架和`fcitx5-rime`中文输入法。
```bash
sudo pacman -S fcitx5 fcitx5-rime
sudo pacman -S fcitx5-configtool  # 输入法配置工具
```
编辑环境变量文件`/etc/environment`, 添加以下内容:
```bash
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS=@im=fcitx5
```

推荐安装雾凇拼音
```bash
cd ~/.local/share/fcitx5/
rm -rf rime/
git clone https://github.com/iDvel/rime-ice.git rime --depth 1
# 没有git先安装git
sudo pacman -S git

# 调整候选词个数
cd ~/.local/share/fcitx5/rime
vim default.yaml  # 找到 page_size 调整参数
```


### 3.2 gnome拓展
使用gnome拓展调整桌面布局和外观
```bash
sudo pacman -S gnome-shell-extensions gnome-tweaks
```

### 3.3 图标与主题
可以在[官方主题网站](https://www.gnome-look.org/browse/)下载主题和图标进行安装。

鼠标与图标放在`~/.icons`目录下, 主题放在`~/.themes`目录下。

注意有些主题下载的压缩包解压后可能有多个不同的主题文件夹, 需要将带有`index.theme`的文件夹放入上述目录下。
