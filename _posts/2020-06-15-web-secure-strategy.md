---
layout: post
title: Web 安全策略
categories: web
---

## Cookie

### 限制

* 设置到 example.com 的 Cookie 可以被其子域名（test.example.com）读取。
* 设置到子域名（test.example.com）的 Cookie 只能被它自己以及它的子域名（foo.test.example.com）读取（它的父域名不行）。
* 子域名可以设置其父域名或者它的子域名的 Cookie ，但是不能设置兄弟域名的 Cookie。
* test.example.com 不能为 test2.example.com 设置 Cookie，但是可以为 foo.test.example.com 或者 example.com 设置 Cookie。

### Secure 以及 HTTPOnly

这两个标志是由服务器在第一次传输 Cookie 的时候通过 Set-Cookie 头部设置的。

* Secure：Cookie 只能被 HTTPS 页面读取。
* HTTPOnly：Cookie 不能被 Javascript 读取（只能通过 Web 请求）。

## SOP

Same-Origin Poliy（SOP 同源策略）

比 Cookie 安全策略要严格很多。

### 限制

1. 必须匹配协议，不能跨 HTTP/HTTPS。
2. 端口号必须匹配。
3. 域名必须确切匹配，不能匹配通配符或者子域名。

### 主要作用

1. 限制能使用 XMLHttpRequest 的域名。
2. 跨 frames/windows 获取 DOM。

### CORS（Cross-Origin Resource Sharing）

可以设置匹配模式，使得其可以对 SOP 限制之外的域名使用 XMLHttpRequest。

### CSRF（Cross-Site Request Forgery）

冒充受害者向目标站点提交数据。

服务器在正常情况下并不能分辨出用户是不是在页面上提交的数据（HTTP Referer 并不可信，但是可以用来辅助验证）。

目前的做法是使用随机生成的 CSRF Token（这个 CSRF 可以是用户 id 的编码后的信息（无状态），或者是有状态保存） 嵌入到表单中（Hidden Input）随表单一起提交并验证，只能用于 Post，不能也不应该用于 Get（直接通过链接来提交对于用户并不安全）。

Post 和 Get 的区别只是安全性上的问题，Post 要更为安全。比如你随时可以通过点击论坛上其他人的链接来使用 Get 提交数据，但是使用 Post 则一般需要 JS，不会那么容易。

不应该通过 Javascript 代码来设置 CSRF（每个表单会从服务器获取一个 csrf.js 文件，`$csrf = 'xxxx'` 然后通过 `<script src="csrf.js">` 引入），这样会使得攻击者能很方便的获取到 csrf 并伪装。

## 参考：

1. https://www.hacker101.com/sessions/web_in_depth.html
