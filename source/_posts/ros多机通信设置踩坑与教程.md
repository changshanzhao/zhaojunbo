---
title: ros多机通信设置踩坑与教程
date: 2023-03-11 15:06:09
categories:
	- 教程
---

本教程针对EPRobot小车开发，但对ros通信的设置也具有一定的普适性。

## 1.确保树莓派是AP连接模式（默认是AP模式，这个一般没问题）

## 2.修改虚拟机home目录下的.bashrc文件，在最后附加上这段

```shell
export ROS_MASTER_URL=http://EPRobot:11311
export ROS_HOSTNAME=ubuntu
```

这段话的意思是声明主机为EPRobot，也就是我们的小车，从机为ubuntu（我们虚拟机的用户名）

## 3.先查一下你的ip地址，运行这条指令

```shell
ifconfig
```

inet后面的就是你的ip地址，AP模式下你的地址应该是196.168.12.xxx

实际上在=的后面，我们应该输入对应的ip地址，但是我们现在是用EPRobot和ubuntu替代了，我的理解是Ubuntu系统也有一个类似c语言宏定义的东西。在etc文件夹下有hostname和hosts这两个文件

其中，hostname里应该是这样的

```
ubuntu
```

hosts文件中应该有这两句话，需要对这两句进行修改

```shell
127.0.1.1   ubuntu
# 需要改成你查到的ip地址
192.168.12.xxx   ubuntu
# 确认一下有没有下面这句话，没有就加上
192.168.12.1 EPRobot
```

注意：

要用root用户权限编辑etc目录下的文件，例如：

```shell
sudo vim /etc/hosts
```

如果不会用vim，就执行下面这句：

```shell
sudo gedit /etc/hosts
```

## 4.我就是在下面这步踩坑了

其实说起来很弱智，不过确实是我疏忽大意了。上述的这些配置，小车也要改，呜呜呜。

首先

```
ssh EPRobot@192.168.12.1
```

输入密码

```
ncut1234
```

然后你就能用命令行操作树莓派了，我建议大家还是学一下Linux命令行操作，有时候不得不用。

## 补充

#### vim的简单使用

按i进入insert模式（可编辑模式）

修改完之后按esc

最后输入:wq退出
