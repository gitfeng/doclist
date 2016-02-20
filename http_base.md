---
title:  HTTP 协议
categories: 技术
date: 2016-02-20 16:20:01
tags: [http]
description: http协议介绍，http header字段说明，http 状态码说明

---

## HTTP 协议介绍

HTTP（HyperText Transport Protocol）是超文本传输协议的缩写，它用于传送WWW方式的数据，关于HTTP协议的详细内容请参考RFC2616。HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求，请求头包含请求的方法、URL、协议版本、以及包含请求修饰符、客户信息和内容的类似于MIME的消息结构。服务器以一个状态行作为响应，响应的内容包括消息协议的版本，成功或者错误编码加上包含服务器信息、实体元信息以及可能的实体内容。
通常HTTP消息包括客户机向服务器的请求消息和服务器向客户机的响应消息。这两种类型的消息由一个起始行，一个或者多个头域，一个指示头域结束的空行和可选的消息体组成。HTTP的头域包括通用头，请求头，响应头和实体头四个部分。每个头域由一个域名，冒号（:）和域值三部分组成。域名是大小写无关的，域值前可以添加任何数量的空格符，头域可以被扩展为多行，在每行开始处，使用至少一个空格或制表符。

## 通用头域
通用头域包含请求和响应消息都支持的头域，通用头域包含Cache-Control、Connection、Date、Pragma、Transfer-Encoding、Upgrade、Via。对通用头域的扩展要求通讯双方都支持此扩展，如果存在不支持的通用头域，一般将会作为实体头域处理。下面简单介绍几个在UPnP消息中使用的通用头域：

1. Cache-Control头域
Cache-Control指定请求和响应遵循的缓存机制。在请求消息或响应消息中设置Cache-Control并不会修改另一个消息处理过程中的缓存处理过程。请求时的缓存指令包括no-cache、no-store、max-age、max-stale、min-fresh、only-if-cached，响应消息中的指令包括public、private、no-cache、no-store、no-transform、must-revalidate、proxy-revalidate、max-age。各个消息中的指令含义如下：
	- Public指示响应可被任何缓存区缓存。
	- Private指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效。
	- no-cache指示请求或响应消息不能缓存
	- no-store用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存。
	- max-age指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。
	- min-fresh指示客户机可以接收响应时间小于当前时间加上指定时间的响应。
	- max-stale指示客户机可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。

2. HTTP connection: Keep-Alive
Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。市场上的大部分Web服务器，包括iPlanet、IIS和Apache，都支持HTTP Keep-Alive。对于提供静态内容的网站来说，这个功能通常很有用。但是，对于负担较重的网站来说，这里存在另外一个问题：虽然为客户保留打开的连接有一定的好处，但它同样影响了性能，因为在处理暂停期间，本来可以释放的资源仍旧被占用。当Web服务器和应用服务器在同一台机器上运行时，Keep-Alive功能对资源利用的影响尤其突出。
KeepAliveTime 值控制 TCP/IP 尝试验证空闲连接是否完好的频率。如果这段时间内没有活动，则会发送保持活动信号。如果网络工作正常，而且接收方是活动的，它就会响应。如果需要对丢失接收方敏感，换句话说，需要更快地发现丢失了接收方，请考虑减小这个值。如果长期不活动的空闲连接出现次数较多，而丢失接收方的情况出现较少，您可能会要提高该值以减少开销。缺省情况下，如果空闲连接 7200000 毫秒（2 小时）内没有活动，Windows 就发送保持活动的消息。通常，1800000 毫秒是首选值，从而一半的已关闭连接会在 30 分钟内被检测到。 KeepAliveInterval 值定义了如果未从接收方收到保持活动消息的响应，TCP/IP 重复发送保持活动信号的频率。当连续发送保持活动信号、但未收到响应的次数超出 TcpMaxDataRetransmissions 的值时，会放弃该连接。如果期望较长的响应时间，您可能需要提高该值以减少开销。如果需要减少花在验证接收方是否已丢失上的时间，请考虑减小该值或 TcpMaxDataRetransmissions 值。缺省情况下，在未收到响应而重新发送保持活动的消息之前，Windows 会等待 1000 毫秒（1 秒）。 KeepAliveTime 根据你的需要设置就行，比如10分钟，注意要转换成MS。 XXX代表这个间隔值得大小。

2. Date头域
Date头域表示消息发送的时间，时间的描述格式由rfc822定义。例如，Date:Mon,31Dec200104:25:57GMT。Date描述的时间表示世界标准时，换算成本地时间，需要知道用户所在的时区。

