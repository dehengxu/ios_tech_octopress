---
layout: post
title: "Add android annotation support in AndroidStudio project"
date: 2016-03-05 10:34:56 +0800
comments: true
published: false
sharing: true
footer: true
categories:
---

## 配置 gradle 环境

编辑 build.gradle 文件

```
apply plugin: 'android-apt'
def AAVersion = '3.3.2'

dependencies {
    apt "org.androidannotations:androidannotations:$AAVersion"
    compile "org.androidannotations:androidannotations-api:$AAVersion"
}

apt {
    arguments {
        androidManifestFile variant.outputs[0]?.processResoures?.manifestFile
    }
}

buildscript {
    repostories {
        mavenCentral()
    }

    dependencies {
        classpath 'com.needbedankt.gradle.plugins:android-apt:1.8' //For apt
    }
}

```
