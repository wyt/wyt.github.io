---
title: Linux下ssh公钥认证
date: 2019-7-1 19:15:33
updated: 2019-7-1 19:15:37
comments: true
categories:
- linux
tags:
- linux
- ssh
---

|编号|机器名|IP|
|:---:|:---:|:---:|
|1|ansible-manager|192.168.91.140|
|2|cluster_001|192.168.91.141|
|...|...|...|

#### 创建密钥对

在ansible-manager上执行ssh-keygen命令，一路回车。

<!--more-->
``` bash
[root@ansible-manager .ssh]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:PQtPisFEFbco9jI1loIXyK7y/xr9n6vp562aQ/OkJJs root@ansible-manager
The key's randomart image is:
+---[RSA 2048]----+
|   . .o.o..      |
|    oo . + .     |
|   .. * * .      |
|    .= * o       |
|   .  = S +      |
|. .  ..=+=.o     |
| o  . o*.=o      |
|  .  .E.oooo     |
|   .oo..BB*o.    |
+----[SHA256]-----+

# 生成密钥对id_rsa & id_rsa.pub，其中id_rsa.pub是公钥。
[root@ansible-manager .ssh]# ls -lh
total 8.0K
-rw-------. 1 root root 1.7K Jul  2 03:25 id_rsa
-rw-r--r--. 1 root root  402 Jul  2 03:25 id_rsa.pub
-rw-r--r--. 1 root root    0 Jul  2 03:24 known_hosts
```

#### 发送公钥

将ansible-manager公钥发送给cluster_001，第一次发送公钥的时候，需要输入密码。

``` bash
[root@ansible-manager .ssh]# ssh-copy-id -i ./id_rsa.pub "root@192.168.91.141"
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./id_rsa.pub"
The authenticity of host '192.168.91.141 (192.168.91.141)' can't be established.
ECDSA key fingerprint is SHA256:HcrnywII1gZEW3uk2muw+V+JyD0tbedR8hbdvWNrFMM.
ECDSA key fingerprint is MD5:55:76:c1:5a:f5:ec:f6:3d:74:e7:d3:ec:ab:49:80:4d.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.91.141's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.91.141'"
and check to make sure that only the key(s) you wanted were added.
```

#### 测试

检查被管理机器cluster_001`/root/.ssh/authorized_keys`中，写入了ansible-manager 公钥内容。ansible-manager使用ssh连接cluster_001:

``` bash
[root@ansible-manager .ssh]# ssh 192.168.91.141
Last login: Tue Jul  2 03:39:14 2019 from 192.168.91.140
[root@cluster_001 ~]# hostname
cluster_001
```

#### 总结

1. 管理机上创建ssh密钥对；
2. 将管理机公钥分发给被管理机；
3. 实现互免认证，可在另一台机器上执行上述流程。
