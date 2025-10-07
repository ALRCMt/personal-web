---
title: 注意事项
author: ALRCMt
date: 2025-06-11
category: Jekyll
layout: post
---


以下为我实际搭建过程中的一些“小问题”（并不）和小巧思
<hr />

## 01.PVE 安装时卡死

如果你有一张独立显卡，那么在安装 PVE 时可能会卡在 Loading Driver...，这是因为缺少显卡驱动导致的

解决方法：

- 启动 Proxmox VE 安装程序
  启动计算机并进入 Proxmox VE 的引导程序菜单
- 选择安装选项：
  在引导菜单中，使用箭头键选择“Install Proxmox VE (Terminal UI)”选项
- 编辑引导参数：
  按下键盘上的 e 键进入编辑模式
- 修改 Linux 引导行：
  使用箭头键导航到以 linux 开头的那一行。
  将光标移动到该行的末尾
- 添加 nomodeset 参数：
  在该行的末尾，确保与最后一个参数之间有一个空格，然后输入 nomodeset。
  启动安装程序：
  完成编辑后，按下 Ctrl + X 或 F10 键（具体取决于系统提示）以启动安装程序

将通过禁用图形化模块解决该问题

## 02._PVE 网卡莫名其妙掉线问题 不确定_

~~网上看到的原因基本是 intel 的网卡所致，怀疑是驱动兼容性问题~~
有可能，不确定

```shell
# 先安装工具
apt -y install ethtool
ethtool -K eno1 tso off gso off
# 修改vi /etc/network/interfaces ，在iface eno1 inet manual下新增post-up ethtool -K eno1 tso off gso off
auto lo
iface lo inet loopback

iface eno1 inet manual
        post-up ethtool -K eno1 tso off gso off   #  新增这句

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.33/24
        gateway 192.168.0.1
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0

source /etc/network/interfaces.d/*
```

## 03.ssh 功能开启问题

因为 ubuntu 虚拟机的控制台与本机粘贴板不互通，又不想安装其它插件，于是打算用 windows 的 cmd 远程 ssh，但是 ubuntu 的 ssh 功能死活打不开，最终发现他妈的命令中是`ssh`而不是`sshd`

```shell
# 首先启动SSH服务
systemctl start ssh
# 确认SSH服务已启动，输入以下命令查看SSH服务状态
systemctl status ssh
```

## 04.PVE8 概要面板显示 CPU 温度

通过 shell 脚本自动配置，省时省力省心
运行这段指令：

```shell
(curl -Lf -o /tmp/temp.sh https://raw.githubusercontent.com/a904055262/PVE-manager-status/main/showtempcpufreq.sh || curl -Lf -o /tmp/temp.sh https://mirror.ghproxy.com/https://raw.githubusercontent.com/a904055262/PVE-manager-status/main/showtempcpufreq.sh) && chmod +x /tmp/temp.sh && /tmp/temp.sh remod
```

然后刷新页面就可以了

安装 CPU 温度检测软件 sensors

```shell
apt install lm-sensors -y
```

传感器探测

```shell
sensors-detect
```

全部选择 yes 即可，可能其中一个地方提示 ENTER ，按 回车键 即可

ISA adapter：CPU 温度信息

acpitz-acpi-0：主板温度信息

nvme-pci-0200：nvme 固态硬盘温度（如果有安装的话）普通的 sata 固态硬盘不会显示

