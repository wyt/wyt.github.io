---
title: Docker Swarm节点维护标签
date: 2018-10-30 15:50:00
updated: 2018-10-30 13:38:00
comments: true
categories:
- docker
tags:
- docker
- docker swarm
---

### Swarm节点维护标签

``` bash
[root@docker_001 ~]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
lw72u2jd7xi1xagw94cmd86kk *   docker_001          Ready               Active              Leader              18.06.1-ce
2igmbyfampcduad32ustl00y8     docker_003          Ready               Active                                  18.06.1-ce
wudald9nltt9mq9jo9aau3tvf     docker_005          Ready               Active                                  18.06.1-ce

# 增加一个type=dop的标签
[root@docker_001 ~]# docker node update --label-add type=dop lw72u2jd7xi1xagw94cmd86kk
lw72u2jd7xi1xagw94cmd86kk

# 查看标签
[root@docker_001 ~]# docker node inspect lw72u2jd7xi1xagw94cmd86kk --pretty
ID:			lw72u2jd7xi1xagw94cmd86kk
Labels:
 - type=dop
Hostname:              	docker_001
Joined at:             	2018-10-24 13:52:32.919572847 +0000 utc
...

# 移除标签
[root@docker_001 ~]# docker node update --label-rm type lw72u2jd7xi1xagw94cmd86kk
lw72u2jd7xi1xagw94cmd86kk
[root@docker_001 ~]# docker node inspect lw72u2jd7xi1xagw94cmd86kk --pretty
ID:			lw72u2jd7xi1xagw94cmd86kk
Hostname:              	docker_001
Joined at:             	2018-10-24 13:52:32.919572847 +0000 utc
```

[Add-or-remove-label-metadata](https://docs.docker.com/engine/swarm/manage-nodes/#add-or-remove-label-metadata)