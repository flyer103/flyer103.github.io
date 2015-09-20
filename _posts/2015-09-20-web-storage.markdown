---
layout: post
title: "web storage 小结"
date: 2015-09-20 18:28:55
categories: web, http, storage
---
迁移下 2015-05-03 写的一篇 blog。
<hr>

最近了解到一个叫 [web storage][web storage] 的概念，系统总结下。

alias: dom storage

作用: 支持在浏览器中存储数据。  

有两种形式:  

* session storage  
    * 类似 session cookie  
    * 每个 window 对应一个存储  
    * 当 window 关闭时，该存储会被删除掉  
    * 数据只有在当前的 window 下才能被访问到  
    * Session storage is intended to allow separate instances of the same web application to run in different windows without interfering with each other  
* local storage  
    * 类似 persistent cookie  
    * 每个存储由 "protocol、hostname、port number” 的集合体唯一标记，数据只能在相同的域下才能被访问到，遵守 [same-origin policy][same-origin policy]  
    * 数据不会被自动删除，只能通过客户端脚本来删除  

访问:  

* 只有客户端脚本才能访问到该数据  
* 客户端脚本访问时，不像传统的数据库一样需要指明 database 和 table，由浏览器控制访问的数据空间，即客户端脚本访问时，已经在可以访问的数据空间内  
* 浏览器发送请求时，不会主动带上 storage 中的数据  

存储容量:  

* 不同版本的浏览器支持的存储大小不同  

<hr>
参考:  

* [wiki: web storage][wiki: web storage]  
* [wiki: same_origin policy][wiki: same_origin policy]  
* [w3school: html5 local storage][w3school: html5 local storage]  
* [stackoverflow: when do items in HTML5 local storage expire?][stackoverflow: when do items in HTML5 local storage expire?]  
* [stackoverflow: make localStorage or sessionStorage expire like cookies][stackoverflow: make localStorage or sessionStorage expire like cookies]  
* [mdn: web storage api][mdn: web storage api]  
* [mdn: using the web storage api][mdn: using the web storage api]  


[wiki: web storage]: http://en.wikipedia.org/wiki/Web_storage
[wiki: same_origin policy]: http://en.wikipedia.org/wiki/Same-origin_policy
[w3school: html5 local storage]: http://www.w3schools.com/html/html5_webstorage.asp
[stackoverflow: when do items in HTML5 local storage expire?]: http://stackoverflow.com/questions/2326943/when-do-items-in-html5-local-storage-expire
[stackoverflow: make localStorage or sessionStorage expire like cookies]: http://stackoverflow.com/questions/13011944/make-localstorage-or-sessionstorage-expire-like-cookies
[mdn: web storage api]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
[mdn: using the web storage api]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API
[same-origin policy]: http://en.wikipedia.org/wiki/Same-origin_policy
