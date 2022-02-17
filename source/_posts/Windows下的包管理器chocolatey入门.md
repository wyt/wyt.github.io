---
title: Windows下的包管理器chocolatey入门
date: 2022-2-17 15:16:17
categories:
- 包管理器
- windows
tags:
- chocolatey
---

#### 介绍

一个`windows`包管理工具，类似`mac`下的`Homebrew`。

#### 安装

打开powershell，执行如下命令：

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

<!--more-->

![这里写图片描述](https://objects.yongtao.wang/images/20220217/20220217152250.png)

#### 基本使用

```shell script
# 本地安装包列表
choco list --local-only

#安装包
choco install <package>

# 安装指定版本
choco install <package> -version <版本号>

# 卸载包
choco uninstall <package>

# 升级本地包
choco upgrade <package>

# 搜索包相关
choco search <package>

choco list <package>

# 搜索所有版本
choco search <package> -all
```
  
More info: [chocolatey install](https://chocolatey.org/install#individual)
