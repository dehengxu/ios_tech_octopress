---
layout: post
title: "为vagrant 创建你自己的 my_own_linux box"
date: 2016-06-16 13:23:01 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---
1. 安装 Virtualbox
2. 安装 vagrant
3. Virtualbox 新建本地虚拟机 起名为 my_own_linux
4. 安装 ubuntu 16.06 server
5. 系统用户名密码都设置成 `vagrant`
6. 建立 `/home/vagrant/.ssh/authorized_keys` 文件
7. 在宿主系统中执行 `vagrant package --base my_own_linux`，这时 vagrant 会从 Virtualbox 中寻找该虚拟机，并导出成 box包。
8. 通过 `vagrant box add name my_own_linux.box` 添加到 vagrant 中。
9. 建立 vagrant 虚拟机环境 `vagrant init my_own_linux`
10. 启动虚拟机  `vagrant up`
11. `sudo -u vagrant wget https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub -O .ssh/authorized_keys`  为你的系统下载公钥，vagrant 才能通过 ssh 自动登录。
