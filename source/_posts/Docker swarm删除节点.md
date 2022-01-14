---
title: Docker swarm删除节点
date: 2019-5-15 11:31:49
updated: 2019-5-15 11:31:54
comments: true
categories:
- docker
tags:
- docker
---

### Docker swarm删除节点

``` bash
# 在管理节点上先排空节点
docker node update --availability drain yyxh5zbei71kq7c5tadoq19ri
# 移除节点
docker node rm yyxh5zbei71kq7c5tadoq19ri
# 或者强制删除
docker node rm --force yyxh5zbei71kq7c5tadoq19ri

# 在所要删除的节点上执行
docker swarm leave
# 或强制离开
docker swarm leave --force
```