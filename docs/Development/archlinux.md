# Arch Linxu Note

## Install System

- official website [https://archlinux.org](https://archlinux.org)

1. Ddownload iso
2. Partition the disks
    1. fdisk -l 列出所有的磁盘
    2. fdisk /dev/sda 进入选中磁盘
    3. p 打印分区 d 删除分区表 g 创建GPT分区表 UEFI and GPT
    4. n 创建分区并分配大小
        - /boot +512M
        - /swap +2G
        - /mnt
    5. w 写入分区
3. 格式化分区
    1. 格式化根分区         mkfs.ext4 /dev/分区名称
    2. 格式化交换分区       mkswap /dev/分区名称
    3. 格式化系统启动分区   mkfs.fat -F 32 /dev/分区名称
4. 挂载分区
    1. mount /dev/根分区名称 /mnt
    2. mount --mkdir /dev/系统启动分区 /mnt/boot
    3. swapon /dev/交互分区
5. 安装必要软件
```shell
pacstrap -K /mnt base linux linux-firmware sudo vim dhcpcd grub efibootmgr
```
6. 生成fstab文件
```shell
genfstab -U /mnt >> /mnt/etc/fstab
检查 /mnt/etc/fstab 文件
```
7. chroot到安装的系统
```shell
arch-chroot /mnt
```
8. 修改时区
```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```
9. 本地化文本
    1. 编辑 /etc/locale.gen 文件，取消 en_US.UTF-8 UTF-8 前方注释。
    2. 执行 locale-gen
    3. 编辑 /etc/locale.conf 文件
    ```shell
        LANG=en_US.UTF-8
    ```
10. 配置hostname。
```shell
编辑 /etc/hostname 文件
admin
```
11. root密码
```shell
passwd
```
12. 安装引导程序
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
13. 重启系统
```shell
reboot
```

## Config System

### 安装常用软件
gcc make dhcpcd openssh git sudo

### 创建用户
1. 安装sudo
```shell
pacman -S sudo
```
2. 允许 wheel 获取sudo权限
```shell
EDITOR=vim visudo
取消 %wheel ALL=(ALL:ALL) ALL 前面注释
```
3. 创建用户admin并添加进wheel组
```shell
useradd -m -G wheel admin
```
4. 修改admin密码
```shell
passwd admin
```

### dhcpcd
- 开机自启dhcpcd
```shell
sudo systemctl enable dhcpcd
```

### ssh
- 安装openssh
```shell
sudo pacman -S openssh
```
- 开机自启
```shell
sudo systemctl enable sshd
```