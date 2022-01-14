---
title: Gitlab项目初始化
date: 2018-10-4 11:35:00
updated: 2018-10-4 11:35:00
comments: true
---

## 命令行指令

### Git 全局设置

``` bash
git config --global user.name "wangyt"
git config --global user.email "wangyt4dev@gmail.com"
```

### 创建新版本库

``` bash
git clone git@192.168.1.204:wangyt/test001.git
cd test001
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

<!--more-->

### 已存在的文件夹

``` bash
cd existing_folder
git init
git remote add origin git@192.168.1.204:wangyt/test001.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### 已存在的 Git 版本库

``` bash
cd existing_repo
git remote add origin git@192.168.1.204:wangyt/test001.git
git push -u origin --all
git push -u origin --tags
```