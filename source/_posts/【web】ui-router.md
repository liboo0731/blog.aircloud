---
title: 【web】ui-router学习笔记
date: 2018-10-21 10:11:33
tags: 
    - ui-router
---

##### ui-sref

```html
<a ui-sref="home">Home</a>
<a ui-sref="about">About</a>
<!-- 在当前状态只是参数变更，可以简写 -->
<a ui-sref="contacts">About</a>
<a ui-sref="{page: 2}">Next page</a>
<!-- 带参数写法 -->
<a ui-sref="contacts.detail({ id: contact.id })">{{ contact.name }}</a>
```

##### ui-sref-active

```html
<!-- 如果点击状态包含“home.users.list”，“用户列表”就会高亮 -->
<li ui-sref-active="active">
    <a ui-sref="home.users.list">用户列表</a>
</li>

<!-- 等同上面 -->
<li ui-sref-active="{'active':'home.users.list'}">
    <a ui-sref="home.users.list">用户列表</a>
</li>

<!-- 在同一模块下操作，保持导航栏该模块一直高亮，即该模块active值取公共状态 -->
<!-- 点击“用户1”或“用户2”，“用户列表”仍保持高亮 -->
<a ui-sref="home.users.list({'userId':1})">用户1</a>
<a ui-sref="home.users.list({'userId':2})">用户2</a>

```

##### ui-sref-opts

```html
<!-- location,inherit,reload -->
<a ui-sref="home" ui-sref-opts="{ reload: true }">Home</a>

<!-- instead of `click` -->
<input type="text" ui-sref="contacts" ui-sref-opts="{ events: ['change', 'blur'] }">
```

