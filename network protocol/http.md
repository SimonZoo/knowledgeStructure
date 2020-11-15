### HTTP请求的组成

HTTP请求由三部分组成，分别是：请求行、消息报头、请求正文。



#### 请求行格式

请求行格式：请求方法 统一资源标识符 HTTP版本 

举例： GET /index.html HTTP/1.1



#### 请求

不同HTTP版本中的请求方法，参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)

| HTTP1.0                 | HTTP1.1                                              |
| ----------------------- | ---------------------------------------------------- |
| GET<br />POST<br />HEAD | OPTIONS<br />PUT<br />DELETE<br />TRACE<br />CONNECT |

在[RFC5789](https://tools.ietf.org/html/rfc5789#section-2)加入PATCH方法。



通过是否出发CORS预检来判断是不是简单请求，满足以下**所有条件**的才会被视为简单请求：

❓所以以下条件就是用来判断会不会用options请求来发出CORS预检的咯？

答：这么说不太严谨，只是从简单请求的角度来判断，复杂请求的话会直接发预检。

1. 使用`GET、POST、HEAD`其中一种方法
2. 只使用了如下的安全首部字段，不得人为设置其他首部字段
   - `Accept`
   - `Accept-Language`
   - `Content-Language`
   - `Content-Type`，仅限以下三种
     - `text/plain`
     - `multipart/form-data`
     - `application/x-www-form-urlencoded`
   - HTML头部header field字段：`DPR、Download、Save-Data、Viewport-Width、WIdth`
3. 请求中的任意`XMLHttpRequestUpload` 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问
4. 请求中没有使用 ReadableStream 对象

#### POST/PUT/PATCH区别

参考：

- https://juejin.im/post/6844903813799739399
- MDN
- RFC

| PUT                                                          | POST                                           | PATCH                                                    |
| ------------------------------------------------------------ | ---------------------------------------------- | -------------------------------------------------------- |
| 对资源进行整体覆盖                                           | 没有对标准的补丁格式的提供支持                 | 对资源进行部分修改                                       |
| 幂等：调用一次与连续调用多次是等价的（即没有副作用）         | 非幂等，连续调用同一个POST可能会带来额外的影响 | 非幂等的，这就意味着连续多个的相同请求会产生不同的效果。 |
| **当 URI 指向一个存在的资源，服务器要做的事就是查找并替换。** |                                                | 如果要修改的资源不存在，会创建一个新的资源               |





#### 状态码

1xx：信息响应

2xx：成功响应

3xx：重定向

4xx：客户端响应

5xx：服务端响应





### HTTP2.0

参考：https://juejin.im/entry/6844903984524705800

**二进制分帧层**：所有传输信息分割为更小的消息和帧，并对它们采用二进制格式的编码将其封装。实现性能提升。

**多路复用 (Multiplexing) / 连接共享**：在http1.1中，浏览器客户端在同一时间，针对同一域名下的请求有一定数量的限制，超过限制数目的请求会被阻塞。这也是为何一些站点会有多个静态资源 CDN 域名的原因之一。
而http2.0中的多路复用优化了这一性能。多路复用允许同时通过单一的http/2 连接发起多重的请求-响应消息。

**头部压缩**：使用encoder来减少需要传输的header大小，通讯双方各自缓存一份头部字段表，既避免了重复header的传输，又减小了需要传输的大小。

**请求优先级**：把http消息分为很多独立帧之后，就可以通过优化这些帧的交错和传输顺序进一步优化性能。每个流都可以带有一个31比特的优先值：0 表示最高优先级；2的31次方-1 表示最低优先级。

**服务端推送**：推送遵循同源策略；基于客户端的请求响应来确定的