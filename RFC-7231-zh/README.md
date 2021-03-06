### 超文本传输协议（HTTP/1.1）：语义及内容

#### 摘要

超文本传输协议是一种为分布式、协作化的超文本信息系统而设计的无状态应用层协议。本文档定义了`HTTP/1.1`消息的语义，并通过请求方法、请求头字段、响应状态码、响应状态码、响应头字段描述，同时描述了消息负载（`metadata`和`body content`）和内容协商机制。

####  目录

- [x] 1. 引言

  - [x] 1.1. 一致性和错误处理   
  - [x] 1.2. 语法表示规范

- [x] 2. 资源`Resources`

- [ ] 3. 表示法

  - [ ] 3.1. 元数据表示法
    - [ ] 3.1.1
    - [ ] 3.1.2
    - [ ] 3.1.3
    - [ ] 3.1.4
  - [ ] 3.2. 数据表示法
  - [ ] 3.3
  - [ ] 3.4
    - [ ] 3.4.1
    - [ ] 3.4.2

- [ ] 4.

  - [ ] 4.1
  - [ ] 4.2
    - [ ] 4.2.1
    - [ ] 4.2.2
    - [ ] 4.2.3
  - [ ] 4.3
    - [ ] 4.3.1
    - [ ] 4.3.2
    - [ ] 4.3.3
    - [ ] 4.3.4
    - [ ] 4.3.5
    - [ ] 4.3.6
    - [ ] 4.3.7
    - [ ] 4.3.8

- [ ] 5

  - [ ] 5.1
    - [ ] 5.1.1
    - [ ] 5.1.2
  - [ ] 5.2
  - [ ] 5.3
    - [ ] 5.3.1
    - [ ] 5.3.2
    - [ ] 5.3.3
    - [ ] 5.3.4
    - [ ] 5.3.5
  - [ ] 5.4
  - [ ] 5.5
    - [ ] 5.5.1
    - [ ] 5.5.2
    - [ ] 5.5.3

- [ ] 6

  - [ ] 6.1
  - [ ] 6.2
    - [ ] 6.2.1
    - [ ] 6.2.2
  - [ ] 6.3
    - [ ] 6.3.1
    - [ ] 6.3.2
    - [ ] 6.3.3
    - [ ] 6.3.4
    - [ ] 6.3.5
    - [ ] 6.3.6
  - [ ] 6.4
    - [ ] 6.4.1
    - [ ] 6.4.2
    - [ ] 6.4.3
    - [ ] 6.4.4
    - [ ] 6.4.5
    - [ ] 6.4.6
    - [ ] 6.4.7
  - [ ] 6.5
    - [ ] 6.5.1
    - [ ] 6.5.2
    - [ ] 6.5.3
    - [ ] 6.5.4
    - [ ] 6.5.5
    - [ ] 6.5.6
    - [ ] 6.5.7
    - [ ] 6.5.8
    - [ ] 6.5.9
    - [ ] 6.5.10
    - [ ] 6.5.11
    - [ ] 6.5.12
    - [ ] 6.5.13
    - [ ] 6.5.14
    - [ ] 6.5.15

### 1. 引言

每一条超文本传输协议（HTTP）消息都是一条请求或是一条响应。服务器为一条请求监听一个连接，解析收到的每一条请求，解释关联到确定的（`identified`不知道怎么翻）请求资源的消息的语义，并且通过一条或者多条响应消息响应请求。客户端为特定的通信目的构造请求信息，测试收到的响应消息确定目标是否被响应携带，并决定如何解释结果。这个文档就[[RFC7230]](https://tools.ietf.org/html/rfc7230)定义的结构定义`HTTP/1.1`的语法。

HTTP通过操作和表示法（`representations`）（[Section 3]()）的传输对资源的相互作用提供了一种统一的接口（[Section 2]()），不管它的类型、性质(`nature`)或是实现。

HTTP语法包括了由各种请求方式（[Section 4]()）定义的请求意图，语法的扩展会在请求头字段（[Section 5]()）中被描述， 状态码的含义将表明一种机器可读的响应（[Section 6]()），和另外一些将会在响应头字段（[Section 7]()）给出的控制数据和资源元数据的含义。

这个文档同时定义了描述接收者如何解释载荷的意图的元数据表示法，请求头字段可能会影响数据选取，以及被统称为“内容协商”`content negotiation`的不同的选取算法（[Section 3.4]()）。

#### 1.1. 一致性和错误处理

本文档中的关键词 **MUST** 、 **MUST NOT** 、 **REQUIRED** 、 **SHALL** 、 **SHALL NOT** 、 **SHOULD** 、 **SHOULD NOT** 、 **RECOMMENDED** 、 **MAY** 和 **OPTIONAL** 应依照 [RFC2119](https://tools.ietf.org/html/rfc2119) 中的描述，本译文不会翻译这些关键词。

有关错误处理的一致性标准和注意事项在 [Section 2.5](https://tools.ietf.org/html/rfc7230) 中定义。

#### 1.2. 语法表示规范

这份规范使用[扩充巴科斯范式](https://tools.ietf.org/html/rfc5234)（ `ABNF` ）作为主要的语法表示规范。另外，在 [Section 7]() 中定义了一个 `ABNF` 的列表扩展符号 `#` 操作符（与 `*` 操作符表示重复的形式类似），它允许紧凑定义的、由逗号分隔的列表。

[Appendix B]() 展示了所有本文档中收录的全由标准 `ABNF` 所表示语法。

下列核心规则都可供参考，如同 [RFC5234, Appendix B.1](https://tools.ietf.org/html/rfc5234#appendix-B.1) 中的定义：`ALPHA`（字母），`CR`（回车），`CRLF (CR LF)`，`CTL`，`DIGIT`（数字 0-9），`DQUOTE`（双引号），`HEXDIG`（十六进制数 0-9/A-F/a-f），`HTAB`（水平 Tab），`LF`（换行），`OCTET`（字节），`SP`（空格）和 `VCHAR`（任何可见的 USASCII 字符）。

作为惯例，以 "obs-" 开头的 `ABNF` 规则表示已经废弃的语法规则。

> 两节内容均引自@[Hexilee](https://github.com/Hexilee)所翻译的RFC-7230

### 2. 资源 `Resources`

HTTP请求的目标被称为一个“资源”，HTTP不限制资源的性质，并且它很少定义可能会用来和资源交互的接口。任何一个资源都被一个统一资源标识符`Uniform Resource Identifier`（URI）标识，这在`[RFC7230]`的Section 2.7被提到过。

当一个客户端构造一条`HTTP/1.1`请求消息时，它通过其中一种在（`Section 5.3 of
[RFC7230]`）中定义形式发送目标的`URI`。当请求被接收时，服务器重构一条关于目标的有效的请求`URI`（[Section 5.5 of [RFC7230]]()）。

设计HTTP的一个目的是将资源标识从语法中分离出来，通过将请求语义授予请求方法（[Section 4]()）和一些请求改动（`request -modifying`不知道怎么翻译，是被请求修改还是可以修正请求的）的头字段（[Section 5]()）。如果出现了就像在[Section 4.2.1]()中提到的在方法语义和任何隐含在`URI`本身中的语义冲突时，方法语义占有优先权。







