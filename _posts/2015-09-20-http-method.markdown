---
layout: post
title: "对 http post/get 的理解"
date: 2015-09-20 18:40:37
categories: http, web, http method, theory
---
迁移下 2015-05-10 写的一篇 blog。
<hr>

这周做项目时，被 HTTP POST 和 GET 方法的几个小问题绊住了，现在总结下。  
下面文章的结构为:  

* http 协议的简单结构  
* Internet Media Type 和 MIME  
* http post 方法  
* html form  
* 常见工具对 http post 的处理  
* http get 参数长度  

<br><br>
### http 协议的简单结构  
参考下 [这个图片][http protocol format]，比较清晰地勾画出了 http 协议的格式：  

![http protocol]({{site.url}}/assets/http_protocol_format.png)

整体而言，http 协议由两部分构成:  

* http header  
  用来描述一些基本信息，这些信息一般用于控制方面。
* http body  
  用来描述实际需要处理的数据。  
  
数据需要经过编码后才能进行传输，即 http body 中的数据需要是编码过的，同时，http body 可以有不同的组成结构。  
[Internet media type][Internet media type] 可以用来描述 http body 的结构，且可部分描述 http body 的编码 (例外情况为在 http response 中，通过 Content-Encoding 描述 http body 是否被压缩过)。  

### Internet Media Type 和 MIME   
[Internet media type][Internet media type] 用来描述数据实体的类型，常见的用法如下:  

* Email 客户端通过它来识别附件  
* web 浏览器通过它来判断如何处理非 html 格式的数据  
* 搜索引擎通过它来对 web 上的数据进行分类  

Internet media type 的常见形式为:  

> type/subtype[; 属性对; ...; 属性对]
    
如:  

> text/html; charset=UTF-8  
    
以 html form 中 `enctype` 使用的两种形式为例再解释下:  

* application/x-www-form-urlencoded  
  该类型是 html form 采用 post 方法时默认的编码形式，它把 form 提交的数据转换为类似 http get 的参数形式，这些数据作为 http body 进行传输。  
  这种编码的缺陷是，不能传输非 ASCII 字符串和二进制数据。  
* multipart/form-data  
  该类型把 http body 逻辑上分成不同的部分，每个部分相互独立，可以有不同的编码方式、数据类型。但 `form-data` 这个 subtype 又限定了这种情况下的 `multipart` 的格式:  
      * 在 `multipart/form-data` 的声明后必须加上 `boundary` 这个属性，用来指明 http body 中，不同的 part 通过什么来分隔  
      * 每个 part 希望存在 `Content-Disposition: form-data; name=xxx` 的部分，用来描述 html form 传输的 key，同时该 part 的数据部分作为 `name` 对应的值。  
  
  来自 [w3_form][w3_form] 的一个例子:  
  
>      Content-Type: multipart/form-data; boundary=AaB03x
>
>       --AaB03x
>       Content-Disposition: form-data; name="submit-name"
>
>       Larry
>       --AaB03x
>       Content-Disposition: form-data; name="files"
>       Content-Type: multipart/mixed; boundary=BbC04y
>    
>       --BbC04y
>       Content-Disposition: file; filename="file1.txt"
>       Content-Type: text/plain
>
>       ... contents of file1.txt ...
>       --BbC04y
>       Content-Disposition: file; filename="file2.gif"
>       Content-Type: image/gif
>       Content-Transfer-Encoding: binary
>
>       ...contents of file2.gif...
>       --BbC04y--
>       --AaB03x--

Internet media type 中定义的一些 type 参考了 [MIME][MIME]。MIME 全称是 `Multi-Purpose Internet Mail Extensions`，用来指导 Email 传输非 ASCII 的数据。具体的信息可以参考下 [wiki][MIME]

### http post 方法  
通过 http post 进行数据传输时，需要在 http header 的 `Content-Type` 采用 Internet media type 允许的 type 描述 http body 中的数据。  
若 Content-Type 字段缺失，HTTP/1.1 给出的指导为:  

> Any HTTP/1.1 message containing an entity-body SHOULD include a Content-Type header field defining the media type of that body. If and only if the media type is not given by a Content-Type field, the recipient MAY attempt to guess the media type via inspection of its content and/or the name extension(s) of the URI used to identify the resource. If the media type remains unknown, the recipient SHOULD treat it as type "application/octet-stream".   
    
理论上来说，通过 http post 进行数据传输时，Content-Type 可以指定为 Internet media type 中的任意一种，同时接收数据的一方必须能够处理这种类型的 Content-Type，否则需要返回 415 (Unsupported media type) 状态码。

### html form  
html form 可通过 http get 和 http post 两种形式传输数据，根据 [这篇文档的描述][w3org: form]:  

* 采用 http get 时，user agent (如浏览器) 通过 `application/x-www-form-urlencoded` 方式编码提交的数据，并通过常见的 http get 形式传输数据。这种方式严格限定于处理 ASCII 字符串，同时必须是 [幂等行为][wiki: idempotence]，简单来说就是，这次行为不会影响到处理方的状态，如数据库中的数据不会有变更等  
* 采用 http post 时，可以接收 3 种形式的 enctype:  
    * application/x-www-from-urlencoded  
      它是默认的 Content-Type，只能用于处理 ASCII 字符串。
    * multipart/form-data  
      它可以用来处理大量的二进制数据和非 ASCII 编码的字符串。
    * text/plain  
      参考 [html5_forms:plain-encoding-algorithm][html5_forms:plain-encoding-algorithm]，不建议使用  
      
