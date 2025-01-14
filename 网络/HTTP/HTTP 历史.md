[![返回目录](https://parg.co/Udx)](https://parg.co/UdT)

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/12/3/http.jpg)

# HTTP 概述: HTTP/1.1 与 HTTP/2

![](https://i.postimg.cc/zDL2K4Ns/image.png)

随着网络技术的发展，1999 年设计的 HTTP/1.1 已经不能满足需求，所以 Google 在 2009 年设计了基于 TCP 的 SPDY，后来 SPDY 的开发组推动 SPDY 成为正式标准，不过最终没能通过。不过 SPDY 的开发组全程参与了 HTTP/2 的制定过程，参考了 SPDY 的很多设计，因此一般认为 SPDY 就是 HTTP/2 的前身。无论 SPDY 还是 HTTP/2，都是基于 TCP 的，TCP 与 UDP 相比效率上存在天然的劣势，所以 2013 年 Google 开发了基于 UDP 的名为 QUIC 的传输层协议，QUIC 全称 Quick UDP Internet Connections，希望它能替代 TCP，使得网页传输更加高效。后经提议，互联网工程任务组正式将基于 QUIC 协议的 HTTP （HTTP over QUIC）重命名为 HTTP/3。

# HTTP

HTTP(HyperTextTransferProtocol)是超文本传输协议的缩写，它用于传送 WWW 方式的数据，关于 HTTP 协议的详细内容请参考 RFC2616。HTTP 协议采用了请求 / 响应模型。客户端向服务器发送一个请求，请求头包含请求的方法、URI 、协议版本、以及包含请求修饰符、客户 信息和内容的类似于 MIME 的消息结构。服务器以一个状态行作为响应，相应的内容包括消息协议的版本，成功或者错误编码加上包含服务器信息、实体元信息以及可能的实体内容。 HTTP 是一种无状态性的协议。这是因为此种协议不要求浏览器在每次请求中标明它自己的身份，并且浏览器以及服务器之间并没有保持一个持久性的连接用于多个页面之间的访问。当一个用户访问一个站点的时候，用户的浏览器发送一个 HTTP 请求到服务器，服务器返回给浏览器一个 HTTP 响应。其实很简单的一个概念，客户端一个请求，服务器端一个回复，这就是整个基于 HTTP 协议的通讯过程。

## 消息格式

通常 HTTP 消息包括客户机向服务器的请求消息和服务器向客户机的响应消息。这两种类型的消息由一个起始行，一个或者多个头域，一个只是头域结束的空行和可选的消息体组成。HTTP 的头域包括**通用头，请求头，响应头和实体头**四个部分。每个头域由一个域名，冒号(: )和域值三部分组成。域名是大小写无关的，域值前可以添加任何数量的空格符，头域可以被扩展为多行，在每行开始处，使用至少一个空格或制表符。

当用户访问 HTTP://example.com 这个域名的时候，浏览器就会自动和服务器建立 tcp/ip 连接，然后发送 HTTP 请求到 example.com 的服务器的 80 端口。该个请求的语法如下所示：

```
GET / HTTP/1.1
Host: example.org
```

以上第一行叫做请求行，第二个参数 ( 一个反斜线在这个例子中 ) 表示所请求资源的路径。反斜线代表了根目录 ; 服务器会转换这个根目录为服务器文件系统中的一个具体目录。

Apache 的用户常用 DocumentRoot 这个命令来设置这个文档根路径。如果请求的 url 是 HTTP://example.org/path/to/script.php, 那么请求的路径就是 /path/to/script.php。假如 document root 被定义为 usr/lcoal/apache/htdocs 的话 , 整个请求的资源路径就是 /usr/local/apache/htdocs/path/to/script.php。

第二行描述的是 HTTP 头部的语法。在这个例子中的头部是 Host, 它标识了浏览器希望获取资源的域名主机。还有很多其它的请求头部可以包含在 HTTP 请求中，比如 user-Agent 头部，在 php 可以通过 \$\_SERVER['HTTP_USER_AGENT']获取请求中所携带的这个头部信息。

# HTTP 前世今生

## HTTP 0.9

最早版本是 1991 年发布的 0.9 版。该版本极其简单，只有一个命令`GET`。

```
GET /index.html
```

上面命令表示，TCP 连接(connection )建立后，客户端向服务器请求(request )网页`index.html`。协议规定，服务器只能回应 HTML 格式的字符串，不能回应别的格式。

```
<html>
  <body>Hello World</body>
 </html>
```

服务器发送完毕，就关闭 TCP 连接。

## HTTP 1.0

1996 年 5 月，HTTP/1.0 版本发布，内容大大增加。首先，任何格式的内容都可以发送。这使得互联网不仅可以传输文字，还能传输图像、视频、二进制文件。这为互联网的大发展奠定了基础。其次，除了`GET`命令，还引入了`POST`命令和`HEAD`命令，丰富了浏览器与服务器的互动手段。再次，HTTP 请求和回应的格式也变了。除了数据部分，每次通信都必须包括头信息(HTTP header )，用来描述一些元数据。其他的新增功能还包括状态码(status code )、多字符集支持、多部分发送(multi-part type )、权限(authorization )、缓存(cache )、内容编码(content encoding )等。

### 短暂连接的缺陷

HTTP 1.0 规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个 TCP 连接，服务器完成请求处理后立即断开 TCP 连接，服务器不跟踪每个客户也不记录过去的请求。但是，这也造成了一些性能上的缺陷，例如，一个包含有许多图像的网页文件中并没有包含真正的图像数据内容，而只是指明了这些图像的 URL 地址，当 WEB 浏览器访问这个网页文件时，浏览器首先要发出针对该网页文件的请求，当浏览器解析 WEB 服务器返回的该网页文档中的 HTML 内容时，发现其中的<img>图像标签后，浏览器将根据<img>标签中的 src 属性所指定的 URL 地址再次向服务器发出下载图像数据的请求: ![](http://p.blog.csdn.net/images/p_blog_csdn_net/zhangxiaoxiang/i5.jpg) 显然，访问一个包含有许多图像的网页文件的整个过程包含了多次请求和响应，每次请求和响应都需要建立一个单独的连接，每次连接只是传输一个文档和图像，上一 次和下一次请求完全分离。即使图像文件都很小，但是客户端和服务器端每次建立和关闭连接却是一个相对比较费时的过程，并且会严重影响客户机和服务器的性 能。当一个网页文件中包含 Applet，JavaScript 文件，CSS 文件等内容时，也会出现类似上述的情况。

## HTTP 1.1

### 持久连接

在 HTTP1.0 中，每对 Request/Response 都使用一个新的连接。HTTP 1.1 则支持持久连接 Persistent Connection, 并且默认使用 persistent connection. 在同一个 tcp 的连接中可以传送多个 HTTP 请求和响应 . 多个请求和响应可以重叠，多个请求和响应可以同时进行 . 更加多的请求头和响应头 ( 比如 HTTP1.0 没有 host 的字段 ).HTTP 1.1 的持续连接，也需要增加新的请求头来帮助实现，例如，Connection 请求头的值为 Keep-Alive 时，客户端通知服务器返回本次请求结果后保持连接；Connection 请求头的值为 close 时，客户端通知服务器返回本次请求结果后关闭连接。HTTP 1.1 还提供了与身份认证、状态管理和 Cache 缓存等机制相关的请求头和响应头。HTTP 1.0 规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个 TCP 连接，服务器完成请求处理后立即断开 TCP 连接，服务器不跟踪 每个客户也不记录过去的请求。此外，由于大多数网页的流量都比较小，一次 TCP 连接很少能通过 slow-start 区，不利于提高带宽利用率。 ![](http://p.blog.csdn.net/images/p_blog_csdn_net/zhangxiaoxiang/i6.jpg)

### 管道机制

1.1 版还引入了管道机制(pipelining )，即在同一个 TCP 连接里面，客户端可以同时发送多个请求。这样就进一步改进了 HTTP 协议的效率。举例来说，客户端需要请求两个资源。以前的做法是，在同一个 TCP 连接里面，先发送 A 请求，然后等待服务器做出回应，收到后再发出 B 请求。管道机制则是允许浏览器同时发出 A 请求和 B 请求，但是服务器还是按照顺序，先回应 A 请求，完成后再回应 B 请求。

### 分块传输编码

**分块传输编码**(**Chunked transfer encoding**)是超文本传输协议(HTTP )中的一种数据传输机制，允许 HTTP 由应用服务器发送给客户端应用( 通常是网页浏览器)的数据可以分成多个部分。分块传输编码只在 HTTP 协议 1.1 版本(HTTP/1.1 )中提供。通常，HTTP 应答消息中发送的数据是整个发送的，Content-Length 消息头字段表示数据的长度。数据的长度很重要，因为客户端需要知道哪里是应答消息的结束，以及后续应答消息的开始。然而，使用分块传输编码，数据分解成一系列数据块，并以一个或多个块发送，这样服务器可以发送数据而不需要预先知道发送内容的总大小。通常数据块的大小是一致的，但也不总是这种情况。

HTTP 1.1 引入分块传输编码提供了以下几点好处：

1. HTTP 分块传输编码允许服务器为动态生成的内容维持 HTTP 持久连接。通常，持久链接需要服务器在开始发送消息体前发送 Content-Length 消息头字段，但是对于动态生成的内容来说，在内容创建完之前是不可知的。**[动态内容，content-length 无法预知]**
2. 分块传输编码允许服务器在最后发送消息头字段。对于那些头字段值在内容被生成之前无法知道的情形非常重要，例如消息的内容要使用散列进行签名，散列的结果通过 HTTP 消息头字段进行传输。没有分块传输编码时，服务器必须缓冲内容直到完成后计算头字段的值并在发送内容前发送这些头字段的值。**[散列签名，需缓冲完成才能计算]**
3. HTTP 服务器有时使用压缩 (gzip 或 deflate)以缩短传输花费的时间。分块传输编码可以用来分隔压缩对象的多个部分。在这种情况下，块不是分别压缩的，而是整个负载进行压缩，压缩的输出使用本文描述的方案进行分块传输。在压缩的情形中，分块编码有利于一边进行压缩一边发送数据，而不是先完成压缩过程以得知压缩后数据的大小。**[gzip 压缩，压缩与传输同时进行]**

一般情况 HTTP 的 Header 包含 Content-Length 域来指明报文体的长度。有时候服务生成 HTTP 回应是无法确定消息大小的，比如大文件的下载，或者后台需要复杂的逻辑才能全部处理页面的请求，这时用需要实时生成消息长度，服务器一般使用 chunked 编码。在进行 Chunked 编码传输时，在回复消息的 Headers 有 transfer-coding 域值为 chunked，表示将用 chunked 编码传输内容。使用 chunked 编码的 Headers 如下(可以利用 FireFox 的 FireBug 插件或 HttpWatch 查看 Headers 信息)：

```
  　　Chunked-Body=*chunk
  　　　　　　　　　"0"CRLF
  　　　　　　　　　footer
  　　　　　　　　　CRLF
  　　chunk=chunk-size[chunk-ext]CRLF
  　　　　　　chunk-dataCRLF

  　　hex-no-zero=<HEXexcluding"0">

  　　chunk-size=hex-no-zero*HEX
  　　chunk-ext=*(";"chunk-ext-name["="chunk-ext-value])
  　　chunk-ext-name=token
  　　chunk-ext-val=token|quoted-string
  　　chunk-data=chunk-size(OCTET)


  　　footer=*entity-header
```

编码使用若干个 Chunk 组成，由一个标明长度为 0 的 chunk 结束，每个 Chunk 有两部分组成，第一部分是该 Chunk 的长度和长度单位(一般不 写)，第二部分就是指定长度的内容，每个部分用 CRLF 隔开。在最后一个长度为 0 的 Chunk 中的内容是称为 footer 的内容，是一些没有写的头部内容。下面给出一个 Chunked 的解码过程(RFC 文档中有)：

```
  　　length:=0
  　　readchunk-size,chunk-ext(ifany)andCRLF
  　　while(chunk-size>0){
  　　readchunk-dataandCRLF
  　　appendchunk-datatoentity-body
  　　length:=length+chunk-size
  　　readchunk-sizeandCRLF
  　　}
  　　readentity-header
  　　while(entity-headernotempty){
  　　appendentity-headertoexistingheaderfields
  　　readentity-header
  　　}
  　　Content-Length:=length
  　　Remove"chunked"fromTransfer-Encoding
```

## HTTP 2

1. HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式。二进制格式在协议的解析和优化扩展上带来更多的优势和可能。
2. HTTP/2 对消息头采用 HPACK 进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。头压缩能够很好的解决该问题。
3. 多路复用，直白的说就是所有的请求都是通过一个 TCP 连接并发完成。HTTP/1.x 虽然通过 [pipeline](http://en.wikipedia.org/wiki/HTTP_pipelining) 也能并发请求，但是多个请求之间的响应会被[阻塞](http://en.wikipedia.org/wiki/Head-of-line_blocking)的，所以 [pipeline](http://en.wikipedia.org/wiki/HTTP_pipelining) 至今也没有被普及应用，而 HTTP/2 做到了真正的并发请求。同时，流还支持优先级和流量控制。
4. Server Push ：服务端能够更快的把资源推送给客户端。例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 再发送这些请求。当客户端需要的时候，它已经在客户端了。

### 二进制协议支持

HTTP/1.1 版的头信息肯定是文本(ASCII 编码)，数据体可以是文本，也可以是二进制。HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制，并且统称为 " 帧 "(frame )：头信息帧和数据帧。二进制协议的一个好处是，可以定义额外的帧。HTTP/2 定义了近十种帧，为将来的高级应用打好了基础。如果使用文本实现这种功能，解析数据将会变得非常麻烦，二进制解析则方便得多。

### 多工复用

HTTP/2 复用 TCP 连接，在一个连接里，客户端和浏览器都可以同时发送多个请求或回应，而且不用按照顺序一一对应，这样就避免了 " 队头堵塞 "。举例来说，在一个 TCP 连接里面，服务器同时收到了 A 请求和 B 请求，于是先回应 A 请求，结果发现处理过程非常耗时，于是就发送 A 请求已经处理好的部分， 接着回应 B 请求，完成后，再发送 A 请求剩下的部分。这样双向的、实时的通信，就叫做多工(Multiplexing )。

### 数据流

因为 HTTP/2 的数据包是不按顺序发送的，同一个连接里面连续的数据包，可能属于不同的回应。因此，必须要对数据包做标记，指出它属于哪个回应。HTTP/2 将每个请求或回应的所有数据包，称为一个数据流(stream )。每个数据流都有一个独一无二的编号。数据包发送的时候，都必须标记数据流 ID，用来区分它属于哪个数据流。另外还规定，客户端发出的数据流，ID 一律为奇数，服务器发出的，ID 为偶数。数据流发送到一半的时候，客户端和服务器都可以发送信号(RST_STREAM 帧)，取消这个数据流。1.1 版取消数据流的唯一方法，就是关闭 TCP 连接。这就是说，HTTP/2 可以取消某一次请求，同时保证 TCP 连接还打开着，可以被其他请求使用。客户端还可以指定数据流的优先级。优先级越高，服务器就会越早回应。

### 头信息压缩

HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如 Cookie 和 User Agent，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。HTTP/2 对这一点做了优化，引入了头信息压缩机制(header compression )。一方面，头信息使用 gzip 或 compress 压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就提高速度了。

### 服务器推送

HTTP/2 允许服务器未经请求，主动向客户端发送资源，这叫做服务器推送(server push )。常见场景是客户端请求一个网页，这个网页里面包含很多静态资源。正常情况下，客户端必须收到网页后，解析 HTML 源码，发现有静态资源，再发出静态资源请求。其实，服务器可以预期到客户端请求网页后，很可能会再请求静态资源，所以就主动把这些静态资源随着网页一起发给客户端了。

# HTTP General Header:HTTP 通用头

就整个网络资源传输而言，包括 message-header 和 message-body 两部分。首先传递 message-header，即**HTTP** **header**消息。HTTP header 消息通常被分为 4 个部分：general header, request header, response header, entity header 。但是这种分法就理解而言，感觉界限不太明确。根据维基百科对 HTTP header 内容的组织形式，大体分为 Request 和 Response 两部分。笔者在这里只是对于常见的协议头内容做一个列举，不同的设置会有不同的功能效果，会在下文中详细说明。本部分只介绍请求头的通用构成，具体的请求与响应参考各自章节。

> **注意每个 Header 的冒号后面有个空格**

通用头域包含请求和响应消息都支持的头域，通用头域包含 Cache-Control、 Connection 、 Date、Pragma 、 Transfer-Encoding、Upgrade 、 Via。对通用头域的扩展要求通讯双方都支持此扩展，如果存在不支持的通用头域，一般将会作为实体头域处理。

## Date

Date 头域表示消息发送的时间，时间的描述格式由 rfc822 定义。例如，Date:Mon,31Dec200104:25:57GMT 。 Date 描述的时间表示世界标准时，换算成本地时间，需要知道用户所在的时区。

## Pragma

Pragma 头域用来包含实现特定的指令，最常用的是 Pragma:no-cache。在 HTTP/1.1 协议中，它的含义和 Cache-Control:no-cache 相同。

## Entity

请求消息和响应消息都可以包含实体信息，实体信息一般由实体头域和实体组成。实体头域包含关于实体的原信息，实体头包括 Allow、Content- Base 、 Content-Encoding、Content-Language 、 Content-Length、Content-Location 、 Content-MD5、Content-Range 、 Content-Type、 Etag 、 Expires、Last-Modified 、 extension-header。extension-header 允许客户端定义新的实体 头，但是这些域可能无法未接受方识别。实体可以是一个经过编码的字节流，它的编码方式由 Content-Encoding 或 Content-Type 定 义，它的长度由 Content-Length 或 Content-Range 定义。

## Content-Type

Content-Type 实体头用于向接收方指示实体的介质类型，指定 HEAD 方法送到接收方的实体介质类型，或 GET 方法发送的请求介质类型 Content-Range 实体头。关于字符的编码，1.0 版规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。因此，服务器回应的时候，必须告诉客户端，数据是什么格式。Content-Range 实体头用于指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式：

```
Content-Range:bytes-unitSPfirst-byte-pos-last-byte-pos/entity-legth
```

Content-Type 表明信息类型，缺省值为 " text/plain"。它包含了主要类型(primary type )和次要类型(subtype )两个部分，两者之间用 "/" 分割。主要类型有 9 种，分别是 application、audio 、 example、image 、 message、model 、 multipart、text 、 video。每一种主要类型下面又有许多种次要类型，常见的有：

- text/plain ：纯文本，文件扩展名 .txt
- text/html：HTML 文本，文件扩展名 .htm 和 .html
- image/jpeg：jpeg 格式的图片，文件扩展名 .jpg
- image/gif：GIF 格式的图片，文件扩展名 .gif
- audio/x-wave：WAVE 格式的音频，文件扩展名 .wav
- audio/mpeg：MP3 格式的音频，文件扩展名 .mp3
- video/mpeg：MPEG 格式的视频，文件扩展名 .mpg
- application/zip：PK-ZIP 格式的压缩文件，文件扩展名 .zip

## Content-Length

TCP 1.0 中允许单个 TCP 连接可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是 Content-length 字段的作用，声明本次回应的数据长度。只有当浏览器使用持久 HTTP 连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStram，完成后查看其大小，然后把该值放入 Content-Length 头，最后通过 byteArrayStream.writeTo(response.getOutputStream() 发送内容。

## Content-Encoding

由于发送的数据可以是任何格式，因此可以把数据压缩后再发送。`Content-Encoding`字段说明数据的压缩方法。

> ```
> Content-Encoding: gzip
> Content-Encoding: compress
> Content-Encoding: deflate
> ```

客户端在请求时，用`Accept-Encoding`字段说明自己可以接受哪些压缩方法。

```
Accept-Encoding: gzip, deflate
```

# HTTP Lint

> [Lint for HTTP:HTTPolice](HTTP://www.tuicool.com/articles/yuaeyuj)

HTTPolice 是一个简单的基于命令行的对于 HTTP 请求格式规范进行检测的工具，可以直接使用`pip`命令进行安装 :

```
pip install HTTPolice
```

当我们使用 Google Chrome、Firefox 或者 Microsoft Edge 进行网络访问时，可以使用开发者工具导出某个 HAR 文件，这也就是 HTTP Lint 工具可以用来解析的文件。使用如下命令进行分析 :

```
$ httpolice -i har /path/to/file.har
------------ request: GET /1441/25776044114_0e5b9879a0_z.jpg------------ response: 200 OKC 1277 Obsolete 'X-' prefix in X-Photo-FarmC 1277 Obsolete 'X-' prefix in X-Photo-OriginE 1000 Malformed Expires headerE 1241 Date + Age is in the future
```

默认的 HTTPolice 以文本形式输出报告文本，如下所示

```
------------ request: PUT /articles/109226/
E 1000 Malformed If-Match header
C 1093 User-Agent contains no actual product
------------ response: 204 No Content
C 1110 204 response with no Date header
E 1221 Strict-Transport-Security without TLS
------------ request: POST /articles/109226/comments/
...
```

纯文本的方式可能比较难以理解，我们可以使用`-o html`选项来设置更详细的基于 HTML 风格的输出，譬如 :

```
$ httpolice -i har -o html /path/to/file.har >report.html
```

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/8/2/6BF80CC5-C64A-4753-8EBA-FDDEA87C0297.png)
