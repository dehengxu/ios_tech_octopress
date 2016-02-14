---
layout: post
title: "Use jenkins with "Self-Organizing Swarm Plug-in Modules""
date: 2015-12-29 16:15:39 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---

使用 Jenkins 做 ci，如果你的编译服务器和 Jenkins 服务器是分离的，那么你需要通过设置成 从节点 让需要执行 ci 任务的机器被 Jenkins 管理。

Jenkins 自带有一个叫做 Self-Origanzing Swarm Plug-in Modules 的插件，可以完成服务器自管理工作。这和从服务器有区别的地方在于，从服务器的工作方式是 Jenkins 作为主服务器直接登录从服务器进行主动工作；而自管理模式则是在“从”服务器上运行一个服务反向搜寻 Jenkins 的服务，自己登记到 Jenkins 的从节点中。然后 Jenkins 就可以为它分配任务了。

可以到这个地址：http://maven.jenkins-ci.org/content/repositories/releases/org/jenkins-ci/plugins/swarm-client/  下载对应版本的swarm-client jar 包，如果想独立使用，可以直接下载 with-dependency.jar 的文件（自带所有依赖包）的版本。

启动 swarm-client 的脚本

```
#!/usr/bin/env bash
java -jar swarm-client-2.0-jar-with-dependencies.jar \
-name MacMini-compiling-server \
-username xxx \
-password xxxxxxx \
-master http://localhost:8080/
```

这里的 `http://localhost:8080/` 可以改成你的 jenkins 服务器地址。