3. Pragma头域
Pragma头域用来包含实现特定的指令，最常用的是Pragma:no-cache。在HTTP/1.1协议中，它的含义和Cache-Control:no-cache相同。

### 请求消息

请求消息的第一行为下面的格式：
'{Method} {Request-URI} {HTTP-Version}CRLF'

{Method}表示对于Request-URI完成的方法，这个字段是大小写敏感的，包括OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE。方法GET和HEAD应该被所有的通用WEB服务器支持，其他所有方法的实现是可选的。GET方法取回由Request-URI标识的信息。HEAD方法也是取回由Request-URI标识的信息，只是可以在响应时，不返回消息体。POST方法可以请求服务器接收包含在请求中的实体信息，可以用于提交表单，向新闻组、BBS、邮件群组和数据库发送消息。
Request-URI遵循URI格式，在此字段为星号（*）时，说明请求并不用于某个特定的资源地址，而是用于服务器本身。HTTP-Version表示支持的HTTP版本，例如为HTTP/1.1。CRLF表示换行回车符。请求头域允许客户端向服务器传递关于请求或者关于客户机的附加信息。
请求头域可能包含下列字段Accept、Accept-Charset、Accept-Encoding、Accept-Language、Authorization、From、Host、If-Modified-Since、If-Match、If-None-Match、If-Range、If-Range、If-Unmodified-Since、Max-Forwards、Proxy-Authorization、Range、Referer、User-Agent。对请求头域的扩展要求通讯双方都支持，如果存在不支持的请求头域，一般将会作为实体头域处理。

1. Host头域
Host头域指定请求资源的Intenet主机和端口号，必须表示请求url的原始服务器或网关的位置。HTTP/1.1请求必须包含主机头域，否则系统会以400状态码返回。
2. Referer头域
Referer头域允许客户端指定请求uri的源资源地址，这可以允许服务器生成回退链表，可用来登陆、优化cache等。他也允许废除的或错误的连接由于维护的目的被追踪。如果请求的uri没有自己的uri地址，Referer不能被发送。如果指定的是部分uri地址，则此地址应该是一个相对地址。
3. Range头域
Range头域可以请求实体的一个或者多个子范围。例如，
表示头500个字节：bytes=0-499
表示第二个500字节：bytes=500-999
表示最后500个字节：bytes=-500
表示500字节以后的范围：bytes=500-
第一个和最后一个字节：bytes=0-0,-1
同时指定几个范围：bytes=500-600,601-999
但是服务器可以忽略此请求头，如果无条件GET包含Range请求头，响应会以状态码206（PartialContent）返回而不是以200（OK）。
4. User-Agent头域
User-Agent头域的内容包含发出请求的用户信息。

### 响应消息

响应消息的第一行为下面的格式：
'{HTTP-Version} {Status-Code} {Reason-Phrase}CRLF'

HTTP-Version表示支持的HTTP版本，例如为HTTP/1.1。Status-Code是一个三个数字的结果代码。
Reason-Phrase给Status-Code提供一个简单的文本描述。Status-Code主要用于机器自动识别，Reason-Phrase主要用于帮助用户理解。Status-Code的第一个数字定义响应的类别，后两个数字没有分类的作用。第一个数字可能取5个不同的值：
1xx:信息响应类，表示接收到请求并且继续处理
2xx:处理成功响应类，表示动作被成功接收、理解和接受
3xx:重定向响应类，为了完成指定的动作，必须接受进一步处理
4xx:客户端错误，客户请求包含语法错误或者是不能正确执行
5xx:服务端错误，服务器不能正确执行一个正确的请求
响应头域允许服务器传递不能放在状态行的附加信息，这些域主要描述服务器的信息和Request-URI进一步的信息。响应头域包含Age、Location、Proxy-Authenticate、Public、Retry-After、Server、Vary、Warning、WWW-Authenticate。对响应头域的扩展要求通讯双方都支持，如果存在不支持的响应头域，一般将会作为实体头域处理。

### 实体信息

请求消息和响应消息都可以包含实体信息，实体信息一般由实体头域和实体组成。实体头域包含关于实体的原信息，实体头包括Allow、Content-Base、Content-Encoding、Content-Language、Content-Length、Content-Location、Content-MD5、Content-Range、Content-Type、Etag、Expires、Last-Modified、extension-header。extension-header允许客户端定义新的实体头，但是这些域可能无法被接受方识别。实体可以是一个经过编码的字节流，它的编码方式由Content-Encoding或Content-Type定义，它的长度由Content-Length或Content-Range定义。

