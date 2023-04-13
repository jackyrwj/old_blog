---

layout: post
title: " 网络协议 - HTTP协议详解"
date: 2023-4-13
tags: [计算机网络]
comments: true
author: jackyrwj
toc: true

---

## 1、HTTP 方法

客户端发送的 **请求报文** 第一行为请求行，包含了方法字段。

### [#](#get) GET

> 获取资源

当前网络请求中，绝大部分使用的是 GET 方法。

### [#](#head) HEAD

> 获取报文首部

和 GET 方法一样，但是不返回报文实体主体部分。

主要用于确认 URL 的有效性以及资源更新的日期时间等。

### [#](#post) POST

> 传输实体主体

POST 主要用来传输数据，而 GET 主要用来获取资源。

更多 POST 与 GET 的比较请见第八章。

### [#](#put) PUT

> 上传文件

由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。

```
PUT /new.html HTTP/1.1
Host: example.com
Content-type: text/html
Content-length: 16

<p>New File</p>
```

### [#](#patch) PATCH

> 对资源进行部分修改

PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改。

```
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

### [#](#delete) DELETE

> 删除文件

与 PUT 功能相反，并且同样不带验证机制。

```
DELETE /file.html HTTP/1.1
```

### [#](#options) OPTIONS

> 查询支持的方法

查询指定的 URL 能够支持的方法。

会返回 Allow: GET, POST, HEAD, OPTIONS 这样的内容。

### [#](#connect) CONNECT

> 要求在与代理服务器通信时建立隧道

使用 SSL(Secure Sockets Layer，安全套接层)和 TLS(Transport Layer Security，传输层安全)协议把通信内容加密后经网络隧道传输。

```
CONNECT www.example.com:443 HTTP/1.1
```

### [#](#trace) TRACE

> 追踪路径

服务器会将通信路径返回给客户端。

发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器就会减 1，当数值为 0 时就停止传输。

通常不会使用 TRACE，并且它容易受到 XST 攻击(Cross-Site Tracing，跨站追踪)。

## [#](#三、http-状态码) 二、HTTP 状态码

服务器返回的 **响应报文** 中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 | 类别                     | 原因短语          |
| --- | ---------------------- | ------------- |
| 1XX | Informational(信息性状态码)  | 接收的请求正在处理     |
| 2XX | Success(成功状态码)         | 请求正常处理完毕      |
| 3XX | Redirection(重定向状态码)    | 需要进行附加操作以完成请求 |
| 4XX | Client Error(客户端错误状态码) | 服务器无法处理请求     |
| 5XX | Server Error(服务器错误状态码) | 服务器处理请求出错     |

### [#](#_1xx-信息) 1XX 信息

- **100 Continue** : 表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

### [#](#_2xx-成功) 2XX 成功

- **200 OK**

- **204 No Content** : 请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

- **206 Partial Content** : 表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

### [#](#_3xx-重定向) 3XX 重定向

- **301 Moved Permanently** : 永久性重定向

- **302 Found** : 临时性重定向

- **303 See Other** : 和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

- 注: 虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

- **304 Not Modified** : 如果请求报文首部包含一些条件，例如: If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

- **307 Temporary Redirect** : 临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

### [#](#_4xx-客户端错误) 4XX 客户端错误

- **400 Bad Request** : 请求报文中存在语法错误。

- **401 Unauthorized** : 该状态码表示发送的请求需要有认证信息(BASIC 认证、DIGEST 认证)。如果之前已进行过一次请求，则表示用户认证失败。

- **403 Forbidden** : 请求被拒绝。

- **404 Not Found**

### [#](#_5xx-服务器错误) 5XX 服务器错误

- **500 Internal Server Error** : 服务器正在执行请求时发生错误。

- **503 Service Unavailable** : 服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

### 三、缓存机制

#### [#](#_1-优点) 1. 优点

- 缓解服务器压力；
- 降低客户端获取资源的延迟: 缓存通常位于内存中，读取缓存的速度更快。并且缓存在地理位置上也有可能比源服务器来得近，例如浏览器缓存。

#### [#](#_2-实现方法) 2. 实现方法

- 让代理服务器进行缓存；
- 让客户端浏览器进行缓存。

#### 3. Cache-Control

HTTP/1.1 通过 Cache-Control 首部字段来控制缓存。

**(一)禁止进行缓存**

no-store 指令规定不能对请求或响应的任何一部分进行缓存。

```
Cache-Control: no-store
```

**(二)强制确认缓存**

no-cache 指令规定缓存服务器需要先向源服务器验证缓存资源的有效性，只有当缓存资源有效才将能使用该缓存对客户端的请求进行响应。

```
Cache-Control: no-cache
```

**(三)私有缓存和公共缓存**

private 指令规定了将资源作为私有缓存，只能被单独用户所使用，一般存储在用户浏览器中。

```
Cache-Control: private
```

public 指令规定了将资源作为公共缓存，可以被多个用户所使用，一般存储在代理服务器中。

```
Cache-Control: public
```

**(四)缓存过期机制**

max-age 指令出现在请求报文中，并且缓存资源的缓存时间小于该指令指定的时间，那么就能接受该缓存。

max-age 指令出现在响应报文中，表示缓存资源在缓存服务器中保存的时间。

```
Cache-Control: max-age=31536000
```

Expires 首部字段也可以用于告知缓存服务器该资源什么时候会过期。

- 在 HTTP/1.1 中，会优先处理 max-age 指令；
- 在 HTTP/1.0 中，max-age 指令会被忽略掉。

```
Expires: Wed, 04 Jul 2012 08:26:05 GMT
```

[#](#%E5%8F%AF%E7%BC%93%E5%AD%98) 可缓存

如果要对响应进行缓存，需要满足以下条件:

- 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
- 响应报文的状态码是可缓存的，包括: 200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
- 响应报文的 Cache-Control 首部字段没有指定不进行缓存。

#### [#](#_4-%E7%BC%93%E5%AD%98%E9%AA%8C%E8%AF%81) 4. 缓存验证

需要先了解 ETag 首部字段的含义，它是资源的唯一标识。URL 不能唯一表示资源，例如 `http://www.google.com/` 有中文和英文两个资源，只有 ETag 才能对这两个资源进行唯一标识。