即 html form 采用 http post 方法传输数据时，限定了可以采用的 Content-Type 类型。

### 常见工具对 http post 的处理  
对 jQuery 而言，通过 ajax 方式发送 post/get 请求时，默认的 Content-Type 是 `application/x-www-form-urlencoded; charset=UTF-8` 。  

PHP 中，[$_POST][php_manual: $_POST] 只能处理 `application/x-www-form-urlencoded` 和 `multipart/form-data` 的数据，对于其他 Internet media type 的数据，先通过 [php://input][php_manual: input] 获取原始的 http body 数据，然后再根据 http header 中的 Content-Type 进行解析。  

[python requests][python requests] 通过 http post 传输数据时，底层通过 post 的参数决定 content-type 类型 (也可以显式指定)。  

对于不同的 web 开发框架，处理 post 数据时，建议先详细了解下能够处理的 Content-Type 类型包括哪些。

### http get 参数长度  
http 协议中没有限定 http get 参数的长度，但在实际使用时，`web server`、`web 浏览器`、`web 后端开发语言` 支持的 http get 参数长度有限，对于过长的 http get 请求，会返回 414(Request-URI too long) 状态码。对于这种情况，建议改为 http post 方法或重新考虑 http get 语义。  

### 小感悟  
先大概理解协议，再使用处理该协议的工具，然后重复这个过程加深对协议和相应工具的理解。

<hr>
参考:  

* [Communication Networks/HTTP Protocol][Communication Networks/HTTP Protocol]  
* [Internet media type][Internet media type]  
* [MIME][MIME]  
* [w3_form][w3_form]  
* [RFC2616-Entity][RFC2616-Entity]  
* [What are idempotent and/or safe methods?][What are idempotent and/or safe methods?]  
* [w3school:att_form_enctype][w3school:att_form_enctype]  
* [What does enctype='multipart/form-data' mean?][What does enctype='multipart/form-data' mean?]  
* [Form content type for a json HTTP POST?][Form content type for a json HTTP POST?]  
* [How are parameters sent in an HTTP POST request?][How are parameters sent in an HTTP POST request?]  
* [RFC2388: multipart/form-data][RFC2388: multipart/form-data]  
* [四种常见的 POST 提交数据方式][四种常见的 POST 提交数据方式]  
* [php:php//input][php_manual: input]  
* [PHP “php://input” vs $_POST][PHP “php://input” vs $_POST]  
* [Is there a limit to the length of a GET request?][Is there a limit to the length of a GET request?]  
* [What is the limit on QueryString / GET / URL parameters][What is the limit on QueryString / GET / URL parameters]  
* [What is the maximum length of a URL in different browsers?][What is the maximum length of a URL in different browsers?]  
* [maximum length of HTTP GET request?][maximum length of HTTP GET request?]  
* [Max size of URL parameters in _GET][Max size of URL parameters in _GET]  


[http protocol format]: http://commons.wikimedia.org/wiki/File:Prj5-responseHeader.png
[Internet media type]: http://en.wikipedia.org/wiki/Internet_media_type
[w3_form]: http://www.w3.org/TR/html401/interact/forms.html
[MIME]: http://en.wikipedia.org/wiki/MIME
[w3org: form]http://www.w3.org/TR/html401/interact/forms.html#h-17.13.3
[wiki: idempotence]: http://en.wikipedia.org/wiki/Idempotence
[html5_forms:plain-encoding-algorithm]: http://www.w3.org/TR/html5/forms.html#text/plain-encoding-algorithm
[php_manual: $_POST]: http://php.net/manual/en/reserved.variables.post.php
[php_manual: input]: http://php.net/manual/en/wrappers.php.php#wrappers.php.input
[python requests]: http://docs.python-requests.org/en/latest/
[Communication Networks/HTTP Protocol]: http://en.wikibooks.org/wiki/Communication_Networks/HTTP_Protocol
[RFC2616-Entity]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html
[What are idempotent and/or safe methods?]: http://restcookbook.com/HTTP%20Methods/idempotency/
[w3school:att_form_enctype]: http://www.w3schools.com/tags/att_form_enctype.asp
[What does enctype='multipart/form-data' mean?]: http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean
[Form content type for a json HTTP POST?]: http://stackoverflow.com/questions/4249609/form-content-type-for-a-json-http-post
[How are parameters sent in an HTTP POST request?]: http://stackoverflow.com/questions/14551194/how-are-parameters-sent-in-an-http-post-request
[RFC2388: multipart/form-data]: http://tools.ietf.org/html/rfc2388
[四种常见的 POST 提交数据方式]: https://www.imququ.com/post/four-ways-to-post-data-in-http.html
[PHP “php://input” vs $_POST]: http://stackoverflow.com/questions/8893574/php-php-input-vs-post
[Is there a limit to the length of a GET request?]: http://stackoverflow.com/questions/266322/is-there-a-limit-to-the-length-of-a-get-request
[What is the limit on QueryString / GET / URL parameters]: http://stackoverflow.com/questions/3091485/what-is-the-limit-on-querystring-get-url-parameters
[What is the maximum length of a URL in different browsers?]: http://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers
[maximum length of HTTP GET request?]: http://stackoverflow.com/questions/2659952/maximum-length-of-http-get-request
[Max size of URL parameters in _GET]: http://stackoverflow.com/questions/7724270/max-size-of-url-parameters-in-get
