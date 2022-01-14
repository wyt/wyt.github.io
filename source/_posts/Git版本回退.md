---
title: Git版本回退
date: 2019-3-4 15:39:00
updated: 2019-3-4 15:40:00
comments: true
tags:
- Git
---

改动未提交至远程仓库的情况

``` bash
# 将HEAD指向commit_id
[root@localhost ~]# git reset --hard commit_id
```

改动已提交至远程仓库的情况

``` bash
# revert是放弃指定提交的修改，会生成一次新的提交，需要填写注释，历史记录都在；而reset是指将HEAD指针指到指定提交;
[root@localhost ~]# git revert HEAD
[root@localhost ~]# git push origin master
```

<!--more-->

https://segmentfault.com/q/1010000000140446
https://blog.csdn.net/yxlshk/article/details/79944535
https://www.cnblogs.com/iloveyou-sky/p/6534409.html