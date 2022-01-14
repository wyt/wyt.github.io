---
title: Hibernate5.0快速入门
date: 2017-12-20 22:22:00
categories:
- Hibernate
tags:
- Java
---

参考 [Hibernate ORM 5.0 User Guide](https://docs.jboss.org/hibernate/orm/5.0/userguide/html_single/Hibernate_User_Guide.html)整理，作为快速入门简明手册。

## 体系结构

### 概述

![这里写图片描述](https://docs.jboss.org/hibernate/orm/5.0/userguide/html_single/images/architecture/data_access_layers.svg)

如上图所示：java应用利用Hibernate API 来完成 load, store, query等对其领域数据的操作。

作为JPA的提供者，Hibernate实现了JPA规范，JPA接口和Hibernate具体的实现关系如下图所示：

![这里写图片描述](https://docs.jboss.org/hibernate/orm/5.0/userguide/html_single/images/architecture/JPA_Hibernate.svg)

<!--more-->

SessionFactory (`org.hibernate.SessionFactory`)

Session实例工厂，一个线程安全的，不可变的代表应用领域模型到一个数据库的映射。`EntityManagerFactory`在JPA中等价于SessionFactory

Session (`org.hibernate.Session`)

一个单线程，短暂的对象，使用PoEAA《Patterns of Enterprise Application Architecture》中的“Unit of Work”概念设计。


Transaction (`org.hibernate.Transaction`)






















