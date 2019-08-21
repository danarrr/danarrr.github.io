---
title: http小书
date: 2019-02-11 18:50:59
categories: 
- 网络协议
tags: 
- http
---

## http简介
在地址栏内输入URL（http://example.com/hello.htm） ,确认回车后等待一些时间，就可以看到一个html页面呈现在浏览器内。
正是http这个协议规定了如何把客户端的请求打包为HTTP请求消息并发送给服务器，也是它规定了把一个响应打包成HTTP响应消息，然后送回客户端。

以hello.htm资源的获取过程为例，具体过程是这样的：
1.客户端软件打开到服务器的连接
> 请求：

```javascript
GET /hello.htm HTTP/1.1    // 客户端发了一个GET请求
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT) // 表明客户端接受美国英语的内容
Host: example.com
Accept-Language: en-us
Accept-Encoding: gzip, deflate
```

2.服务器软件根据资源定位符在服务器上定位并找到此资源，打包给出如下响应到客户端:
> 响应：
 
```javascript
HTTP/1.1 200 OK  //协议版本为1.1，状态码为200 由状态码可以知道这个请求在服务器已经成功处理
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache/2.2.14 (Win32)
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
Content-Length: 88  //本消息的主体内承载的长度
Content-Type: text/html //本消息的主体内承载的是一个html文件
Connection: Closed
```
```javascript
<html>
   <body>

   <h1>Hello, World!</h1>

   </body>
</html>
```

然后在这些首部头字段的帮助下，就解析出实体内的html文件内容，并呈现html给用户。

### 请求消息
#### 请求行
请求消息的第一行就是请求行。使用的 `请求方法 `、`资源标示符`、和`HTTP版本`。

- GET方法 表示我要请求一个指定名称的资源。
- PUT方法 表示如果指定URL不存在就创建它，否则就修改它。资源数据由消息主体提供。
- POST方法 表示要创建一个新的子资源，或者更新一个存在的资源。资源数据由消息主体提供。
- DELETE方法 表示我要删除一个指定名称的资源。
- CONNECT方法、OPTIONS方法、TRACE方法会在后面单独讲解。
PUT 和 POST 都可以创建和更新资源，如何选择？假设我们正在设计电子订单系统，那么：
- PUT /orders/1 创建订单号1的资源；如果此订单已经存在，那么就更新它。订单号是由客户端指定的。
- POST /orders 创建一个订单，新订单号由服务器指定。
- POST /orders/1 如果订单1存在就更新它。如果1不存在，应该抛出“资源未找到”错误。

####  首部字段/头字段行
零行或者多行头字段行。可以用来传递客户端的更多信息，以及传递解析消息主体的必要信息。案例中的:

```javascript
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: example.com
Accept-Language: en-us
Accept-Encoding: gzip, deflate
```
这里就是 头部字段名：头部字段值

#### 空行（CRLF)
指示头字段区完成，消息主体开始（如果有消息主体的话）。

#### 消息主体
消息主体是请求消息的承载数据。比如在提交POST表单，并且表单方法不是GET时，表单数据就是打包在消息主体内的。消息主体是可选的，本案例是没有主体的。



### 响应消息
响应消息由一个状态行、一个或者多个首部字段行、一个空行、消息主体 构成。
#### 状态行
由`http版本`、`状态码`、`状态描述文字`构成。
如`HTTP/1.1 200 OK`。状态码200表示成功。

状态码共有5组
* 200-299 成功。 指明客户端请求是正确的，并被成功执行。
* 300-399 重定向。指明客户端请求是正确的，不过当前请求资源的位置在别处，请再次定向你的资源位置，发起新的请求。
* 400-499 客户端错误。 指明客户端的请求是不正确的，可能是格式无法识别，或者URL太长等等。
* 500-599 服务器端错误。 指明客户端的请求正确，但是服务器因为自身原因无法完成请求。
* 100-199 信息提示。 这个系列的状态码只有2个，但是比较费解，会专门单独的做出解释。

#### 首部字段
```javascript
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache/2.2.14 (Win32)
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
Content-Length: 88
Content-Type: text/html
Connection: Closed
```
比如Server: Apache/2.2.14 (Win32)指示服务器使用的是Apache Server。而Content-Type: text/html 指明消息主体是html格式的资源。

#### 一个空行（CRLF)
指示头字段完成。

#### 可选的消息主体
案例中就是一个hello.htm文件的内容。这里响应主体有很多种格式，如果是gif的话会返回gif的二进制字节集合。


