---
title: 记录一次安装 ArchLinux 的过程
published: 2024-12-17
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

### 观前提示

以下安装过程均使用**虚拟机**。  
我不会什么东西，安装也只是别人推荐。本文只旨在**记录安装过程**。

## 安装前的准备

### 物品

1. 计算机 *1（这不是废话吗）
2. U 盘 *1（别太小就行）
3. 键盘 *1（至少得能打字吧）
4. 手

### 计算机准备工作

连接到互联网，把键盘插上确保可以使用。

### 准备安装 U 盘

首先访问 [Arch Linux - Downloads](https://archlinux.org/download/) 下载镜像
![下载](/archlinux-install/download_1.png)
可以用上面这个下载，也可以往下翻到你所在的国家，通过镜像站进行下载。  
下载完后你会得到一个名字为 `archlinux-x86_64.iso` 的文件，这就是 Arch Linux 的安装镜像。  
我们可以使用如 **Rufus** 这样的烧写工具进行烧写。  
![rufus](/archlinux-install/rufus.png)
烧录完成后，一切就绪，启动计算机！

## 开始安装

### 调整启动顺序

插入 U 盘，调整启动顺序。
这里我就不说了，具体请自行百度或使用其他工具查询自己主板的调整方法。  
总之就是把刚才烧录的 U 盘调到第一位。

### 启动到 live 环境

前面的启动顺序调整完成后，重启应该就会进入一个这样的界面：
![boot](/archlinux-install/boot.png)
我们选择第一个选项，进入到 live 环境。  
进入后应该是这样的界面：
![shell](/archlinux-install/shell.png)
Arch Linux 的官方指南有讲使用 `archinstall` 进行安装，但是因为是初学者，所以我们最好还是使用命令行安装。  

### 硬盘分区

我们输入：
```
fdisk -l
``` 
查看硬盘，记住你需要分区的硬盘（例如：/dev/sda）。  
然后我们使用：
```
cfdisk /dev/sda(你需要分区的硬盘)
``` 
进入分区工具。
![cfdisk](/archlinux-install/cfdisk.png)
然后使用 New 和 Type 新建并修改分区：
1. 建立一个 EFI 分区，建议 1 GiB。然后使用 Type 修改分区类型至 `EFI System`。
2. 建立一个 Swap 分区，建议至少 4 GiB。然后使用 Type 修改分区类型至 `Linux swap`。
3. 建立一个根目录分区，可以直接使用设备剩余空间。然后使用 Type 修改分区类型至 `Linux root (x86-64)`。
4. 最后 Write 然后 Quit。退出 cfdisk。

然后就到了格式化的环节，我们继续使用：
```
fdisk -l
```
记住你分的三个区。  
1. 首先格式化根目录分区，我们使用 btrfs，所以使用命令：
```
mkfs.btrfs /dev/root_partition（根目录分区）
```
2. 格式化 Swap 分区，使用命令：
```
mkswap /dev/swap_partition（交换空间分区）
```
3. 格式化 EFI 分区，使用命令： 
```
mkfs.fat -F 32 /dev/efi_system_partition（EFI 系统分区）
```

最后挂载分区：  
1. 将根磁盘卷挂载到 `/mnt`：
```
mount /dev/root_partition（根分区） /mnt
```
2. 挂载 EFI 系统分区：
```
mount --mkdir /dev/efi_system_partition /mnt/boot
```
3. 启用 Swap 分区：
```
swapon /dev/swap_partition（交换空间分区）
```

### 选择镜像站

以中国大陆为例，使用如下命令下载位于中国大陆的 HTTPS 镜像站：
```
curl -L "https://archlinux.org/mirrorlist/?country=CN&protocol=https" -o /etc/pacman.d/mirrorlist
```
在操作后记得去除想要的源前面的#，以 nano 编辑器为例：
```
nano /etc/pacman.d/mirrorlist
```

### 安装必需的软件包

使用 pacstrap 安装：
```
pacstrap -K /mnt base linux-zen linux-firmware btrfs-progs networkmanager nano base-devel
```
linux-zen 被称为最适合日常使用的内核，但是如果你想更换，只需要把上面命令中的 `linux-zen` 更换为 [官方支持的内核](https://wiki.archlinuxcn.org/wiki/%E5%86%85%E6%A0%B8#%E5%AE%98%E6%96%B9%E6%94%AF%E6%8C%81%E7%9A%84%E5%86%85%E6%A0%B8) 即可。  
这时候可以同时额外安装计算机的 CPU 微码包。如果计算机是 Intel 的 CPU ，使用 `intel-ucode`，AMD CPU 则使用 `amd-ucode`。也可以暂时都不安装，等到进入系统后再安装。  

## 配置系统

### 生成 fstab 文件

通过以下命令生成 fstab 文件：
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot 到新安装的系统

通过以下命令 chroot 到新安装的系统：
```
arch-chroot /mnt
```

### 设置时区

以中国大陆为例，通过以下命令设置时区：
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
`Asia/Shanghai` 可以更换为 `Region（地区名）/City（城市名）`。  

然后运行 hwclock 以生成 `/etc/adjtime`：
```
hwclock --systohc
```

### 区域和本地化设置

请先设置 locale 为 en_US.UTF-8：
```
nano /etc/locale.gen
```
去掉 `en_US.UTF-8` 前面的 # 后保存并退出。  
然后创建 `locale.conf` 文件，并编辑设定 LANG 变量：
```
nano /etc/locale.conf
```
写入 `LANG=en_US.UTF-8` 后保存并退出。

### 写入主机名

创建 hostname 文件：
```
nano /etc/hostname
```
写入 *yourhostname（主机名）* 后保存并退出。

### 设置 root 密码

使用以下命令设置 root 密码：
```
passwd
```
会让你输入两遍，且输入过程中没有显示密码。

### 安装 GRUB 并配置

使用以下命令以安装 grub 和 efibootmgr：
```
pacman -S grub efibootmgr
```
在这里，我遇见了一个错误：`signature from ... is unknown trust`。   
手动升级 archlinux-keyring 也提示该错误，所以通过百度等搜索引擎得到以下解决方法：
```
sudo pacman-key --init
sudo pacman-key --populate archlinux
sudo pacman-key --refresh-keys
sudo pacman -Syu
```
解决问题后继续配置 GRUB：
```
grub-install --target=x86_64-efi --efi-directory=/mnt/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
配置完重启即可进入 Arch Linux 的界面！
![enter](/archlinux-install/enter.png)
图形界面下次再说，最近流感了然后不知道为什么一看电脑就会晕，已经要晕死了。