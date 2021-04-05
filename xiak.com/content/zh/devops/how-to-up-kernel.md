---
title: "如何升级CentOS内核"
date: 2019-03-05T21:6:00+08:00
author: Xiak
image : "images/blog/light.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["DevOps"]
tags: ["devops", "Linux"]
description: ""
draft: false
type: "post"
---

# 如何升级 CentOS7 内核

## CentOS 7 内核包
- kernel: This package contains the kernel for single-core, multi-core, and multi-processor systems
- kernel-devel : This contains kernel headers and makefiles used to build kernel modules against installed Kernel version.
- kernel-headers: This package includes the C header files that powers interfacing between the Linux kernel and user-space libraries and programs.
- kernel-tools: Contains tools for manipulating the Linux kernel and supporting documentation.
- perf: This package contains the perf tool, which enables performance monitoring of the Linux kernel.
- linux-firmware: This contains the firmware files required by various devices to operate

## 安装

Import GPG key used for signing packages:
```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

> NOTE: 以下链接版本可能不是最新, 请按照 [elrepo guide](http://elrepo.org/tiki/HomePage) 更新下面的链接

Add ELRepo repository to your CentOS 7 by running the commands below:
```bash
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```

Add ELRepo repository to your CentOS 8 by running the commands below:
```bash
yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```

This will add elrepo-kernel channel which provides both the long-term(kernel-lt) support kernels and latest stable mainline kernels(kernel-ml) for RHEL and CentOS. This channel is not enabled by default and you need to explicitly enable it before installing Kernel 5.x on CentOS 7.
```bash
$ yum --disablerepo="*" --enablerepo="elrepo-kernel" list available | grep kernel-ml
 kernel-ml.x86_64                        5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-devel.x86_64                  5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-doc.noarch                    5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-headers.x86_64                5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-tools.x86_64                  5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-tools-libs.x86_64             5.0.0-1.el7.elrepo         elrepo-kernel
 kernel-ml-tools-libs-devel.x86_64       5.0.0-1.el7.elrepo         elrepo-kernel
```
ow that we have confirmed availability of Linux Kernel 5.x, we can proceed to install it.
```bash
yum --enablerepo=elrepo-kernel install kernel-ml
```
Setting Kernel 5.1.12 on CentOS 7 as default
```bash
$ awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

0:CentOSLinux(3.10.0-957.27.2.el7.x86_64)7(Core)
1:CentOSLinux(4.4.189-1.el7.elrepo.x86_64)7(Core)
2:CentOSLinux(5.1.12-1.el7.elrepo.x86_64)7(Core)   <------ This num is 2
3:CentOSLinux(4.4.179-1.el7.elrepo.x86_64)7(Core)
4:CentOSLinux(3.10.0-957.12.1.el7.x86_64)7(Core)
5:CentOSLinux(3.10.0-514.el7.x86_64)7(Core)
6:CentOSLinux(0-rescue-d05df9772421403f9808fd87bb58abb6)7(Core)

$ grub2-set-default 2
```
Reboot
```bash
$ reboot
$ uname -r
5.0.0-1.el7.elrepo.x86_64
```

Also install kernel-ml-devel,kernel-ml-headers,kernel-ml-tools,perf
```bash
yum -y --enablerepo=elrepo-kernel install kernel-ml-{devel,headers,perf}
```



## 删除 kernel

### 查看已经安装的内核

```bash
rpm -qa | grep kernel
```

```bash
sudo yum remove kernel-3.10.0-957.el7.x86_64
```