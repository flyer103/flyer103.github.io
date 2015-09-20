---
layout: post
title: "对 garbage collection 的理解"
date: 2015-09-20 15:35:39
categories: thinkings,theory,garbage collection,memory management
---
周末没事儿，补下基础知识，系统理解下 `garbage collection` 相关的内容。由于它属于
`memory management` 的范畴，先理解下 `memory mangement` 相关的。  

理解可能会存在错误，请指正。  

这篇 blog 按如下方式组织:
* 什么是『memory management』？  
* 什么是『garbage collection』?  
* 『garbage collection』的策略和应用有哪些？  
* 常用语言的『memory management』策略  
* Reference  

### 什么是『memory management』？
来自 [维基百科][memory management] 的解释:  

> `Memory management` is the act of managing computer memory at the system
> level. The essential requirement of memory management is to provide ways to
> dynamically alloate portions of memory to programs at their request, and free
> it for reuse when no longer needed.

即『memory management』包含两部分:  
* 内存分配  
* 内存收回  

对于『内存分配』，会将变量或对象分配到 stack 或 heap 上。  

对于函数创建和使用的简单变量 (非指针)，通常会在 stack 分配内存，在该函数退出时，
该 stack 中的元素会自动被全部释放。但 stack 的大小有限，超出 stack 大小时会报
[stack overflow][stack overflow] 的异常。  

heap 的空间很大，一些通过显式分配内存的变量通常占用的都是 heap 空间，这些占用的
空间需要手动或通过自动化的手段进行释放，避免出现 [memory leak][memory leak] 现象，
造成程序性能下降或影响系统的稳定性。  

在 [这篇 blog][memory stack vs heap] 中总结了 stack 和 heap 的优缺点，挺有用的:  

stack:  
* very fast access  
* don't have to explicitly de-allocate variables  
* space is managed efficiently by CPU, memory will not become fragmented  
* local variables only  
* limit on stack size (OS-dependent)  
* variables cannot be resized  

