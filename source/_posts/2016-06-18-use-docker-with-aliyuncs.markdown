---
layout: post
title: "Use docker with aliyuncs"
date: 2016-06-18 09:16:00 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

Docker 基于 LXC，运行docker 必须要提供 linux kernel 支持，所以想要在 Linux 以外的系统使用 docker 就必须通过虚拟机来提供基本的 Linux kernel 支持。因此，在 Windows 和 macOS 上使用 docker 我们需要安装 Virtualbox，然后安装自己心仪的 Linux 发行版虚拟机，由于这个虚拟机只是为了向 docker 提供 Linux 内核，我们可以安装体积尽可能小的发行版。boot2docker 就是这样的一个方案，它其实使用了 TinyCore 发行版提供内核支持，并内置好了 docker 软件。不过同样是小体积的发行版， Alpine linux 提供了一个完整的包管理器 APK 更易于使用，也有较高的安全性，所以我在这里选择了 Alpinelinux 来安装我们的虚拟机。

安装好 Alpine 之后，我们通过 apk 安装 docker `sudo apk add docker`，不过你的系统可能会提示找不到 docker ，因为 docker 只发布在 alpine 的社区源中，根据 [官方介绍](https://wiki.alpinelinux.org/wiki/Docker#How_to_use_docker ) 我们还需要添加 Alpine 的社区源，编辑 `/etc/apk/repostories` 文件增加一行 `http://dl-6.alpinelinux.org/alpine/edge/community`， 执行 `apk update` 更新软件信息后，就可以安装 docker 了。最后通过 `rc-update add docker boot` 让 docker 后台能够开机运行。

手动启动 docker 服务，`sudo service docker start`


下面我们来说说使用阿里云的容器服务吧：

首先开通阿里云容器服务，并设置你的 hub 密码，这个请牢记，后面会用到。

然后在我们的 Alpine 虚拟机中通过下面的命令添加阿里云容器服务镜像地址到 docker 参数中

```
echo "DOCKER_OPTS=\"\$DOCKER_OPTS --registry-mirror=https://这个地址是你的加速地址，和你的帐户挂钩.mirror.aliyuncs.com\"" | sudo tee -a /etc/default/docker
sudo service docker restart
```

当我们想使用阿里云的容器服务时，需要先进行登录:

```
sudo docker login -u 你的阿里云帐号 -p 你的hub密码(不是你的阿里云帐号密码)  registry.aliyuncs.com
```

然后就可以通过阿里云的容器服务来快速下载或者发布你需要的 docker 镜像了。

```
sudo docker pull debian
```

这是你可能会得到失败提示 `failed to register layer: ApplyLayer exit status 1 stdout:  stderr: chmod /bin/mount: permission denied`

你需要关闭内和安全特性来允许你构建内核：
`sysctl -w kernel.grsecurity.chroot_deny_chmod=0`
`sysctl -w kernel.grsecurity.chroot_deny_mknod=0`


总结：Linux 系统提供内核，docker 在内核基础上通过各种镜像组合启动新的容器实例，我们的应用开发，测试，不熟就可以通过镜像的方式安全高效的进行了。

最后为了提高开发测试阶段创建虚拟机的速度，我们还可以利用 Vagrant 来打包定制过的 Alpine 虚拟机，这一点可以参考这篇文章[为vagrant 创建你自己的 my_own_linux box](/blog/wei-vagrant-chuang-jian-ni-zi-ji-de-my-own-linux-box/)



