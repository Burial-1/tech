---
title: 对浏览器跨域问题的一些理解
date: 2016-01-22 15:08:33 +8
tags: [CORS, corss domain, 同源策略, 跨域, 浏览器]
---

### 问题宝宝

#### 1 移动应用开发

之前用 Ionic 写 mobile app，移动 app 自然是少不了调用 RESTful API 的数据。
开发的时候用浏览器进行调试，然后 console 里不停的出现下面的提示(**Access-Control-Allow-Origin**)：

```
XMLHttpRequest cannot load http://samlino.local/cag/get_leads. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:63342' is therefore not allowed access.
```

而我当时的解决方法是，在 chrome 装了一个叫[Allow-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi)的扩展，启用之后就不报错了。

也不知道什么什么原理，反正就这么一直用着。（不思进取）

#### 2 网络应用开发

后来写单页应用（SPA），静态文件的 js 里调用各大公司的 API， github、facebook 什么的，反正从来没有报过上面的错…
另外还写过用 NodeJS 做中间层，从 NodeJS 去 API 拿数据然后渲染再发到前端，也从来没报过错…

最近的一个 case 是，我们有用 AWS API Gateway, Lambda, DynamoDB 搭建的 API，API 被调用的时候就会 trigger 一系列的任务。然后有个小哥偷懒不想写 PHP，想在前端直接请求 AWS 的 API。于是问题来了…

- 浏览器不停的提示 No 'Access-Control-Allow-Origin'的错误， 拿不到任何返回的数据。
- 但是！服务器该做的 job 却都做了！

这根本不科学啊！！！所以？到底是什么鬼？！

### 同源策略(Same Origin Policy)

同源策略就是只有访问的内容来自**相同协议、相同主机、相同端口**的内容[^1]时，才会加载访问得到的内容。**浏览器是同源策略的一种实现**

- 协议：`location.protocol`，http 和 https 是两种不同的协议
- 主机：`location.host`，不同子域名之间都算跨域，例如www.baidu.com, tieba.baidu.com 是两个不同的源
- 端口：`location.port`

### CORS

之前脑子里大概有个[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)的概念，然后一直以为是服务器拒绝从浏览器跨域访问…因为要解决自己 call 自己不同域名下的 API 问题，就是去服务器设置一下 header…但其实是都是浏览器在作怪…

在浏览器中，允许跨域访问的资源的一些例子[^2]：
`<script src="..."></script>`
`<link rel="stylesheet" href="...">`
`<img>` `<video>` `<audio>`
`<object>` `<embed>` `<applet>`
`@font-face`
`<frame>` `<iframe>`

### Response Headers

先看看可以在浏览器中跨域请求的别人家(github)的 API 返回的 header：

```
Access-Control-Allow-Credentials:true
Access-Control-Allow-Origin:*
Access-Control-Expose-Headers:ETag, Link, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
...
```

看看自家 API 返回的 header

```
Connection:keep-alive
Content-Encoding:gzip
Content-Type:text/html
Date:Fri, 22 Jan 2016 07:07:01 GMT
...
```

差别就在于别人家的 API 返回了**Access-Control-Allow-Origin:\***
浏览器读到这个头部之后，才会加载请求的结果。

### 跨域的时候服务器收到请求了吗？

服务器当然收到请求了，不然怎么能返回头部…而且我上面举的诡异的例子中，请求 trigger 的 job 都完成了。

所以也就是说，服务器其实收到了你的请求，并且给你返回了全部的数据，但是浏览器看到头部就把 body 藏起来了，然后抛出错误提示…

### 为什么浏览器不允许跨域访问？

当然是为了安全……但是这个有点不太好理解，api 拿点数据有什么不安全的？！

下面这个例子比较清楚的说明同源策略是如何避免安全问题的：[^3]
网站 A：一个看起来跟银行网站几乎一样的假网站
网站 B：真正的银行网站
如果没有同源策略： 1.当用户来到 A 网站，以为是真的银行网站，然后输入自己的账号、密码（此时用户的账号、密码已经被网站 A 获取了） 2.网站 A 利用 ajax 把账号密码发到真正的银行网站 B，然后银行网站返回一个带有 token 的 form 需要用户输入手机验证码。 3.网站 A 获得这个 form 之后显示出来，让用户填入手机验证码……至此，网站 A 获得了账号、密码、手机验证码，已经可以登陆用户的账号了。

而当有同源策略时，到第 2 步的时候，网站 A 根本无法获取银行网站返回的带 token 的 form，所以就算有了账户和密码，也无法操作用户的账户。

当然啦，银行的加密机制肯定没有我描述的这么弱智，不然还要 U 盾之类的东东干嘛，这只是一个为了方便理解安全问题而杜撰的例子…

### 好奇宝宝

不知道 chrome 的这个插件[Allow-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi)，是怎么实现允许跨域请求的。
之前发现启用插件的时候，github 上的小图标全都不显示了…

[^1]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
[^2]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#Cross-origin_network_access
[^3]: http://stackoverflow.com/questions/9222822/why-do-browser-apis-restrict-cross-domain-requests
