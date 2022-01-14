---
title: 删除Git所有标签
date: 2018-10-4 11:48:00
updated: 2018-10-4 11:48:00
comments: true
---

### xargs命令

``` bash
[root@localhost ~]# echo "-lh" | cat
-lh
[root@localhost ~]# echo "-lh" | xargs ls
total 4.0K
-rw-------. 1 root root 1.5K Nov 14  2017 anaconda-ks.cfg
```

<!--more-->

### Remove all git tags

``` bash
#Delete local tags.
git tag -l | xargs git tag -d

#Fetch remote tags.
git fetch

#Delete remote tags.
git tag -l | xargs -n 1 git push --delete origin

#Delete local tasg.
git tag -l | xargs git tag -d
```