## 请求
### GET 
> GET方法用来获取URL指定的资源

#### 通过`Range头字段`指定指定获取文件的开始字节索引和结束字节索引

发出如下局部请求：
```javascript
GET /hello.txt HTTP/1.1
Range: bytes=0-2
```

服务器响应：
```javascript
HTTP/1.1 206 Partial Content
Content-Type: text/html
Content-Range: bytes 0-2/12
Content-Length: 3

hello
```
在一些视频的场景，比如用户点击到某个进度，我们可以直接从该进度去请求，而不是整一个视频都请求下来。
响应头的Content-Range: bytes 0-5/12指的是当前返回0-5，总长为12。

#### 配合 `ETag` 做缓存
服务端在响应头里面会给一个ETag标识，这个标识我们在开头的时候有说了是文件的唯一标识符。那么我们是不是可以这么做，在GET请求时使用条件获取头部字段，如果服务端发现这个字段与文件的ETag标识匹配，则可以做缓存，否则发送新的文档过来。

发起请求: 
```javascript
GET /sample.html HTTP/1.1
Host: example.com
If-None-Match:  W/"16-FmHX0hamHjYkHeAP/7PfzA"
```

服务端响应:
```javascript
HTTP/1.1 304 Not Modified
X-Powered-By: Express
ETag: W/"16-dDTk/xb5lvRNBfrz6lE//HVBox8"
Date: Tue, 22 May 2018 12:08:23 GMT
Connection: keep-alive
```
这里我们的ETag与文件匹配上了，所以服务端直接响应 304 状态码(Not Modified) 

### HEAD
GET 方法和 HEAD方法区别是，HEAD请求， 响应消息主体不会返回
那么HEAD的场景一般在什么地方呢？ 一般可用于发起一个请求来获取服务端的资源大小，再分段获取。

### POST
我们平时大部分场景用的都是POST方法，因为POST方法不会把参数都丢在URL上，对请求的数据大小没有限制，请求也不会缓存。

### OPTIONS
OPTIONS方法主要是用来查询URL指定的资源所支持的方法列表

### PUT
PUT方法上面讲了是 对URL指定的资源进行创建，如果存在就修改他。

理论上这种需求直接用POST方法就完全够用了，比如创建订单POST /order/create
更新订单 POST /order/orderId

但是这里如果改成PUT创建的话，可以直接这样 PUT /order 更符合Restful API规范，如果换成前端的层面来说，就像div与header, nav, section, aside, footer这些更有语义化的标签。

### DELETE
删除某个资源

### CONNECT
在当前已经建立HTTP连接的情况下，CONNECT 方法用来告知代理服务器，客户端想要和服务器之间建立SSL连接。

客户端使用如下消息，通知代理服务器，去做一个连接到指定的服务器地址和端口:

```javascript
CONNECT example.com 443 HTTP/1.1
```
代理服务器随后提取CONNECT 方法指定的地址和端口（这里是 example.com 443 ），建立和此服务器的SSL连接，成功后随后通知客户端，需要的连接建立完毕：

```javascript
HTTP/1.1 OK
```
使用CONNECT发起请求，服务端不会去解析字段等操作，而是完完全全单纯的做转发，我们称之为透明代理。

所以理解 Connect 方法，需要对比和区分以下内容：
* Http 可以通过Connection：upgrade的方法升级到TLS。仅仅使用于无代理服务器的情况。
* Connect 方法是 http upgrade tls 的一个替代。针对有代理的情况。
* Connect 方法成功返回后，中间的http代理变成了透明的代理：不再使用HTTP协议解析数据包和修改数据包，而是简单的转发流量。


### TRACE
查询到目标资源经过的中间节点。

## 响应
### 2XX
>`2XX状态码` 均表示请求已经成功处理，并且对 HTTP 客户端有不同的动作指示，比如是否刷新当前页面内容等。

- 200 OK
- 204 No Content 请求处理成功，但是服务器没有提供消息主体 。比如使用DELETE请求情况下，如果服务成功完成，可以返回204 No Content。
- 205 Reset Content
此状态码告诉客户端请求已经成功执行。不同于204，它的意图是要告诉客户应该清除Form的内容或者刷新用户界面。204响应适合对一条记录做一系列的编辑；205更适合输入一系列的记录。
- 206 Partical Content 支持大文件的`分段下载`
用法：服务器如果支持分段下载就通过Accept-Ranges: bytes 的首部字段指示它是支持的。接下来，客户端就可以通过首部字段 Range 来指定要获取资源的范围。而服务器通过206 Partial Content指示获取成功、本次获取为部分内容、和本次获取部分资源的范围。