1. Content-Type实体头
Content-Type实体头用于向接收方指示实体的介质类型，指定HEAD方法送到接收方的实体介质类型，或GET方法发送的请求介质类型
2. Content-Range实体头
Content-Range实体头用于指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式：
Content-Range:bytes-unitSPfirst-byte-pos-last-byte-pos/entity-legth
例如，传送头500个字节次字段的形式：Content-Range:bytes0-499/1234如果一个http消息包含此节（例如，对范围请求的响应或对一系列范围的重叠请求），Content-Range表示传送的范围，Content-Length表示实际传送的字节数。
3. Last-modified实体头
Last-modified实体头指定服务器上保存内容的最后修订时间。
例如，传送头500个字节次字段的形式：Content-Range:bytes0-499/1234如果一个http消息包含此节（例如，对范围请求的响应或对一系列范围的重叠请求），Content-Range表示传送的范围，Content-Length表示实际传送的字节数。

## 工作原理
一次HTTP操作称为一个事务，其工作过程可分为四步：
首先客户机与服务器需要建立连接。只要单击某个超级链接，HTTP的工作就开始了。
建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。
服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。
客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。
如果在以上过程中的某一步出现错误，那么产生错误的信息将返回到客户端，由显示屏输出。对于用户来说，这些过程是由HTTP自己完成的，用户只要用鼠标点击，等待信息显示就可以了。
许多HTTP通讯是由一个用户代理初始化的并且包括一个申请在源服务器上资源的请求。最简单的情况可能是在用户代理和服务器之间通过一个单独的连接来完成。在Internet上，HTTP通讯通常发生在TCP/IP连接之上。缺省端口是TCP 80，但其它的端口也是可用的。但这并不预示着HTTP协议在Internet或其它网络的其它协议之上才能完成。HTTP只预示着一个可靠的传输。
这个过程就好像我们打电话订货一样，我们可以打电话给商家，告诉他我们需要什么规格的商品，然后商家再告诉我们什么商品有货，什么商品缺货。这些，我们是通过电话线用电话联系（HTTP是通过TCP/IP），当然我们也可以通过传真，只要商家那边也有传真。

## 状态消息

### 1xx:信息

| 消息 | 描述 |
| ---  | -- |
|100 Continue|服务器仅接收到部分请求，但是一旦服务器并没有拒绝该请求，客户端应该继续发送其余的请求。|
|101 Switching Protocols |服务器转换协议：服务器将遵从客户的请求转换到另外一种协议。|

### 2xx:成功

| 消息 | 描述 |
| ---  | -- |
| 200 OK |请求成功（其后是对GET和POST请求的应答文档。）|
| 201 Created | 请求被创建完成，同时新的资源被创建。|
|202 Accepted | 供处理的请求已被接受，但是处理未完成。|
| 203 Non-authoritative Information | 文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝。|
| 204 No Content | 没有新文档。浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而Servlet可以确定用户文档足够新，这个状态代码是很有用的。|
|205 Reset Content|没有新文档。但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容。|
| 206 Partial Content|客户发送了一个带有Range头的GET请求，服务器完成了它。|

### 3xx:重定向

| 消息 | 描述 |
|---|--|
| 300 Multiple Choices |多重选择。链接列表。用户可以选择某链接到达目的地。最多允许五个地址。|
|301 Moved Permanently|所请求的页面已经转移至新的url。|
|302 Found|所请求的页面已经临时转移至新的url。|
|303 See Other|所请求的页面可在别的url下被找到。|
|304 Not Modified|未按预期修改文档。客户端有缓冲的文档并发出了一个条件性的请求<br />（一般是提供If-Modified-Since头表示客户只想比指定日期更新的文档）。<br />服务器告诉客户，原来缓冲的文档还可以继续使用。|
|305 Use Proxy|客户请求的文档应该通过Location头所指明的代理服务器提取。|
|306 Unused|此代码被用于前一版本。目前已不再使用，但是代码依然被保留。|
|307 Temporary Redirect|被请求的页面已经临时移至新的url。|

### 4xx:客户端错误

