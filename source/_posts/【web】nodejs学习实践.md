---
title: 【web】nodejs学习实践
date: 2018-11-12 21:21:12
tags: 
    - web
    - nodejs
---

### NPM配置国内源

```shell
# 设置淘宝源
npm config set registry https://registry.npm.taobao.org 
npm info underscore （如果上面配置正确这个命令会有字符串response）

# 出现错误：
npm info retry will retry, error on last attempt: Error: CERT_UNTRUSTED

# 这是因为ssl验证问题，我们取消ssl验证:
npm config set strict-ssl false
```



在我们创建 Node.js 第一个 "Hello, World!" 应用前，让我们先了解下 Node.js 应用是由哪几部分组成的：

1. **引入 required 模块**：我们可以使用 **require** 指令来载入 Node.js 模块。
2. **创建服务器**：服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
3. **接收请求与响应请求** 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。

#### Node.js 事件循环

Node.js 是单进程单线程应用程序，但是因为 V8 引擎提供的异步执行回调接口，通过这些接口可以处理大量的并发，所以性能非常高。

Node.js 几乎每一个 API 都是支持回调函数的。

Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。

Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.