### 3XX
> `3XX状态码` 除了304 Not Modified 之外都是用于重定向的

* 300 Multiple Choices 客户端请求了实际指向多个资源的URL。
* 301 Moved Permanently 请求的 URL 已移走。
* 302 Found 请求的URL临时移走
* 303 See Other 客户端应该使用指定URL
* 307 Temporary Redirect 客户端应该临时定位到指定URL
- 304
当用户代理发起GET请求并设置了修改时间的前条件，而服务器发现被请求的资源并没有在给出的时间后被修改，就会返回这个状态码。这个状态码的存在是为了性能上的考量。不必传递用户代理有的、服务器也没有修改的资源。

### 4XX
> `4XX状态码`400系列响应消息都是用来由服务器告诉客户端，收到的请求是它无法处理的。

- 404 Not Found可以用来指示客户端请求的资源在服务器上根本没有.
- 412 Precondition Failed 客户端发起了条件请求，服务器发现这个请求中的其中一个条件并不成立，那么服务器就会用此错误码作为响应消息的状态码返回给客户端。
- 417 Expectation Failed 客户端在请求头内加入了Expect字段，这个字段要求的期望如果并不能被服务器支持，那么服务器会返回417 状态码给客户端。
案例：
```javascript
GET /todo/1 HTTP/1.1
Host: example.org
Content-Type:json
Expect: 100-continue

HTTP/1.1 417 Expectation Failed
```
- 403 Fobidden 服务器禁止提供资源来响应它，尽管客户端的请求是有效的

### 5XX
>`5XX状态码`不要怀疑不是客户端不是用户的错，是服务器的错。

- 500 Internal Server Error 服务器遇到了一个妨碍它提供服务的错误
- 503 Service Unavailable 说明服务器现在无法提供服务

### 100
>`100状态码`当客户端发送 Expect:100-Continue时, 服务端可以响应 100 Continue 为允许，或者不许可

(417 Expectation Failed) 。100 Continue 状态码通知客户端可以继续发送请求。
在发送大文件之前，客户端可以首先发出询问，要是在服务器不接受大文件的话，服务器就可以直接拒绝继续。否则，当发现不符合条件的时候，实体内容这时候可能已经在传递了。这可是在浪费带宽了。

### Cache-Control
一般我们在服务端响应可以加上这个响应头Cache-Control: max-age=2592000 来让浏览器把这个资源缓存下来，缓存的时间为2592000既30天，也可以在时间后面加上个public来指示该响应可以被任何中间人（cdn,代理)缓存。

## 消息主体
> 无论是请求还是响应的东西都有一个可选的消息主体(message-body)

我们看看每个字段的含义：
```javascript
* Content-Type 实体中所承载对象的类型。
* Content-Length 所传送实体主体的长度或大小。
* Content-Language 与所传送对象最相配的人类语言。
* Content-Encoding 对象数据所做的压缩格式。
* Content-Location 一个备用位置，请求时可通过它获得对象。
* Content-Range 说明它是整体的哪个部分。
* Content-MD5 实体主体内容的校验和。
* Last-Modified 所传输内容在服务器上创建或最后修改的日期时间。
* Expires 实体数据将要失效的日期时间。
* Allow 该资源所允许的各种请求方法，例如，GET 和 HEAD。
* ETag 这份文档的唯一验证码。
```
消息主体可以放置静态文件内容、动态生成内容、还可以放置压缩后的动态和静态内容。整个内容可以一次传输完毕，或者分成多块传输。如果有必要，在内容后，还可以放置拖挂——一些只有内容传递完毕才能够知道的首部字段值。

## 客户连接
> `Cookie`就是用于标识的一个字符串。它会对每个首次来访的客户发一个不同的字符串，客户端会存下此字符串.可以在接下来的会话访问过程中提交此字符串给服务器来亮明身份。这样服务器就知道此次访问时之前的是哪个客户端了。

比如B首次访问，服务器响应：
```javascript
HTTP/1.1 200 OK
Set-Cookie:id=26

这是你的首访
```


B第二次访问时，服务器响应:
```javascript
HTTP/1.1 200 OK
Set-Cookie:id=26//这里可以用cookie或者setcookie

欢迎你第二次访问，你的号牌为26
```


END~话说感觉这本书很简洁~~但是之前听过的网络传输七层协议这里没提到有空再补上~（逃