| 消息 | 描述 |
| ---  | -- |
|400 Bad Request|服务器未能理解请求。|
|401 Unauthorized|被请求的页面需要用户名和密码。|
|401.1|登录失败。|
|401.2|服务器配置导致登录失败。|
|401.3|由于 ACL 对资源的限制而未获得授权。|
|401.4|筛选器授权失败。|
|401.5|ISAPI/CGI 应用程序授权失败。|
|401.7|访问被 Web 服务器上的 URL 授权策略拒绝。这个错误代码为 IIS 6.0 所专用。|
|402 Payment Required|此代码尚无法使用。|
|403 Forbidden|对被请求页面的访问被禁止。|
|403.1|执行访问被禁止。|
|403.2|读访问被禁止。|
|403.3|写访问被禁止。|
|403.4|要求 SSL。|
|403.5|要求 SSL 128。|
|403.6|IP 地址被拒绝。|
|403.7|要求客户端证书。|
|403.8|站点访问被拒绝。|
|403.9|用户数过多。|
|403.10|配置无效。|
|403.11|密码更改。|
|403.12|拒绝访问映射表。|
|403.13|客户端证书被吊销。|
|403.14|拒绝目录列表。|
|403.15|超出客户端访问许可。|
|403.16|客户端证书不受信任或无效。|
|403.17|客户端证书已过期或尚未生效。|
|403.18|在当前的应用程序池中不能执行所请求的 URL。这个错误代码为 IIS 6.0 所专用。|
|403.19|不能为这个应用程序池中的客户端执行 CGI。这个错误代码为 IIS 6.0 所专用。|
|403.20|Passport 登录失败。这个错误代码为 IIS 6.0 所专用。|
|404 Not Found|服务器无法找到被请求的页面。|
|404.0|（无）–没有找到文件或目录。|
|404.1|无法在所请求的端口上访问 Web 站点。|
|404.2|Web 服务扩展锁定策略阻止本请求。|
|404.3|MIME 映射策略阻止本请求。|
|405 Method Not Allowed|请求中指定的方法不被允许。|
|406 Not Acceptable|服务器生成的响应无法被客户端所接受。|
|407 Proxy Authentication Required|用户必须首先使用代理服务器进行验证，这样请求才会被处理。|
|408 Request Timeout|请求超出了服务器的等待时间。|
|409 Conflict|由于冲突，请求无法被完成。|
|410 Gone|被请求的页面不可用。|
|411 Length Required|"Content-Length" 未被定义。如果无此内容，服务器不会接受请求。|
|412 Precondition Failed|请求中的前提条件被服务器评估为失败。|
|413 Request Entity Too Large|由于所请求的实体的太大，服务器不会接受请求。|
|414 Request-url Too Long|由于url太长，服务器不会接受请求。当post请求被转换为带有很长的查询信息的get请求时，就会发生这种情况。|
|415 Unsupported Media Type|由于媒介类型不被支持，服务器不会接受请求。|
|416 Requested Range Not Satisfiable|服务器不能满足客户在请求中指定的Range头。|
|417 Expectation Failed|执行失败。|
|423|锁定的错误。|

### 5xx:服务器错误

| 消息 | 描述 |
| ---  | -- |
|500 Internal Server Error |请求未完成。服务器遇到不可预知的情况。 |
|500.12 |应用程序正忙于在 Web 服务器上重新启动。 |
|500.13|Web 服务器太忙。|
|500.15|不允许直接请求 Global.asa。|
|500.16|UNC 授权凭据不正确。这个错误代码为 IIS 6.0 所专用。|
|500.18|URL 授权存储不能打开。这个错误代码为 IIS 6.0 所专用。|
|500.100|内部 ASP 错误。|
|501 Not Implemented|请求未完成。服务器不支持所请求的功能。|
|502 Bad Gateway|请求未完成。服务器从上游服务器收到一个无效的响应。|
|502.1|CGI 应用程序超时。·|
|502.2|CGI 应用程序出错。|
|503 Service Unavailable|请求未完成。服务器临时过载或当机。|
|504 Gateway Timeout|网关超时。|
|505 HTTP Version Not Supported|服务器不支持请求中指明的HTTP协议版本。|

## 参考文档

原文英文版
RFC 1945 - Hypertext Transfer Protocol -- HTTP/1.0
http://www.w3.org/Protocols/rfc1945/rfc1945
http://www.faqs.org/rfcs/rfc1945.html
RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1
http://www.w3.org/Protocols/rfc2616/rfc2616
http://www.w3.org/Protocols/rfc2616/rfc2616.html
http://www.faqs.org/rfcs/rfc2616.html
RFC 1945 - Hypertext Transfer Protocol -- HTTP/1.0 中文版
http://man.chinaunix.net/develop/rfc/RFC1945.txt