- 如果你不是AMD的CPU就继续看这个：[https://www.dgpyy.com/archives/205/](https://www.dgpyy.com/archives/205/)

最气人的是不知道为什么我这里死活不显示 CPU 各核心温度，所以其他我也懒得配置了，平时使用

```shell
sensors
```

看一下基本温度就行了

> 后续：已破案，原来是 AMD 的 U 不会显示各核心温度  
> 如果你是 amd 的 CPU，当输入`sensors`后如下图  
> SYSTIN 是主板南桥温度  
> AUXTIN 是电源温度（前提是你有传感器，否则数据无效）  
> CPUTIN 是主板监控的 CPU 温度  
> Tctl/Tdie 是 CPU 为降温虚标的高温，目的是使风扇转速加快  
> 详细见 https://ngabbs.com/read.php?tid=42423467&rand=200

<img src="https://github.com/ALRCMt/MtAIO-Build/raw/main/photo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-09%20224901.png" alt="" width="700px"/>

## 05.Ubuntu 空间仅占用一半

在安装 ubuntu server 的过程中，默认只占用一半磁盘空间，如果想补救如下

使用 `df -h` 命令显示文件系统的总空间和可用空间信息

使用 `sudo vgdisplay` 命令查看发现 Free PE / Size 还有 剩余空间

```shell
 sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv # 调整逻辑卷的大小

 sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv # 调整文件系统的大小

 df -h # 再次查看，确认文件系统的总空间大小调整成功
```

## 06.PVE 更换 apt 源后报错

在修改`/etc/apt/sources.list.d/debian.sources`后  
pve 的"更新>存储库"页面报错出现 "\u{200b}" 的字样  
是零宽空格导致，在复制粘贴的过程中产生，自己多检查几遍

至于报错 _没有启用 proxmox ve 存储库没有得到任何更新_
忽视，反正也不更新

## 07.C6-State导致PVE崩溃

不知道为什么PVE运行一段时间会莫名其妙崩溃，且系统日志没有记录  
事后调查发现是**AMD Ryzen 1700X（初代锐龙/Zen 1）启用了C6 State模式自动节能卡死**  
参考文献：  
[https://blog.csdn.net/qq_33026779/article/details/145600293](https://blog.csdn.net/qq_33026779/article/details/145600293)  
[https://forum.proxmox.com/threads/pve-6-raidz2-freeze-every-day-ryzen-7-1700.66629/](https://forum.proxmox.com/threads/pve-6-raidz2-freeze-every-day-ryzen-7-1700.66629/)  

解决方法：进入主板bios，将**Global C-State Control**设置为disabled

## 08.BIOS时区错误

在设置自动开机的时候，我发现主板BIOS时间与PVE系统时间差了8小时  
调查原因：  
PVE默认将BIOS时间视为UTC时间，而上海时间（CST）是UTC+8，因此系统会自动将BIOS时间减去8小时以“对齐”时区
解决方法：  
``` shell
timedatectl set-local-rtc 1
# 作用：直接设置硬件时钟（RTC）使用本地时间（上海时间），停止UTC转换
```
验证命令：
``` shell
timedatectl | grep "RTC in local TZ"
# 若显示yes即生效
```

## 09.PVE开机显示ZFS导入错误

在将硬盘直通给TrueNAS后，PVE仍会尝试挂载ZFS，同时访问**可能**会导致**数据损坏**  
所以应该确保 PVE 宿主机不主动挂载或导入该 ZFS 池，而是由 TrueNAS 虚拟机独占访问  

立即导出 ZFS 池(如果已导入)
``` shell
zpool export MtData
```
因为我的PVE系统没有使用ZFS作为根文件系统，而是使用**LVM+ext4**  
所以我直接简单粗暴，完全移除ZFS工具  
``` shell
sudo apt purge zfsutils-linux zfs-zed -y
```

## 10.PVE降低功耗
CPU电源策略调整：
``` shell
# 安装 cpupower
apt install linux-cpupower

# 查看支持的 CPU 电源模式
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# 查看当前的 CPU 电源模式
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# CPU实时频率查看
watch -n 1 cpupower monitor

# 查看当前所有CPU的信息
cpupower -c all frequency-info

# 设置所有CPU为节能模式
cpupower -c all frequency-set -g powersave

# 设置所有CPU为性能模式
cpupower -c all frequency-set -g performance
```
<table>
<tr>
<th>电源模式</th><th>解释说明</th>
</tr>
<tr>
<th>performance</th><th>性能模式，将 CPU 频率固定工作在其支持的较高运行频率上，而不动态调节</th>
</tr>
<tr>
<th>userspace</th><th>系统将变频策略的决策权交给了用户态应用程序，较为灵活</th>
</tr>
<tr>
<th>powersave</th><th>省电模式，CPU 会固定工作在其支持的最低运行频率上</th>
</tr>
<tr>
<th>ondemand</th><th>按需快速动态调整 CPU 频率，没有负载的时候就运行在低频，有负载就高频运行</th>
</tr>
<tr>
<th>conservative</th><th>与 ondemand 不同，平滑地调整 CPU 频率，频率的升降是渐变式的，稍微缓和一点</th>
</tr>
<tr>
<th>schedutil</th><th>负载变化回调机制，后面新引入的机制，通过触发 schedutil sugov_update 进行调频动作</th>
</tr>
</table>	


``` shell
# 我这里设置CPU电源策略为模式conservative
cpupower -c all frequency-set -g conservative

```
设置机械硬盘休眠：
编辑 /etc/rc.local，在exit 0之前加如下
``` shell
hdparm -B 192 /dev/sda   # 设置APM级别为192
hdparm -S 240 /dev/sda   # 长时间不活动停转（20分钟）
```

