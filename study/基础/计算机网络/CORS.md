---
tags:
  - 计算机基础
  - 计算机网络
  - HTTP
---
# 同源策略 SOP
## 定义
**浏览器**处于安全考虑，默认禁止**脚本**发起的**跨域**HTTP请求。
- 同源策略是由浏览器做出的限制，如果请求没有浏览器参与，就不会有跨域问题。
- 只有脚本发起的请求（`XHR/Fetch`）会存在跨域问题，`script`标签没有跨域限制。
- 浏览器不会拦截请求，而是**拦截响应**。如果触发跨域，浏览器不会将响应返回，而是会报错。

## 域
域=协议+域名+端口号
只有三者完全相同才是同一个域。
- `https://mail.qq.com`和`https://www.qq.com`
	- 不是同源，因为它们域名不同
- `https://mail.qq.com/?id=1`和`https://mail.qq.com/?name=alice`
	- 同源，因为它们的协议（HTTPS）、域名（`mail.qq.com`）、端口号（443）完全相同

## 原因
> **为什么浏览器需要同源策略？**

浏览器会自动帮用户管理状态。如果用户之前登录过`taobao.com`，浏览器就会保存淘宝网站的`Cookie`，下次向`taobao.com`发送请求时，就会自动带上`Cookie`，淘宝服务器会根据`Cookie`验证用户身份。
如果没有同源策略，用户打开了恶意网站，该网站使用脚本向`taobao.com`发送请求，浏览器会自动带上`Cookie`，淘宝服务器根据`Cookie`，认为是用户本人在操作，就会同意请求。

# 预检请求
只靠拦截响应不能保证安全，因为恶意请求已经成功发送给服务器。
例如，恶意脚本发送的是删除资源的请求，则服务器接收到请求后会删除对应的资源，恶意脚本的目的已经达成，不需要收到响应。为此，某些情况下，在发送正式跨域请求之前，服务器首先要向服务器发送预检`OPTIONS`请求，验证成功后再发送正式请求。

## 简单请求/非简单请求
只有**跨域**的**非简单请求**才需要预检。
简单请求：
- 请求方法为`GET/POST/HEAD`
- 所有请求头必须在安全请求头列表中
- Content-Type必须为以下三者之一
	- `text/plain`
	- `application/x-www-form-urlencoded`
	- `multipart/form-data`

> 只靠同源策略+预检请求依然不能解决所有安全问题，因为POST请求仍然可能不安全。需要搭配其它机制。

## 预检请求

### 请求头
- `Origin` 源
- `Access-Control-Request-Method`   正式请求的请求方法
- `Access-Control-Request-Headers` 正式请求的自定义请求头
### 响应头
- `Access-Control-Max-Age `   预检请求的缓存TTL(s)，后续满足条件的请求不发送预检请求
- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Headers`

# 解决跨域问题
## 反向代理
> **一般的请求流程**

1. 浏览器向前端服务器（A）请求页面的HTML和JS
2. 前端服务器返回的JS中包含xhr或fetch请求
3. 浏览器执行该请求，设置Origin: A，发送给后端服务器（B）
4. 后端服务器发送响应
5. 该请求的Origin为A，而目标源为B，触发跨域，浏览器拦截响应

为了解决这个问题，可以让前端服务器作为代理，代理浏览器向后端服务器发送请求，并转发响应。

> **设置反向代理后的流程**

1. 浏览器向前端服务器（A）请求页面的HTML和JS
2. 前端服务器返回的JS中包含xhr或fetch请求
3. 浏览器执行该请求，设置Origin: A，发送给前端服务器（A）
4. 前端服务器（A）转发该请求给后端服务器（B）
5. 后端服务器（B）发送响应给前端服务器（A）
	- 由于没有浏览器参与，不会有跨域问题
6. 前端服务器（A）将响应转发给浏览器
	- 由于该请求的Origin为A，目标源也为A，不会有跨域问题

## CORS
设置响应头
- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Headers`

## JSONP
历史遗留方案。利用script标签没有跨域限制的机制，解决跨域问题。
- 前端：动态创建script标签
``` js
// 1. 定义接收数据的函数
function handleResponse(data) {
    console.log("拿到跨域数据了：", data);
}

// 2. 动态插入脚本
const script = document.createElement('script');
script.src = 'https://api.example.com/getUser?id=123&callback=handleResponse';
document.body.appendChild(script);
```
- 后端：返回js代码，调用callback
``` js
const callback = query.callback; // 获取 'handleResponse'
const data = { name: "Gemini", role: "AI" };

// 返回 "handleResponse({"name":"Gemini"...})"
res.send(`${callback}(${JSON.stringify(data)})`); 
```

> **JSONP的缺陷**

- 只支持`GET`请求方法
- 不安全，易受`XSS`攻击
- 报错处理困难