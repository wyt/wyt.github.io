---
title: Gradle使用入门
date: 2022-2-17 17:47:20
tags:
- gradle
---

#### 安装
```shell script
# 安装gradle
choco install gradle
```

<!--more-->

#### 配置

在`%HOMEPATH%\.gradle`创建`init.gradle`文件，内容如下：

```groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        mavenLocal()
        mavenCentral()
    }
}
```