```
ETag: "82e22293907ce725faf67773957acd12"
```

可以将缓存资源的 ETag 值放入 If-None-Match 首部，服务器收到该请求后，判断缓存资源的 ETag 值和资源的最新 ETag 值是否一致，如果一致则表示缓存资源有效，返回 304 Not Modified。

```
If-None-Match: "82e22293907ce725faf67773957acd12"
```

Last-Modified 首部字段也可以用于缓存验证，它包含在源服务器发送的响应报文中，指示源服务器对资源的最后修改时间。但是它是一种弱校验器，因为只能精确到一秒，所以它通常作为 ETag 的备用方案。如果响应首部字段里含有这个信息，客户端可以在后续的请求中带上 If-Modified-Since 来验证缓存。服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回，状态码为 200 OK。如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的 304 Not Modified 响应。

```
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

```
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
```

## 四、GET 和 POST 比较

### [#](#作用) 作用

GET 用于获取资源，而 POST 用于传输实体主体。

### [#](#参数) 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具(Fiddler)查看。

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参考支持标准字符集。

```
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
```

```
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```

### [#](#安全) 安全

安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有: HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。

### [#](#幂等性) 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用(统计用途除外)。

所有的安全方法也都是幂等的。

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

GET /pageX HTTP/1.1 是幂等的，连续调用多次，客户端接收到的结果都是一样的:

```
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
```

POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录:

```
POST /add_row HTTP/1.1   -> Adds a 1nd row
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
```

DELETE /idX/delete HTTP/1.1 是幂等的，即便不同的请求接收到的状态码不一样:

```
DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
DELETE /idX/delete HTTP/1.1   -> Returns 404
```

### 五、 加密

#### [#](#_1-对称密钥加密) 1. 对称密钥加密

对称密钥加密(Symmetric-Key Encryption)，加密和解密使用同一密钥。

- 优点: 运算速度快；
- 缺点: 无法安全地将密钥传输给通信方。



#### [#](#_2-非对称密钥加密) 2.非对称密钥加密

非对称密钥加密，又称公开密钥加密(Public-Key Encryption)，加密和解密使用不同的密钥。

公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。

非对称密钥除了用来加密，还可以用来进行签名。因为私有密钥无法被其他人获取，因此通信发送方使用其私有密钥进行签名，通信接收方使用发送方的公开密钥对签名进行解密，就能判断这个签名是否正确。

- 优点: 可以更安全地将公开密钥传输给通信发送方；
- 缺点: 运算速度慢。



#### [#](#_3-https-采用的加密方式) 3. HTTPs 采用的加密方式

HTTPs 采用混合的加密机制，使用非对称密钥加密用于传输对称密钥来保证传输过程的安全性，之后使用对称密钥加密进行通信来保证通信过程的效率。(下图中的 Session Key 就是对称密钥)



### [#](#认证) 认证

通过使用 **证书** 来对通信方进行认证。

数字证书认证机构(CA，Certificate Authority)是客户端与服务器双方都可信赖的第三方机构。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起。

进行 HTTPs 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了。

通信开始时，客户端需要使用服务器的公开密钥将自己的私有密钥传输给服务器，之后再进行对称密钥加密。



![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230413162328.png)

### 六、HTTP/1.0 与 HTTP/1.1 的区别

- HTTP/1.1 默认是长连接

- HTTP/1.1 支持管线化处理

- HTTP/1.1 支持同时打开多个 TCP 连接

- HTTP/1.1 支持虚拟主机

- HTTP/1.1 新增状态码 100

- HTTP/1.1 支持分块传输编码

- HTTP/1.1 新增缓存处理指令 max-age

> 1、HTTP1.1默认开启长连接，在一个TCP连接上可以传送多个HTTP请求和响应。使用 TCP 长连接的方式改善了 HTTP/1.0 短连接造成的性能开销。
> 2、支持管道（pipeline）网络传输，只要第一个请求发出去了，不必等其回来，就可以发第二个请求出去，可以减少整体的响应时间。服务端无法主动push

## 七、HTTP/2.0

### [#](#http-1-x-缺陷) HTTP/1.x 缺陷

HTTP/1.x 实现简单是以牺牲性能为代价的:

- 客户端需要使用多个连接才能实现并发和缩短延迟；
- 不会压缩请求和响应首部，从而导致不必要的网络流量；
- 不支持有效的资源优先级，致使底层 TCP 连接的利用率低下。

### [#](#二进制分帧层) 二进制分帧层

HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230413162551.png)

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流(Stream)。

- 一个数据流都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
- 消息(Message)是与逻辑请求或响应消息对应的完整的一系列帧。
- 帧(Fram)是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230413162557.png)

### [#](#服务端推送) 服务端推送

HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 page.html 页面，服务端就把 script.js 和 style.css 等与之相关的资源一起发给客户端。

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230413162607.png)

### [#](#首部压缩) 首部压缩

HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。

HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。

不仅如此，HTTP/2.0 也使用 Huffman 编码对首部字段进行压缩。

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230413162610.png)
