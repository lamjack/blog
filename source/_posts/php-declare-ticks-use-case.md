---
title: PHP脚本中declare(ticks=N);的作用
date: 2016-10-11 17:28:23
tags:
	- php
	- declare
---
参考PHP进程方面文章时候，经常看到**declare(ticks=N)**，网上搜索了一些资料，这里单独记录下加深印象。

<!--more-->

### 概述

手册解释，
> A tick is an event that occurs for every N low-level statements executed by the parser within the declare block. The value for N is specified using ticks=N within the declare blocks's directive section. 
> 
> The event(s) that occur on each tick are specified using the register_tick_function().

这个解释有三层意思，

1. tick是一个事件。
2. tick事件在PHP每执行N条**低级语句**就发生一次，N由declare语句指定。
3. 可以用register_tick_function()来指定tick事件发生时应该执行的操作。

低级语句，

1. 简单语句：空语句（就一个；号），return,break,continue,throw, goto,global,static,unset,echo, 内置的HTML文本，分号结束的表达式等均算一个语句。
2. 复合语句：完整的if/elseif,while,do...while,for,foreach,switch,try...catch等算一个语句。
3. 语句块：{} 括出来的语句块。
4. 最后特别的：declare块本身也算一个语句(按道理declare块也算是复合语句，但此处特意将其独立出来)。

所有的**statement, function_declare_statement, class_declare_statement**就构成了所谓的**低级语句(low-level statement)**。

### 应用场景
#### 控制代码执行时间
可以粗略的理解为每执行一句PHP**低级语句**，就去执行下已经注册的tick函数。
一个用途就是控制某段代码执行时间，例如下面的代码虽然最后有个死循环，但是执行时间不会超过5秒。
```php
<?php
declare(ticks=1);

// 开始时间
$time_start = time();

// 检查是否已经超时
function check_timeout(){
    // 开始时间
    global $time_start;
    // 5秒超时
    $timeout = 5;
    if(time()-$time_start > $timeout){
        exit("超时{$timeout}秒\n");
    }
}

// Zend引擎每执行一次低级语句就执行一下check_timeout
register_tick_function('check_timeout');

// 模拟一段耗时的业务逻辑
while(1){
    $num = 1;
}

// 模拟一段耗时的业务逻辑，虽然是死循环，但是执行时间不会超过$timeout=5秒
while(1){
    $num = 1;
}
```
#### 检查进程是否有未处理信号
```php
<?php
declare(ticks=1);
pcntl_signal(SIGINT, function(){
    exit("Get signal SIGINT and exit\n");
});

echo "Ctl + c or run cmd : kill -SIGINT " . posix_getpid(). "\n" ;

while(1){
    $num = 1;
}
```

### 参考文章
+ [PHP declare(ticks=N); 的作用](http://blog.csdn.net/udefined/article/details/24333333)
+ [php控制结构语句declare中的tick的详解[整理版]](https://my.oschina.net/Jacker/blog/32936)