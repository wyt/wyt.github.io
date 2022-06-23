---
title: Elasticsearch快速入门
date: 2022-06-23 10:28:00
categories:
- elasticsearch
tags:
- elasticsearch
---

### 基本概念

### 操作索引

ES中访问数据的模式：

```html
<HTTP Verb>/<Index>/<Endpoint>/<ID>
```

```html
<!-- 创建索引 -->
PUT /customer

<!-- 创建文档 -->
PUT /customer/_doc_1
{
    "name":"John Doe"
}

<!-- 获取文档 -->
GET /customer/_doc/1

<!-- 删除索引 -->
DELETE /customer
```

### 修改数据

### 探索数据

* Elasticsearch权威指南，赵建亭，清华大学出版，978-7-302-56594-9
