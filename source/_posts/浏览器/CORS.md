---
title: CORS
tags: ['CORS']
toc: true
date: 2021-05-18 07:38:13
categories: 浏览器
---
# 简介
CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

# 两种请求
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

**只要同时满足以下两大条件，就属于简单请求**
(1) 请求方法是以下三种方法之一：
- HEAD
- GET
- POST

(2) HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

### 简单请求的基本流程    
浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段，就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。【这种错误无法通过状态码识别,HTTP回应的状态码有可能是200】

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。以Access-Control-开头

- Access-Control-Allow-Origin
该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。


- Access-Control-Allow-Credentials
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。

- Access-Control-Expose-Headers
默认只能拿到6个基本字段，如果想拿到其他字段，就必须在Access-Control-Expose-Headers 里面指定

### withCredentials 属性
上面说到，CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。
另一方面，开发者必须在AJAX请求中打开withCredentials属性。


*需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。*

### 非简单请求
1. 预检请求
非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。

- Access-Control-Request-Method
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法

- Access-Control-Request-Headers
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段




2. 预检请求的回应
服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。  

返回 Access-Control-Allow-Origin  表示同意任意跨源请求

- Access-Control-Allow-Methods 该字段必需，表明服务器支持的所有跨域请求的方法，这是为了避免多次"预检"请求。

- Access-Control-Allow-Headers 
如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。不限于浏览器在"预检"中请求的字段。

- Access-Control-Allow-Credentials
允许发送cookie 

- Access-Control-Max-Age 
该字段可选，用来指定本次预检请求的有效期，单位为秒。在有效期内，不用发出另一条预检请求。

3. 浏览器的正常请求和回应
一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。

# 与JSONP比较  
CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。  

# 总结
1. CORS 请求分成两类：简单请求、非简单请求 
2. 简单请求     
请求头：origin    
响应头：   
Access-Control-Allow-Origin 、Access-Control-Allow-Credentials(可选) 、 Access-Control-Allow-Expose-Header (可选)

3. 发送cookie    
服务端要指定 Access-Control-Allow-Credentials字段。
开发者在AJAX请求中打开withCredentials 属性 

4. 非简单请求 预检  
要发送预检请求，请求头：
Origin、Access-Control-Request-Method、Access-Control-Request-Headers（可选）

服务器收到预检请求，检查后 确认允许跨域请求会作出响应

响应头：Access-Control-Allow-Origin 、Access-Control-Allow-Methods、Access-Control-Allow-Headers（可选）、Access-Control-Allow-Credentials（可选）、Access-Control-Max-Age 用于指定本次预检请求的有效期（可选）

5. 非简单请求 正常请求和回应
通过预检请求后，和简单请求一样会有一个Origin请求头，Access-Control-Allow-Origin 响应头

6. JSONP 只支持get 请求，但兼容老浏览器。CORS支持所有类型的http请求

> [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)