heap:  
* variables can be accessed globally  
* no limit on memory size  
* (relatively) slower access  
* no guranteed efficient use of space, memory may become fragmented over time as
blocks of memory are allocated, then freed  
* you must manage memory (you're in charge of allocating and freeing variable)  
* variables canbe resized  

这篇 blog 中有两个例子来说明对 stack 和 heap 的使用。第一个例子中的变量全都分配
在 stack 上，第二个例子中的变量全都分配在 heap 上。  

例 1:  

{% highlight c lineno %}
#include <stdio.h>

double multiplyByTwo(double input) {
  double twice = input * 2.0;
  reutrn twice;
}

int main(int argc, char *argv[])
{
  int age = 30;
  double salary = 12345.67;
  double myList[3] = {1.2, 2.3, 3.4};

  printf("double your salary is %.3f\n", multiplyByTwo(salary));

  return 0;
}
{% endhighlight %}

例 2:  

{% highlight c lineno %}
#include <stdio.h>
#include <stdlib.h>

double *multiplyByTwo(double *input) {
  double *twice = malloc(sizeof(double));
  *twice = *input * 2.0;
  return twice;
}

int main(int argc, char *argv[])
{
  int *age = malloc(sizeof(int));
  *age = 30;
  double *salary = malloc(sizeof(double));
  *salary = 12345.67;
  double *myList = malloc(3 * sizeof(double));
  myList[0] = 1.2;
  myList[1] = 2.3;
  myList[2] = 3.4;

  double *twiceSalary = multiplyByTwo(salary);

  printf("double your salary is %.3f\n", *twiceSalary);

  free(age);
  free(salary);
  free(myList);
  free(twiceSalary);

  return 0;
}
{% endhighlight %}

可知，『内存收回』关心的主要问题是『如何对分配在 heap 上的不再被使用的内存进行收
回』。  

对于『内存收回』，有两种不同的策略:  
* [manual][wiki: manual memory management]
  * 开发人员负责，手动释放，如 C、C++
  * 其中有个比较有趣的，是 [RAII: Resource Acquisition is Initialization][RAII]
* auto  
  * 作为语言的 feature 或实现，如『garbage collection』  

使用了 GC 的语言，一方面可以减轻开发人员的负担，使得开发人员可以将注意力更加集中
到解决问题的逻辑而非语言使用细节上，另一方面可以帮助减少一些因为『内存管理』带来
的风险和 bug。  

来自 [wiki: garbage collection][wiki: garbage collection] 总结了『garbage
collection』 的优缺点:  
* 优点  
  * It frees the programmer from manually dealing with memory deallocation. As a
  result, certain categories of bugs are eliminated or substantially reduced:  
    * Dangling pointer bugs, which occur when a piece of memory if freed while
	there are still pointers to it, and one of those pointers is deferenced. By
	then the memory may have been reassigned to another use, with unpredictable
	results.  
    * Double free bugs, which occur when the program tries to free a region of
	memory that has already been freed, and perhaps already been allocated
	again.  
    * Certain kinds of memory leaks, in which a program fails to free memory
	occupied by objects that have become unreachable, which can lead to memory
	exhaustion. (Garbage collection typically does not deal with the unbound
	accumulation of data that is reachable, but that will actually not be used
	by the program)  
    * Efficient implementation of persistent data structures.
* 缺点  
  * consuming additional resources  
  * performance impacts  
  * possible stalls in program execution  
  * incompatibility with manual resource management  

### 什么是『garbage collection』?
来自 [wiki: garbage collection][wiki: garbage collection] 的解释:

> In computer science, `garbage collection` (GC) is a form of automatic memory
> manegement. The `garbage collector`, or just `collector`, attempts to reclaim
> garbage or memory occupied by objects that are no longer in use by the
> program.  

即『GC』会自动收集已经分配出去、不再被使用的内存。

###『garbage collection』的策略和应用有哪些？  
『GC』的基本策略是 `找到程序中不再被使用的数据对象，释放对应的内存`。  

常量的『GC』策略:  
* [tracing][wiki: tracing garbage collection] (最常用)  
  * The overall strategy consists of determining which objects should be garbage
  collected by tracing which objects are reachable by a chain of references from
  certain root objects, and considering the rest as garbage and collecting
  them.  
  * An object is reachable if it is referenced by at least one variable in the
  program, either directly or through references from other reachable
  objects. More precisely, objects can be reachable in only two ways:    
    * A distinguished set of roots: objects that are assumed to be
	reachable. Typically, these include all the objects referenced from
	anywhere in the call stack (that is, all local variables and parameters in
	the functions currently being invoked), and any global variables  
    * Anything referenced from a reachable object is itself reachable; more
	formally, reachability is a transitive closure.  
  * reference 策略:  
    * strong reference  
    * weak reference
* [reference counting][wiki: reference counting garbage collection]  
  * `Reference counting` is a form fo garbage collection whereby each object has
  a count of number of references to it. `Garbage` is identified by having a
  reference count of zero. An object's reference count is increment when a
  reference to it is created, and decremented when a reference is destroyed. The
  object's memory is reclaimed when the count reaches zero.  
  * As with `manual memory management`, and unlike tracing garbage collection,
  reference counting guarantees that objects are destroyed as soon as their last
  reference is destroyed, and usually only accesses memory which is either in
  CPU caches, in objects to be freed, or directly pointed by those, and thus
  tends to not have significant negative side effects on CPU cache and virtual
  memory operation.

### 常用语言的『memory management』策略
Python 的 `CPython` 实现使用 reference counting based GC。

Java 使用 generational GC。  

PHP 使用 reference counting based GC，同时限制每个实例可以使用的内存。  

Go 混合使用 `stop the world(STW)` 和 `concurrent garbage collection(CGC)` GC。

### Reference
* [memory management][memory management]  
* [stack overflow][stack overflow]  
* [memory leak][memory leak]  
* [memory stack vs heap][memory stack vs heap]  
* [RAII][RAII]  
* [wiki: garbage collection][wiki: garbage collection]  
* [wiki: tracing garbage collection][wiki: tracing garbage collection]  
* [wiki: reference counting garbage collection][wiki: reference counting garbage collection]  
* [wiki: manual memory management][wiki: manual memory management]  
* [stackoverflow: Stack variables vs. Heap variables][[stackoverflow: Stack variables vs. Heap variables]]  
* [stackoverflow: Why Java and Python garbage collection methods are different?][stackoverflow: Why Java and Python garbage collection methods are different?]  
* [quora: What are the differences between Python and Java memory management?][quora: What are the differences between Python and Java memory management?]  
* [php: memory management][php: memory management]  

[memory management]: https://en.wikipedia.org/wiki/Memory_management  
[stack overflow]: https://en.wikipedia.org/wiki/Stack_overflow  
[memory leak]: https://en.wikipedia.org/wiki/Memory_leak  
[memory stack vs heap]: http://gribblelab.org/CBootcamp/7_Memory_Stack_vs_Heap.html  
[RAII]: https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization  
[wiki: garbage collection]: https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29  
[wiki: tracing garbage collection]: https://en.wikipedia.org/wiki/Tracing_garbage_collection  
[wiki: reference counting garbage collection]: https://en.wikipedia.org/wiki/Reference_counting  
[wiki: manual memory management]: https://en.wikipedia.org/wiki/Manual_memory_management  
[stackoverflow: Stack variables vs. Heap variables]: http://stackoverflow.com/questions/5258724/stack-variables-vs-heap-variables  
[stackoverflow: Why Java and Python garbage collection methods are different?]: http://stackoverflow.com/questions/21934/why-java-and-python-garbage-collection-methods-are-different  
[quora: What are the differences between Python and Java memory management?]: https://www.quora.com/What-are-the-differences-between-Python-and-Java-memory-management  
[php: memory management]: http://php.net/manual/en/internals2.memory.php  
