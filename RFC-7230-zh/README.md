### 超文本传输协议（ HTTP/1.1 ）：报文语构和路由

#### 摘要

超文本传输协议是一种为分布式的、协作化的超文本信息系统而设计的无状态应用层协议。本篇文档提供了 HTTP 结构和其相关术语的概述，定义了 "http" 和 "https" 统一资源标识符（URI）规范及 HTTP/1.1 信息语构和解析要求，并且描述了在实现上的相关安全注意事项。

#### 目录

* [x] [1. 引言](#1-引言)
  * [x] [1.1 要求表示规范](#11-要求表示规范)
  * [x] [1.2 语法表示规范](#12-语法表示规范)
* [x] [2. 结构](#2-结构)
  * [x] [2.1 客户端 / 服务器端消息传递](#21-客户端--服务器端消息传递)
  * [x] [2.2 实现的多样性](#22-实现的多样性)
  * [x] [2.3 中介](#23-中介)
  * [x] [2.4 缓存](#24-缓存)
  * [x] [2.5 一致性和错误处理](#25-一致性和错误处理)
  * [x] [2.6 协议版本](#26-协议版本)
  * [x] [2.7 统一资源标识符](#27-统一资源标识符)
     * [ ] [2.7.1 HTTP URI 格式](#271-http-uri-格式)
     * [ ] [2.7.2 HTTPS URI 格式](#272-https-uri-格式)
     * [ ] [2.7.3 HTTP 和 HTTPS URI 的正规化和匹配](#273-http-和-https-uri-的正规化和匹配)
* [ ] [3. 消息格式](#3-消息格式)
  * [ ] [3.1 起始行](#31-起始行)
     * [ ] [3.1.1 请求行](#311-请求行)
     * [ ] [3.1.2 状态行](#312-状态行)
  * [ ] [3.2 头字段](#32-头字段)
     * [ ] [3.2.1 字段的可扩展性](#321-字段的可扩展性)
     * [ ] [3.2.2 字段顺序](#322-字段顺序)
     * [ ] [3.2.3 空白](#323-空白)
     * [ ] [3.2.4 字段解析](#324-字段解析)
     * [ ] [3.2.5 字段限制](#325-字段限制)
     * [ ] [3.2.6 字段值的组成成分](#326-字段值的组成成分)
  * [ ] [3.3 消息体](#33-消息体)
     * [ ] [3.3.1 Transfer-Encoding](#331-transfer-encoding)
     * [ ] [3.3.2 Content-Length](#332-content-length)
     * [ ] [3.3.3 消息体的长度](#333-消息体的长度)
  * [ ] [3.4 处理不完整的消息](#34-处理不完整的消息)
  * [ ] [3.5 消息解析的鲁棒性](#35-消息解析的鲁棒性)


### 1. 引言

超文本传输协议（ HTTP ）是一种基于请求/响应模型的无状态应用层协议，它利用其极具拓展性的语义和具有自我描述性的消息内容来与基于网络的超文本信息系统进行灵活的交互。

本篇文档是系统性定义 HTTP/1.1 规范的系列文档中的第一份：

- [RFC-7230](https://tools.ietf.org/html/rfc7230) 报文语构和路由
- [RFC-7231](https://tools.ietf.org/html/rfc7231) 语义和内容
- [RFC-7232](https://tools.ietf.org/html/rfc7232) 条件请求
- [RFC-7233](https://tools.ietf.org/html/rfc7233) 范围请求
- [RFC-7234](https://tools.ietf.org/html/rfc7234) 缓存
- [RFC-7235](https://tools.ietf.org/html/rfc7235) 认证

这份 HTTP/1.1 规范使得 [RFC-2616](https://tools.ietf.org/html/rfc2616) 及 [RFC-2145](https://tools.ietf.org/html/rfc2145) 中关于 HTTP/1.1 的部分被废弃。同时本规范还更新了之前在 [RFC-2817](https://tools.ietf.org/html/rfc2817) 中被定义的"用 CONNECT 方法建立网络隧道"，而且定义了原先在 [RFC-2818](https://tools.ietf.org/html/rfc2818) 中被非正式性描述的 "https" URI 方案。

HTTP 是一种用于信息系统的通用接口协议。它被设计出以暴露出独立于所提供资源类型的统一接口给客户端的方式来隐藏服务的实现细节。同样地，服务器也不需要知道客户端的任何目的：一次 HTTP 请求可以被视作独立的而非与特定的客户端类型或一连串预定的请求步骤相关。这些特性造就了一个可以高效应用于许多场景并且允许客户端和服务端的实现随着时间推移独立发展的协议。

HTTP 也同样被设计来作为与其它非 HTTP 信息系统交流的中间协议。HTTP 代理（proxy）和网关（gateway）以把其它各种协议的信息服务内容转化为可供 HTTP 客户端获取并操作的超文本格式的方法来让我们可以使用这些服务。

这种灵活性导致的一个后果是，我们不能用术语定义 HTTP 协议里接口后面具体发生的事。我们同样很难去定义通讯的语法、获取资源的目的及接受者的预期行为。如果一次通讯被认为是独立的，那么成功的行为应当反映在服务器所提供观察接口的相应改变上。然而，考虑到可能有多个客户端正在并行请求且相互之间可能存在干扰，我们不能指望这样的改变能在单次响应之后被观察到。
> 原文是"we cannot require that
   such changes be observable beyond the scope of a single response."，猜测是指，可能存在多个客户端请求同一个接口，所以在请求观察接口的时候，并不能确定刚才自己的请求是否为最后的一次请求（但为什么不用 Request ID 呢？欢迎大家讨论）。
   
本篇文档描述了在 HTTP 中被使用或者涉及的结构化的元素，定义了 "http" 和 "https" URI 规范，描述了总体的网络运转和连接管理，还定义了 HTTP 消息的构成及发送需求。我们的目的是定义所有独立于消息语义的关于 HTTP 消息处理的必要机制，从而为消息解析和消息转发中介定义完整的需求集。
   
#### 1.1 要求表示规范

本文档中的关键词 " MUST "、" MUST NOT "、" REQUIRED "、" SHALL "、" SHALL NOT "、" SHOULD "、" SHOULD NOT "、" RECOMMENDED "、" MAY "和" OPTIONAL " 应依照 [RFC2119](https://tools.ietf.org/html/rfc2119) 中的描述，本译文不会翻译这些关键词。

有关错误处理的一致性标准和注意事项会在 [Section 2.5](#25-一致性和错误处理) 中定义。
#### 1.2 语法表示规范

这份规范使用[扩充巴科斯范式（ ABNF ）](https://tools.ietf.org/html/rfc5234)作为主要的语法表示规范。另外，在 [Section 7]() 中定义了一个 ABNF 的列表扩展符号 ‘ # ’ 操作符（与 ‘ * ’ 操作符表示重复的形式类似），它允许紧凑定义的、由逗号分隔的列表。

[Appendix B]() 展示了所有本文档中收录的全由标准 ABNF 所表示语法 （原文"  Appendix B shows the collected grammar with all list operators
   expanded to standard ABNF notation.
 "，不知如何解释？）

下列核心规则都可供参考，如同 [RFC5234, Appendix B.1](https://tools.ietf.org/html/rfc5234#appendix-B.1) 中的定义：ALPHA（字母），CR（回车），CRLF（CR LF），CTL（control 键），DIGIT（数字 0-9），DQUOTE（双引号），HEXDIG（十六进制数 0-9/A-F/a-f），HTAB（水平 Tab），LF（换行），OCTET（字节），SP（空格）和 VCHAR（任何可见的 USASCII 字符）。

作为惯例，以 "obs-" 开头的 ABNF 规则表示已经废弃的语法规则。
### 2. 结构

HTTP 是为万维网（ WWW ）的建设而生的，为了支持世界范围的超文本信息系统的可拓展性需求，它也在不断地发展。HTTP 的大部分结构都反映在它的术语和语构上。

#### 2.1 客户端 / 服务器端消息传递

HTTP 是一种无状态的请求/响应协议，它通过一个可靠的传输层或会话层"连接"交换信息。一个 HTTP 客户端（ Client ）是一个为了发送一个或多个 HTTP 请求（ Request ）而与服务器建立连接的应用程序。一个 HTTP 服务器（ Server ）是一个接收 HTTP 请求并发送 HTTP 响应（ Response ）来提供服务的应用程序。 

术语 **"Client（客户端）"** 和 **"Server（服务器）"** 仅仅指代应用程序的某个特定连接。同一个程序可能同时在一些连接中扮演客户端而在另一些连接中扮演服务器。术语 **"User Agent（用户代理）"** 指代任何可以发起 HTTP 请求的的应用程序，包含但不仅限于：浏览器、爬虫、命令行工具、定制应用和手机 APP。术语 **"Origin Server（源服务器）"** 指代可以正确响应并返回目标资源的应用程序。术语  **"Sender（发送者）"** 和 **"Recipient（接收者）"** 分别指代任何发送或接收某条指定消息的实现。（原文" The
   terms "sender" and "recipient" refer to any implementation that sends
   or receives a given message, respectively. " 如何理解？）
   
HTTP 依赖统一资源标识符（ URI ）标准 [RFC 3986](https://tools.ietf.org/html/rfc3986) 来标识目标资源 [Section 5.1]() 和资源之间的关系。消息以一种跟互联网邮件 [RFC 5322](https://tools.ietf.org/html/rfc5322) 及多用途互联网邮件扩展（ MIME ）[RFC 2045](https://tools.ietf.org/html/rfc2045) 相似的格式被发送（[Appendix A of RFC 7231](https://tools.ietf.org/html/rfc7231#appendix-A) 查看 HTTP 与 MIME 消息的不同点）。

大多数的 HTTP 通讯都从一个为了展示某些被一个 URI 标识的资源而发起的取回请求（ GET ）开始。在最简单的情况下，这个通讯可能只通过用户代理（ UA ）和源服务器（ O ）之间的单个连接（===）就能完成。

```
    request >
UA ====================================== O
                            < response
```
一个客户端发送一个报文形式的 HTTP 请求给服务器，一条请求报文以包含 Method（ HTTP 方法）、URI 和 Protocol Version（协议版本）的[请求行](#311-请求行)开始，请求行之后是由请求修饰符、客户端信息和表示元数据组成的[头字段](#32-头字段)，头字段由一个空行标志结束，最后就是包含有效载荷的[消息主体](#33-消息体)（如果有的话）。

一个服务器响应一个客户端请求通过返回一个或多个 HTTP 响应报文，每个报文都以一个包含 Protocol Version（协议版本），一个成功或失败的状态码和一个可读的状态语句的[状态行](#312-状态行)开始，之后可能会跟着由服务器信息、资源元数据和表示元数据组成的[头字段](#32-头字段)，头字段由一个空行标志结束，最后就是包含有效载荷的[消息主体](#33-消息体)（如果有的话）。

一个连接可能可以被用于多次请求/响应通讯，如 [Section 6.3]() 中定义。

下面的例子演示了一个典型的[ GET 请求](https://tools.ietf.org/html/rfc7231#section-4.3.1)的报文交换，URI 是 "http://www.example.com/hello.txt"：

客户端请求：

```
GET /hello.txt HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi
```

服务器响应

```
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain

Hello World! My payload includes a trailing CRLF.
```
#### 2.2 实现的多样性

当考虑 HTTP 的设计时，很容易陷入一个思维陷阱——认为所有的用户代理都是通用浏览器且所有的源服务器都是大型公共网站。现实并非如此，常见的用户代理包括家用设施、音响、电子秤、固件更新脚本、命令行工具、手机 APP 和各种形状大小的通讯工具。同样，常见的 HTTP 源服务器包括家用自动化单元、可配置的网络组件、办公机器、自动化机器人、新闻提要、交通摄像头、广告选择器以及视频交流平台。

术语 **"User Agent"** 不仅仅指可以在请求时直接与代理软件进行交互的人类用户。在很多情况下，一个用户代理被安装或者配置在后台运行然后储存结果用于之后的查看（或者仅仅储存有意义的子集或者错误信息）。比如说爬虫，就是典型的，给一个起始 URI 和配置就可以在遍历爬取整个互联网时遵循指定的行为。

HTTP 实现的多样性意味着不是所有的用户代理在考虑一些安全或隐私问题时都可以提供给用户交互式的建议或者充足的警告。在少数需要报告错误给用户的情况下，把报告写入错误控制台或者日志文件也是可接受的。同样地，需要用户确认的自动化行为可能会在用户处理之前被提前选择配置、运行时选项或者简单的不安全行为黑名单所解决；确认并不意味着会弹出一个用户界面或者打断当前进程，如果用户早就做出了选择的话。

#### 2.3 中介

HTTP 允许使用 Intermediaries（中介） 来发起链式请求。Intermediaries 一共有三种形式：Proxy（代理），Gateway（网关）和 Tunnel（隧道）。在某些情况下，一个单独的 intermediary 可能会同时扮演 Origin Server、Proxy、Gateway 或 Tunnel，并基于每一条原始请求来切换行为。

```

            >             >             >             >
       UA =========== A =========== B =========== C =========== O
                  <             <             <             <
```

上面这幅图演示了在 User Agent 和 Origin Server 之间的三个 intermediaries（ A、B 和C ）。一个请求或响应报文将会穿过四个单独的连接。一些 HTTP 通信的选项可能只会在某些连接上生效，比如最近的连接、没有隧道的临近节点、链的终端节点、或者所有节点间的连接。虽然这幅图是线性的，但每一个参与节点都可能同时在其它的链里面扮演其它的角色。比如 B 可能从 A 之外的很多客户端接收请求，同时又向 C 之外的其它服务器转发请求，且同时在处理 A 的请求。同样地，之后的请求可能会通过连接中的不同路径，常见于基于动态配置的负载均衡。

术语 **"upstream（上游）"** 和  **"downstream（下游）"** 用于描述报文的流向：所有的报文都从上游流向下游。术语  **"inbound（入站）"** 和 **"outbound（出站）"** 用于描述请求的路由方向："inbound" 表示向着 Origin Server 方向，"outbound" 表示向着 User Agent 方向。

"Proxy" 是一种由客户端自己选择的消息转发代理，通常遵循本地配置文件的规则来接收关于一些 absolute URI 的请求并尝试以转换到 HTTP 接口的方式满足这些请求。一些转换是小菜一碟，比如仅仅改变 HTTP 请求的 URI，但在另一情况下，可能需要将流量转换到另一个完全不同的应用层协议。Proxies 经常用同一个 intermediary 代理一组 HTTP 请求，出于安全、注解服务或共享缓存的目的。也有一些 Porxy 被设计来对特定的消息进行处理，如同 [Section 5.7.2]() 中所描述的那样。

"Gateway（也称作 "reverse proxy(反向代理)"）对出站连接来说可看做 Origin Server，但它会把接收的请求转发到其他的 Server(s)。Gateways 通常用于封装历史遗留系统或者不被信任的信息服务、通过”加速器“缓存来提高服务器性能、或者给跑在多台机器上的 HTTP 服务分区或负载均衡。

所有适用于 Origin Server 的 HTTP 需求可以同样被用于 Gateway 的出站连接。而 Gateway 到后端 Server 的入站通讯则可以用任何协议，包括超出本文档规范的 HTTP 扩展。当然，想要与第三方 HTTP Server 进行互操作的 HTTP-to-HTTP Gateway 应当符合用户代理的需求（即 Gateway 请求其它 Server 的时候应该被视为 User Agent）。

"Tunnel" 在两个连接之间充当中继的作用，它只需要原封不动地转发流量。一旦被启用，tunnel 就不被视作 HTTP 通讯中的一部分，即使它可能发起了一个新的 HTTP 请求。只有在两端的连接都断开时，tunnel 才会停止工作并退出。Tunnels 通常通过一个中介来续接虚拟连接，比如通过一个共享防火墙代理用传输层安全协议（[TLS](https://tools.ietf.org/html/rfc5246)）建立可靠的通讯。
> 原文 "such as when
   Transport Layer Security (TLS, [RFC5246]) is used to establish
   confidential communication through a shared firewall proxy." 不是很能理解这个例子。
   
上文中提到的几种 intermediary 仅仅考虑了其本身参与 HTTP 通讯的行为。还存在 intermediaries 能参与网络协议栈更底层的行为，过滤或重定向 HTTP 流量而不会让发送者察觉，更不用发送者的许可。来自更底层网络中介的中间人攻击是无法在应用层上分辨的，这往往造成 HTTP 语义的误读并导致安全漏洞或交互性问题。

比如，一个 "[interception proxy](https://tools.ietf.org/html/rfc3040)"（也称为 "[transparent proxy](https://tools.ietf.org/html/rfc1919)" 或 "captive portal"）与 HTTP Proxy 的不同之处在于它不是由 client 自己选择的，但它会过滤或者重定向 TCP 端口 80 的出口流量（也可能是其他常见端口的流量）。Interception Proxies 在公共网络接入点是非常常见的，作为一种强制账号订阅以使用非本地网络服务的手段，或者用于企业内部防火墙强制施行网络使用政策。

HTTP 被定义为一种无状态协议，意味着每一个请求报文可以被独立地解析。很多实现都依赖 HTTP 的无状态特性以重用代理服务器的连接或者在多台服务器之间动态负载均衡。所以，一个 Server **MUST NOT** 认为来自同一个连接的两个请求是来自同一个 User Agent 的，除非这个连接是安全且是某个 User Agent 专用的。众所周知的是一些非标准的 HTTP 扩展（比如 [RFC-4559](https://tools.ietf.org/html/rfc4559)）违背了这项规定，从而导致了安全和交互性问题。

#### 2.4 缓存

"Cache" 指代储存之前的响应信息的本地仓库，以及控制、存取、删除储备信息的子系统。Cache 会缓存允许储存的响应以降低之后的响应时间和网络带宽消费，并得到与真实请求一致的结果。任何 Client 或者 Server **MAY** 采用一些缓存方案，但是当 Server 作为 Tunnel 的时候不能使用缓存。

缓存的作用是缩短了请求/响应链，如果链上的某个节点返回被缓存的响应。下图描述了实际的通讯链，在 B 缓存了之前 O 返回的（经由 C）的响应，而 UA 和 A 没有缓存的情况下：

```
            >             >
     UA =========== A =========== B - - - - - - C - - - - - - O
                <             <

```

一份响应是 "cacheable(可缓存的)" 意味着缓存系统可以拷贝并储存它的响应报文以用于接下来的请求。即使一份响应是可缓存的，依然可能存在来自 Client 或 Origin Server 的关于缓存使用场景的额外约束。HTTP 的缓存行为要求细则和可缓存响应定义在 [Section 2 of RFC7234](https://tools.ietf.org/html/rfc7234#section-2)。

关于缓存方案实施的结构和配置广泛存在于万维网和各大组织中。包括国际级的代理服务器使用缓存来节省跨洋带宽、通用的缓存内容广播或组播系统、打包缓存资源用于离线或高延迟环境等等。

#### 2.5 一致性和错误处理

这份规范要规定 HTTP 通讯中各参与角色的一致性标准。HTTP 的要求细节存在于 senders、recipients、clients、servers、user agents、intermediaries、origin servers、proxies、gateways、或者 caches 中，取决于要求中约束了哪些行为。额外的（社会的）要求则存在于具体实现、资源拥有者和协议元素注册，只要他们并非只存在于单次通讯。

动词 "generate" 和 "send" 在要求中的区别在于一个创建了协议元素而另一个仅仅转发了它收到的元素给下游。

如果一个具体实现遵循了它在 HTTP 通讯中所扮演角色的全部要求，那么可以认为它是符合规范的。

一致性包括协议元素的语构一致性和语义一致性。一个 sender **MUST NOT** 创建它认为是错误的协议元素。一个 sender **MUST NOT** 创建不符合相应 ABNF 规则语法的元素。在一个给定的报文内，一个 sender **MUST NOT** 创建只能由其它角色创建的协议元素或语构。

当一个已经收到的协议元素被解析时，recipient  **MUST** 能够解析出任何适用于 recipient 长度合理的值并匹配所有在 ABNF 规则中定义的语法。值得注意的是，即使是这样，一些被接收的协议元素依然可能无法解析。比如，一个 intermediary 转发一个报文时可能会把一个头字段解析为通用的字段名和字段值，但在它转发这个头字段之前并不会对字段值进行进一步的解析。

HTTP 对它的大部分协议元素都没有特别的长度限制，因为长度存在大范围的波动是合理的，这取决于使用场景和实现目的。于是，怎样的长度是合理的取决于 senders 和 recipients 的协商。此外，一些协议元素的通常意义上的合理长度在 HTTP 投入使用的二十年来也在不断变化，并且可以认定在未来还会继续发生变化。

但至少，recipient **MUST** 能够处理和它在其它报文中创建的同一协议元素一样长的内容。比如，一个 Origin Server 给它的某个资源创建了一个非常长的 URI references，那当它收到以此为目标的请求时，也应当可以处理它。

一个 recipient **MUST** 根据本规范来翻译其所接收协议元素的语义，包括这篇规范的扩展，除非 recipient 认为（凭借经验或者配置）sender 误解并错误地实现了这些语义。比如，一个 Origin Server 可能会无视所接收的 Accept-Encoding 头字段，如果对其 User-Agent 头字段的检查发现特定的内容编码会在其特定版本的 User Agent 上失败。

除非有什么特别的规定，否则，一个 recipient **MAY** 尝试从一个非法的结构中恢复可用的协议元素。HTTP 并没有定义特别的错误处理机制，除非这个错误对服务的安全性有着直接的冲击，因为不同的应用程序可能需要不同的错误处理策略。比如，一个 Web 浏览器可能想要正大光明地恢复一个定位头字段没有通过 ABNF 解析的 Response，但一个系统控制客户端可能认为任何形式的错误恢复都是危险的。

#### 2.6 协议版本

HTTP 使用一种 "<major>.<minor>" 的版本号形式去表明协议版本。这份规范定义了版本 "1.1"。这个协议版本仅作为 sender 一致性（本规范中规定的完整要求集合）的标识。

这个版本的 HTTP 版本信息存在于报文第一行的 HTTP-version 字段。HTTP-version 是区分大小写的。

```

     HTTP-version  = HTTP-name "/" DIGIT "." DIGIT
     HTTP-name     = %x48.54.54.50 ; "HTTP", case-sensitive
     
```

HTTP 版本号由两个以 "." 隔开的数字组成。第一个数字（ "major version" ）决定了 HTTP 报文的语构，而第二个数字（ "minor version" ）表示在这个主版本中最高位的副版本号，为了之后更进一步的通讯，副版本号往往用来说明 sender 的通讯标准。副版本号通知 sender 的通讯兼容性，即使 sender 只能使用这个协议版本向后兼容的子集，从而让 recipient 知道哪一些新特性能够在响应或者之后的请求中使用。

当一个 HTTP/1.1 的报文发给一个 [HTTP/1.0](https://tools.ietf.org/html/rfc1945) 或协议版本未知的 recipient，在这种所有新特性都会被忽略的情况下，HTTP/1.1 的报文会以兼容 HTTP/1.0 的形式被构建。这份规范列举了一些新特性的接受者版本要求，所以 sender 只会使用 recipient 兼容的特性，直到它通过配置或返回消息发现 recipient 已经支持 HTTP/1.1 了。

头字段的解析在同一主版本号的不同副版本号之间是没有区别的，即使 recipient 对缺省值的默认行为不同。除非特别规定，定义在 HTTP/1.1 中的头字段就是定义在 HTTP/1.x 中的头字段。尤其是 Host 和 Connection 头字段应当在所有的 HTTP/1.x 的实现中得到相应的实现，无论它们是否兼容 HTTP/1.1。

新的头字段能在不改变版本号的情况下被引入，只要他们定义的语义能允许他们被无法识别他们的 recipient 安全地忽略。在 [Section 3.2.1 中](#321-字段的可扩展性) 我们讨论了头字段的可拓展性。

处理 HTTP 报文的 intermediaries（即除 tunnels 之外的所有 intermediaries） **MUST** 在转发报文时附带它们自己的 HTTP-version。换句话来说，无论在接收消息时还说发送消息时，他们都必须确认报文的 HTTP-version 与自身的兼容性，而不应盲目地转发报文的 start-line。转发 HTTP 报文而不重写 HTTP-version 可能会导致通讯错误，当下游的 recipients 根据 sender 的协议版本去判断哪些特性是安全的。

一个 Client **SHOULD** 发送不超过 Server 能支持的最高主版本，同时与自身能兼容的最高版本相同版本的请求，如果这些都是已知的话。同时，一个 Client **MUST NOT** 发送与自身版本不兼容的请求。

一个 Server **MAY** 返回一个 HTTP/1.0 的响应，当它知道或怀疑该 client 错误地实现了 HTTP 规范或无法正确地处理更高版本的响应，比如 client 无法正确地解析版本号或者发现有一个 intermediary 盲目地转发了 HTTP-version 即使它并不兼容该报文的协议版本。这样的协议降级 **SHOULD NOT** 被实施，除非被 client 某些特定的属性触发，比如某些请求头字段（比如 User-Agent ）的值正好与某些已知错误的值相匹配。

HTTP 版本规范设计的目的是，只有在引入了不兼容的报文语构时才会升主版本号，而只有在引入了影响报文语义的变化或者为 sender 增加了额外的功能时，才会升副版本号。当然，[RFC-2068](https://tools.ietf.org/html/rfc2068) 到 [RFC-2616](https://tools.ietf.org/html/rfc2616) 之间的改变不足以升级副版本号，这份规范的版本重定义也特别地避免了影响版本号的的改变。

当一个 recipient 收到了与自身所实现协议的主版本号一致，但副版本号更高的 HTTP 报文，它 **SHOULD** 把改报文当做自身可兼容的最高版本来处理。recipient 可以假定拥有更高副版本号的报文对任何主版本号相同的实现都是充分向后兼任并可以被安全处理的，只要它自身并没有表明过它支持该版本的协议。

#### 2.7 统一资源标识符

统一资源标识符（ [URIs](https://tools.ietf.org/html/rfc3986) ）作为一种标识资源的手段被用于 HTTP 中。URI references 被用于表明请求目的、表示重定向、以及定义关系。

有关 "URI-reference", "absolute-URI", "relative-part", 'scheme', "authority", "port", "path-abempty", "segment", "query", 和 "fragment" 的定义均源于 [URI 通用语构](https://tools.ietf.org/html/rfc3986)。"absolute-path" 规则是为了可以包含一个非空路径的协议元素定义的（这个规则与允许使用空路径的 "path-abempty" 以及不允许以 "//" 开头路径的 "path-absolute" 规则略有不同）。"partial-URI" 规则是为了能包含无 fragment 的 relative-URI 的协议元素而定义的。


`URI-reference = <URI-reference, see`[RFC3986, Section 4.1](https://tools.ietf.org/html/rfc3986#section-4.1)\>

`absolute-URI  = <absolute-URI, see` [RFC3986, Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3)\>

`relative-part = <relative-part, see` [RFC3986, Section 4.2](https://tools.ietf.org/html/rfc3986#section-4.2)\>

`scheme        = <scheme, see` [RFC3986, Section 3.1](https://tools.ietf.org/html/rfc3986#section-3.1)\>

`authority     = <authority, see` [RFC3986, Section 3.2](https://tools.ietf.org/html/rfc3986#section-3.2)\>

`uri-host      = <host, see ` [RFC3986, Section 3.2.2](https://tools.ietf.org/html/rfc3986#section-3.2.2)\>

`port          = <port, see` [RFC3986, Section 3.2.3](https://tools.ietf.org/html/rfc3986#section-3.2.3)\>

`path-abempty  = <path-abempty, see` [RFC3986, Section 3.3](https://tools.ietf.org/html/rfc3986#section-3.3)\>

`segment       = <segment, see` [RFC3986, Section 3.3](https://tools.ietf.org/html/rfc3986#section-3.3)\>

`query         = <query, see` [RFC3986, Section 3.4](https://tools.ietf.org/html/rfc3986#section-3.4)\>

`fragment      = <fragment, see` [RFC3986, Section 3.5](https://tools.ietf.org/html/rfc3986#section-3.5)\>

`absolute-path = 1*( "/" segment )`

`partial-URI   = relative-part [ "?" query ]`

任何一个允许 URI reference 的 HTTP 协议元素都会以 ABNF   的形式表明它被允许的 reference 形式，比如只能以绝对形式（ absolute-URI ）、只有 path 和可能的 query 成分（貌似与 URI reference 的定义产生了矛盾？），或者一些上述规则的组合。除非特别指明，URI references 会以相对 [effective request URI](#Section55) 的形式被解析。

##### 2.7.1 HTTP URI 格式

"http" URI 格式被定义的目的是标识与他们存在联系的，监听指定端口 [TCP](https://tools.ietf.org/html/rfc793) 连接的潜在 HTTP origin server。（原文 "The "http" URI scheme is hereby defined for the purpose of minting
   identifiers according to their association with the hierarchical
   namespace governed by a potential HTTP origin server listening for
   TCP ([ [RFC0793] ](https://tools.ietf.org/html/rfc793)) connections on a given port." 这段可能存在更好的翻译？）
   
```
http-URI = "http:" "//" authority path-abempty [ "?" query ]
                [ "#" fragment ]

```

"http" URI 中由包含 [host](https://tools.ietf.org/html/rfc3986#section-3.2.2) 并可能附带 TCP 端口的 authority 来标识 origin server。分级的 path 和可选的 query 则作为在 origin server 的名字空间中定位潜在目标资源的标识。可选的 fragment 可作为定位二级资源的间接标识，它是独立于 URI scheme 的，如同 [Section 3.5 of RFC-3986](https://tools.ietf.org/html/rfc3986#section-3.5) 中所定义的那样。

一个 Sender **MUST NOT** 创建一个没有 Host 标识的 HTTP URI。收到这种 URI 的 Recipient **MUST** 把它作为无效 URI 拒绝处理。

如果提供的 Host 标识是一个 IP 地址，则 origin server 监听在此 IP 地址的指定 TCP 端口上。如果 host 是一个注册名（域名？），这个注册名就是一个通过像 DNS 这样的名字解析服务来找 origin server IP 地址的间接标识。如果端口号为空或者没有指定，则默认 TCP 端口 80（万维网服务的保留端口）。

要记住，有着指定 authority 的 URI 的存在并不意味着这个 Host 和 Port 总有一个 HTTP Server 在监听。每个人都可以创建一个 URI。Authority 决定的是谁有权利、有权威性地响应一个请求并返回目标资源。注册名和 IP 地址被赋予的天然特性，基于对指定 host 和 port 是否存在 HTTP Server 的控制，创造了联合的命名空间。在 [Section 9.1]() 中我们可以看到关于建立权威性的安全考量。

当一个 "http" URI 在某些语境下被用于访问指定的资源，一个客户端 **MAY** 尝试访问此资源，通过把 Host 解析为 IP 地址、在指定端口建立 TCP 连接，然后发送含有 [URI 标识数据](#Section5) 的[请求报文](#3-消息格式)。如果该 Server 用一个非过渡的 HTTP 响应报文来回复某请求，像 [Section 6 of RFC-7231](https://tools.ietf.org/html/rfc7231#section-6) 中描述的那样，则可以认为这是一次对该客户端请求的具有权威性的响应。

虽然 HTTP 独立于传输层协议，当 "http" scheme 特指基于 TCP 的服务，因为名称代理（猜测是指 authority，即名字空间？）依赖 TCP 来建立权威的链接。基于其它底层连接协议的 HTTP 服务一般会使用其它的 URI scheme 作为标识，就像 "https" scheme 被用于传递有端对端加密需求的资源。其它的协议可能也会被用于提供对 "http" 标识资源的访问 —— 它只是一种基于 TCP 的特定的，广为人知的接口。

URI 通用语法中的 authority 也含有一个 deprecated 的 userinfo 子字段 [Section 3.2.1 of RFC-3986](https://tools.ietf.org/html/rfc3986#section-3.2.1) 以便于在 URI 中包含用户认证信息。某些实现充分地利用了 userinfo 字段来进行认证信息的内部配置，比如命令调用中的选项、配置文件、或者书签列表，即使如此这样的使用可能会暴露用户的标识或者密码。一个 Sender **MUST NOT** 创建 useinfo 子字段（以及它的 "@"）当一个 HTTP URI references 在一个报文中作为请求目标或者头字段的值。在利用一个从不被信任来源收到的 HTTP URI references 之前，该 recipient **SHOULD** 解析其中的 userinfo 字段并把它作为一种错误来处理。userinfo 字段可能被用于混淆 authority 以发起网络钓鱼攻击。

##### 2.7.2 HTTPS URI 格式
##### 2.7.3 HTTP 和 HTTPS URI 的正规化和匹配

### 3. 消息格式
#### 3.1 起始行
##### 3.1.1 请求行
##### 3.1.2 状态行
#### 3.2 头字段
##### 3.2.1 字段的可扩展性
##### 3.2.2 字段顺序
##### 3.2.3 空白
##### 3.2.4 字段解析
##### 3.2.5 字段限制
##### 3.2.6 字段值的组成成分
#### 3.3 消息体
##### 3.3.1 Transfer-Encoding
##### 3.3.2 Content-Length
##### 3.3.3 消息体的长度
#### 3.4 处理不完整的消息
#### 3.5 消息解析的鲁棒性



