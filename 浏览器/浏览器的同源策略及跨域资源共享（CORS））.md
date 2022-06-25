## 同源策略
>浏览器同源策略（Same-origin policy ）：
>当两个 URL 的协议、主机、端口号都相同时，称这两个 URL是 同源。 -- MDN

同源策略作用：控制不同源之间的资源交互，阻隔恶意文档，减少被 XSS、CFSR 攻击。   
**「协议、主机、端口号」只要有一个不相同，即为跨域。**

**网络访问**
- 允许跨域写操作：链接、重定向、表单提交。
- 允许跨域资源嵌入：<script src=xxx> <link href=xxx> <img src=xxx> <video> <audio <iframe> @font-face跨域字体。
- 不允许跨域读操作。

**API 访问**
- 允许对部分 Window 属性跨域访问：window.postMessage window.location window.opener window.frames [等属性和方法](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy#window)。
- 允许对部分 Location 属性跨域访问：location.replace。

**对存储在浏览器中的数据访问**
- 不允许访问 localStorage 和 Indexed。
- 允许给定域名以及其任何子域名(sub-domains) 访问 cookie，不管使用哪个协议（HTTP/HTTPS）或端口号。

## 如何获取跨域资源
  跨源资源共享（CORS）：基于 HTTP Header 的机制，通过一些字段配置，允许服务器标示除了它自己以外的其它源（协议、主机、端口号），使浏览器允许这些源访问自己的资源。
  这种机制通过浏览器发送一个“预检”请求（OPTIONS），检查服务器是否会允许要发送的真实请求。在预检请求中，浏览器发送的请求头中标示有 HTTP 方法和真实请求中会用到的头。
  真实请求为简单请求时，不需要发送预检请求。
  
#### CORS HTTP Header 配置字段
```shell
Access-Control-Allow-Origin: *    // 允许请求服务器资源的 origin，通常有白名单
Access-Control-Allow-Methods: POST, GET, OPTIONS    // 真实请求使用的请求方式
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type   // 真实请求携带的自定义首部字段
Access-Control-Max-Age: 86400   // 该响应的有效时间，在有效时间内，浏览器无须为同一请求再次发起预检请求。用于优化预检请求。
```
  
#### 简单请求
- HTTP 请求方法是： GET、POST、HEAD
- HTTP 头部字段不超出：Accept、Accept-Language、Content-Languge、Content-Type、Viewport-Width、Width、DPR、Downlink、Save-Data
- Content-Type 的值为三者之一：text/plain、multipart/form-data、application/x-www-form-urlencoded

  除此之外的请求就是**复杂请求**。
  
  
## 跨域请求携带 cookie
  **跨域请求默认不会携带 Cookie 信息**，如果需要携带，请配置下述参数：
```shell
// 请求头配置字段：
"withCredentials": true
  
// 响应头配置字段：
"Access-Control-Allow-Credentials": true
```
