---
title: vue3插槽
date: 2022-03-02 11:33:00
categories:
- 前端
tags:
- vue
---

### 基本内容

定义一个组件com，使用slot标签占位：
```javascript
vm.component('com', {
    template: `<div><slot></slot></div>`
})
```

使用这个组件com：
```html
<div>
    <com>是非成败转头空，浪花淘尽英雄。</com>
</div>
```

组件com中的slot会被替换成`是非成败转头空，浪花淘尽英雄。`


