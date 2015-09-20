---
layout: post
title: "从 资源访问控制 角度理解 PHP 的几个关键字"
date: 2015-09-20 18:12:38
categories: php, class, resource
---
### 综述
这几天从『资源访问控制』 的角度来理解 PHP 的 `static`、`public`、`protected` 和 `private` 关键字，感觉还挺有趣的。  

class 可以用来描述一组资源，其中包括描述静态资源的 `property` 和动态资源的 `method`。instance 是 class 的实例，具体化了 class 描述的资源。  

此时，就涉及到资源的访问控制，即『谁可以访问 `class/instance` 描述的资源』。  

这种设计思想挺有帮助的，在系统设计方面有借鉴作用。

### 第一个层面的访问控制
该访问控制回答的问题是『class 和 instance 可以访问到的资源范围包括哪些？』，解决方案是使用 `static` 关键字:  

* class 只能访问 static 定义的资源  
* instance 可以访问 class 描述的所有资源，包括非 staitic 定义的资源  

### 第二个层面的访问控制
该访问控制回答的问题是『instance 对外可以使用的资源包括哪些？』，解决方案是使用 `public` 和 `protected` 关键字:  

* public 描述的资源可以对外使用，且所有资源默认情况下都是 public  
* 不对外使用的资源通过 protected 描述  

P.S.: `private` 关键字在该层面的访问控制上，默认与 `protected` 有相同的行为。

### 第三个层面的访问控制
该访问控制回答的问题是『对于有继承关系的 class, 可被父/子使用的资源有哪些』，解决方案是引入 `private` 关键字:  

* 被 private 定义的资源只能供自身 instance 使用，不能被具有继承关系的 instance 使用  
* 被 public/protected 定义的资源可被具有继承关系的 instance 使用
