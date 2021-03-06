---
layout:     post
title:      "跨域问题/cookie/CSRF"
subtitle:   "Cross-site request forgery"
date:       2017-06-09 12:00:00
author:     "Lusha"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 网站
    - 后端
---


> 一次性弄懂跨域系问题：跨域是什么？为什么要跨域？为什么又要限制跨域？

## 跨域
跨域是指从一个域名的网页去请求另一个域名的资源。比如从www.baidu.com 页面去请求 www.google.com 的资源。跨域的严格一点的定义是：只要 协议，域名，端口有任何一个的不同，就被当作是跨域

## 为什么浏览器限制跨域
原因就是安全问题：如果一个网页可以随意地访问另外一个网站的资源，那么就有可能在客户完全不知情的情况下出现安全问题。比如下面的操作就有安全问题：

1. 用户访问www.mybank.com ，登陆并进行网银操作，这时cookie啥的都生成并存放在浏览器
2. 用户突然想起件事，并迷迷糊糊地访问了一个邪恶的网站 www.xiee.com（或一个被植入了危险代码的常用网站）
3. 这时该网站就可以在它的页面中，拿到银行的cookie，比如用户名，登陆token等，然后发起对www.mybank.com 的操作。
4. 如果这时浏览器不予限制，并且银行也没有做响应的安全处理的话，那么用户的信息有可能就这么泄露了。

因此浏览器在一定程度限制了跨域：ajax。

但是

* 浏览器一般不对script，img等进行跨域限制。因此为跨域访问（合理）和CSRF攻击（安全问题）留出了空间

## 为什么要跨域
既然有安全问题，那为什么又要跨域呢？ 有时公司内部有多个不同的子域，比如一个是location.company.com ,而应用是放在app.company.com , 这时想从 app.company.com去访问 location.company.com 的资源就属于跨域。

## 怎么跨域
1. 由于浏览器一般不对script，img等进行跨域限制，所以我们有机会通过script的方式来实现跨域访问。
2. 如果限制了script，还可以用JSONP，只支持get，不支持post
3. 代理：例如www.123.com/index.html需要调用www.456.com/server.php，可以写一个接口www.123.com/server.php，由这个接口在后端去调用www.456.com/server.php并拿到返回值，然后再返回给index.html，这就是一个代理的模式。相当于绕过了浏览器端，自然就不存在跨域问题。
4. 服务器端（PHP）修改header（XHR2方式）：
在php接口脚本中加入以下两句即可：
header('Access-Control-Allow-Origin:*');//允许所有来源访问
header('Access-Control-Allow-Method:POST,GET');//允许访问的方式
5. CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。
它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。
6. web sockets。web sockets是一种浏览器的API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对web sockets不适用)
web sockets原理：在JS创建了web socket之后，会有一个HTTP请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用HTTP升级从HTTP协议交换为web sockt协议。
只有在支持web socket协议的服务器上才能正常工作。
7. 还有几个前端策略，没看懂

## CSRF（Cross-site request forgery）跨站请求伪造
既然有多种多样的跨域方法，就有人利用这些方法来伪造请求，浏览器发送cookie，得到受信网站的信任。

几种防止CSRF的方法：

1. 表单验证码。比较粗暴，用户不友好
2.  验证HTTP Referer字段。它记录了该HTTP请求的来源地址。在通常情况下，访问一个安全受限页面的请求必须来自于同一个网站。比如某银行的转账是通过用户访问http://bank.test/test?page=10&userID=101&money=10000页面完成，用户必须先登录bank. test，然后通过点击页面上的按钮来触发转账事件。
3.  CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信息不存在于Cookie之中。鉴于此，系统开发者可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。该防御的关键在于，恶意网站访问受信网站时浏览器会自动发送cookie，但是它不能读取cookie。受信网站的前端根据cookie的token填充表单，而恶意网站的表单不知道这个token。
4.  在HTTP头中自定义属性并验证。自定义属性的方法也是使用token并进行验证，和前一种方法不同的是，这里并不是把token以参数的形式置于HTTP请求之中，而是把它放到HTTP头中自定义的属性里。通过XMLHttpRequest这个类，可以一次性给所有该类请求加上csrftoken这个HTTP头属性，并把token值放入其中。这样解决了前一种方法在请求中加入token的不便，同时，通过这个类请求的地址不会被记录到浏览器的地址栏，也不用担心token会通过Referer泄露到其他网站。

