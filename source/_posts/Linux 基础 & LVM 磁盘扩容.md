---
title: Linux 基础 & LVM 磁盘扩容   
date: 2019-11-2 21:03:2  
tags:  
- Linux  
- Ubuntu
- LVM
---

### 下载、安装Ubuntu  
> 官网下载：https://ubuntu.com/download/server  

### Linux目录结构  

| 目录   | 说明                                       |
|------|------------------------------------------|
| **bin**  | 存放二进制可执行文件\(ls,cat,mkdir等\)              |
| boot | 存放用于系统引导时使用的各种文件                         |
| dev  | 用于存放设备文件                                 |
| **etc**  | 存放系统配置文件                                 |
| **home** | 存放所有用户文件的根目录                             |
| lib  | 存放跟文件系统中的程序运行所需要的共享库及内核模块                |
| mnt  | 系统管理员安装临时文件系统的安装点                        |
| opt  | 额外安装的可选应用程序包所放置的位置                       |
| proc | 虚拟文件系统，存放当前内存的映射                         |
| root | 超级用户目录                                   |
| sbin | 存放二进制可执行文件，只有root才能访问                    |
| tmp  | 用于存放各种临时文件                               |
| **usr**  | 用于存放系统应用程序，比较重要的目录/usr/local 本地管理员软件安装目录 |
| **var**  | 用于存放运行时需要改变数据的文件                         |

### 修改数据源  

```
# 查看系统版本
lsb_release -a
# 编辑数据源  
vi /etc/apt/sources.list
## 删除全部内容，添加  
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
# 更新数据源
apt-get update
```

### LVM的基本概念  
#### 物理卷 Physical volume（PV）
> 可以在上面建立卷组的媒介，可以是**硬盘分区**，也可以是**硬盘本身**或者**回环文件**（loopback file）。物理卷包括一个特殊的 header，其余部分被切割为一块块物理区域（physical extents）  

#### 卷组 Volume group (VG)  
> 将一组物理卷收集为一个**管理单元**  

#### 逻辑卷 Logical volume (LV)  
> **虚拟分区**，由物理区域（physical extents）组成  

#### 物理区域 Physical extent (PE)
> 硬盘可供指派给逻辑卷的最小单位（通常为 4MB）  

### 开始LVM扩容  

```
# 查看 fdisk
fdisk -l  
```

![image](https://note.youdao.com/yws/api/personal/file/158366A2ECF241F38EF67284942E1B67?method=download&shareKey=03efad7f6657df0955557c96be683842)

> 因为这台机器默认开启了 LVM，所以目前有一个 **extended** 分区和一个 **LVM** 分区，并且他们是完全重叠的。这是因为，LVM 分区作为一个虚拟的分区，完全占用了这个 extended 分区，原理图见下：  
![image](https://note.youdao.com/yws/api/personal/file/4641A2265CDD4FCAA12DD369E1F9B6E3?method=download&shareKey=bb9a1f1d593a7653ab75f0d108755840)  
> 因此，现在需要做的就是将 extended partition (sda2) 扩展到最大，然后创建一个新的 LVM logical partition (sda6)，用它来填满 sda2  

#### 查看所有连接到电脑上的储存设备  

```
fdisk -l |grep '/dev'
```

**1 块磁盘效果图**  
![image](https://note.youdao.com/yws/api/personal/file/E01530FE7C014C08B037D8757099FA4B?method=download&shareKey=109c7985d3cfdeca6dec59c97c58f3fb)

**2 块磁盘效果图**
![image](https://note.youdao.com/yws/api/personal/file/474A8AA4FF1442CF948E14453A3BD1CB?method=download&shareKey=2f3b44b4c0ce214376481b1ed57cba36)

#### 创建 sdb 分区  

```
fdisk /dev/sdb
n	# 新建分区
l	# 选择逻辑分区，如果没有，则首先创建扩展分区（p），然后再添加逻辑分区（硬盘：最多四个分区 P-P-P-P 或 P-P-P-E）
# 格式化磁盘  
mkfs -t ext4 /dev/sdb1
# 创建 PV  
pvcreate /dev/sdb1
# 查看卷组
pvscan
# 扩容 VG
vgdisplay
vgextend ubuntu-vg /dev/sdb1
# 增加指定大小
lvextend -L +30G /dev/ubuntu-vg/root
# 按百分比扩容
lvextend -l +100%FREE /dev/ubuntu-vg/root
# 刷新分区
resize2fs /dev/ubuntu-vg/root
# 删除 unknown device
pvscan
vgreduce --removemissing ubuntu-vg
```


