---
title: PHP进程编程
date: 2016-10-11 15:46:10
tags:
	- php
---
# 进程编程（持续更新中1003）
有时候我们会使用PHP写一些命令行脚本去执行一些常用的任务，比如定时发送邮件等等。之后我们再探讨PHP作为命令行脚本的问题，现在，我们先研究下，如何通过PHP来控制进程。

使用PHP解析器（Windows系统上为php.exe，其它类Unix操作系统为php）来做脚本解析，同一时刻我们只能运行一个进程。如果你的计算机只是单核处理器还好，如果是多核处理器呢？或者是在单核处理器下，但是你处理的是一个比较耗时的任务，比如下载多张图片呢？

为了解决这些问题，我们需要深入到流程控制的领域：产生子进程并控制他们，以及对父进程进行高级控制的能力。悲剧的是，进程控制只能在Unix操作系统上工作，**如果你使用的是Windows操作系统，那读到这里就行了。**如果使用的是Unix系统，记得在编译PHP的时候加上 --enable-pcntl。

在继续学习之前，我们先简单了解下什么是[进程](https://zh.wikipedia.org/zh-hant/行程)。简单来说，一个进程就是一个运行中的**程序实例**，它有自己的进程ID以及内核调度。比如我们只有一个PHP脚本，但是可能有多个在运行这个PHP脚本的**程序实例**。每个进程都有一个唯一的编号**PID**，通过**PID**我们可以访问这个进程。

<!--more-->

## 进程信号捕获 pcntl\_signal()
> bool pcntl\_signal ( int signo, callback handle [, bool restart\_syscalls] )

我们通过**信号**来跟进程进行交互，如果信号对于你来说是一个新的概念的话，你可能把信号理解为**程序与程序间**带着一些**非常简单的指令的信息**，举个栗子，你想退出命令行，只需要按Ctrl-C就退出了，但是为什么呢？

当你按下Ctrl-C的时候，会产生一个**SIGINT**中断信号，发送到应用程序，除非这个应用程序定义了一个**特殊的处理器（handler）**去处理**SIGINT**信号，否则应用程序将执行默认动作**立即终止**。如果应用程序定义了忽略**SIGINT**信号的处理器，我们是否能终止它呢？答案是肯定的，通常做法是使用**kill -9 [PID]**，这将会产生一个**SIGKILL**信号，和**SIGINT**信号不同，这个信号不会被应用程序的处理器捕获，所以应用程序一定会终止。

这听起来好像Unix有两个信号，他们非常相似，做同一件事情。很多应用程序重写了**SIGINT（2）**信号，以便于他们更好地在进程结束之前进行清理，而**SIGKILL（9）**信号则完成不管当前进程的状态去停止进程。**SIGTERM（15）**信号与**SIGINT（2）**、**SIGKILL（9）**信号也非常相似，这个信号是由kill命令默认发送的。更令人困惑的是还有一个叫**SIGQUIT（3）**信号，它和**SIGINT（2）**一样，除了它是通过按Ctrl-\而不是Ctrl-C生成的并且可以在需要的时候生成核心转储。

通过pcntl\_signal()函数，我们可以让我们的PHP脚本来响应除了**SIGKILL**之外的信号。函数的第一个参数为信号名称，一个回调PHP函数名作为第二个函数，通常我们也称之为“信号处理函数”，最后一个参数我们以后再说。

首先看下这个PHP脚本，

```php
<?php
declare(ticks = 1);

pcntl_signal(SIGTERM, "signal_handler");
pcntl_signal(SIGINT, "signal_handler");

function signal_handler($signal) {
    switch($signal) {
        case SIGTERM:
            print "Caught SIGTERM\n";
            exit;
        case SIGKILL:
            print "Caught SIGKILL\n";
            exit;
        case SIGINT:
            print "Caught SIGINT\n";
            exit;
        }
    }

while(1) {
    echo '';
}
```

第一行是一个declare，需要的话可以参考下另外一篇文章[《PHP脚本中declare(ticks=N);的作用》](https://lamjack.github.io/2016/php-declare-ticks-use-case/)。

然后我们调用了两次pcntl\_signal函数，将**SIGTERM**和**SIGINT**信号绑定到信号处理函数signal_handler上，注意SIGTERM和SIGINT都是PHP常量，他们的值跟Unix系统的信号量是相等的。

接下来我们的信号处理函数接受一个信号量参数，我们把信号输出出来以便于查看，注意这里我添加了一个**SIGKILL**信号的case，看看我们的信号处理函数是否能捕获它。

在所有的case块中，我们输出信号的值，并立即用exit退出脚本。如果不调用exit，**SIGINT**和**SIGTERM**都不会终止这个脚本。

最后我们通过一个while循环让脚本一直运行，因为我们将在CLI模式下进行测试，所以不需要担心PHP的脚本最大执行时间的限定，唯一能终止这个脚本的只能是系统信号。<font color=red>注意while循环里面里面有一个空语句。</font>

当然，退出程序是**SIGINT**和**SIGTERM**的默认行为，为什么我们还要麻烦地捕获这些信号呢？当我们的PHP脚本作为守护进程的时候，在退出之前，我们需要做一些善后工作，比如关闭打开的文件，保存数据，写入日志等。当然，当善后工作做完，要记得exit，否则进程将无法通过默认信号停止，只能通过kill -9 [PID]。

## 为进程设置闹钟：pcntl_alarm()
> int pcntl\_alarm ( int seconds )

现在我们已经了解了始终信号处理函数去捕获基本信号并做出处理，然而，还有一个牛逼的信号，进程闹钟 - 你可以让系统每隔固定的秒数就发送一个信号到你的PHP脚本。这是一个很有用的功能，两个方面，第一他可以让你的脚本每隔一段时间就进行一些维护动作，比如释放资源；第二他可以让一些功能函数强制超时。这些都可以通过**pcntl\_alarm()**函数和**SIG_ALRM**信号来实现，pcntl\_alarm()用来设置闹钟，它只需要一个秒数N作为参数，<font color=red>注意，不是循环的</font>。然后N秒后你的脚本就会收到一个**SIG_ALRM**信号。。

重要的是pcntl\_alarm()是非阻塞的，也就是你调用pcntl_alarm()之后立即返回，程序将会继续处理脚本中的其它指令。一旦进程接收到**SIG_ALRM**信号，控制权将转交到你的信号处理函数，然后处理信号处理函数中的相关逻辑，当逻辑处理完毕，控制权又将交回给上一个脚本，当然，这一些的前提是你没有在信号处理函数中exit。

如果你的脚本是处理一些非线性执行的话，这些闹钟信号非常有用。比如，你写了一个邮件程序，然后在邮件列表中，用户可以看到他们的邮件列表和查看每一封邮件内容，然后你的程序每隔60秒要去检查下有没有新的邮件，用这个函数就可以实现。

下面是一段示例代码，每3秒输出"Caught SIGALRM"，我们可以看到我们信号处理函数中又一次使用了pcntl\_alarm()函数，记得它的作用是，设置一次闹钟。

```php
<?php
declare(ticks = 1);

function signal_handler($signal) {
    print "Caught SIGALRM\n";
    pcntl_alarm(3);
}

pcntl_signal(SIGALRM, "signal_handler", true);
pcntl_alarm(3);

while(1) {
    echo '';
}
?>
```

另外pcntl\_alarm()还有三个比较有趣的特性，合理的使用可以让我们的程序功能更加强大，

1. 同一时间只能设置一个闹钟，如果你设置了一个闹钟，在它还没闹之前又设置了一个闹钟，那么新的闹钟会取代旧的闹钟，然后函数的返回值是上一个闹钟还剩下的秒数。
2. 如果设置一个新的闹钟时候把参数设为0，就可以清除所有闹钟，函数的返回值也是上一个闹钟还剩下的秒数。
3. 闹钟信号传到进程的时候，进程的任何工作都会被忽略掉，包括sleep()。

## 开始多进程：pcntl_fork(), pcntl_waitpid(), and pcntl_wexitstatus()
> int pcntl\_fork ( void )
> 
> int pcntl\_waitpid ( int pid, int &status, int options )
> 
> int pcntl\_wexitstatus ( int status )

我们经常讨论PHP脚本单一进程没办法充分利用机器多核的资源，并且如果需要耗时的操作总是需要等待这个操作完成之后才能继续执行。现代的操作系统在后台是具备多任务的处理能力的，如果我们把所有要做的事情从之前一个进程分散到多个进程里去处理，就可以利用操作系统多任务的特性。

在继续之前，先解释下多进程和多线程。进程是有独立的进程号，独立的存储器空间等的程序的唯一实例。线程可以认为是一个虚拟进程，它没有进程ID，没有内存空间，但是也可以利用操作系统多任务的特性。另外还有一个超线程的概念，感兴趣的可以看下这篇文章[《进程，线程，超线程，并发，并行等概念》](http://www.oschina.net/question/733118_80913?fromerr=XvkPjLE2)。

PHP扩展中的Unix进程控制部分只支持fork（复刻？复制?），它通过pcntl\_fork()函数来实现，我们需要多花一些精力来研究这个函数，因为它有点繁琐。

pcntl\_fork()函数有三个返回值。如果返回-1，则代表产生子进程失败，一般都是由于内存不足或者达到了系统对用户进程数量的限定引起的；如果返回值大于0，则代表当前是调用pcntl\_fork()的父进程；如果返回值为0，则代表当前是调用pcntl_\fork()的子进程。

如果fork成功，我们将有两个副本同时执行这个相同的脚本。它们都从pcntl\_fork()行继续执行，最重要的是，子进程获得在父进程中设置的所有变量的副本，甚至资源（指针、引用值），这里将会导致另外一个问题，我们后面会讨论到。下面来一个栗子：

```php
<?php
$pid = pcntl_fork();

switch($pid) {
    case -1:
        print "Could not fork!\n";
        exit;
    case 0:
        print "In child!\n";
        break;
    default:
        print "In parent!\n";
}
```

这个脚本如果fork成功的话，会显示“In parent!”和“In child!”两段话，但是里面没有涉及到变量的传递，我们再来一个栗子：

```php
<?php
for ($i = 1; $i <= 5; ++$i) {
    $pid = pcntl_fork();

    if (!$pid) {
        sleep(1);
        print "In child $i\n";
        exit;
    }
}
```

这里我们fork了5个子进程，变量$i是在父进程中，pcntl\_fork()后子进程获得了父进程的变量$i的副本（注意副本和引用的区别）。因为我们是在循环体中创建子进程，所以每个子进程执行的最后加了exit，防止子进程进入循环出现，

我们看下执行的结果：

![运行结果](1.gif)

注意子进程输出的顺序，这是多进程的基本原则之一：一旦你产生了进程，它是由操作系统决定什么时候被执行以及执行多少时间。虽然父进程运行完之后已经退出，但是子进程的信息还是打印出来了，如果我把sleep(1)；去掉，那么顺序和子进程在父进程结束之后的输入就没有这么明显，这里是为了更好地告诉大家，子进程事实上也是拥有自己的生命周期。

但是如果这样做，那么子进程的行为就不可把控以及难以调试了（孩子总是需要管的）。如果我们需要在父进程中管理子进程，需要通过两个新的函数来实现：pcntl\_waitpid()，它负责让父进程等待子进程；pcntl\_wexitstatus()，它获取一个已经停止的子进程的退出值，我们可以在子进程中通过exit()函数返回一个值，在父进程中用pcntl\_wexitstatus()来获取这个值。

在展示代码之前，我们先了解下两个函数怎么使用。首先，pcntl\_waitpid()至少需要两个参数，第一个参数是要等待哪个进程号的子进程，第二个参数是要存放子进程退出状态的变量。默认情况下，pcntl\_waitpid()将导致父进程无限期地暂停，等待子进程终止。当子进程退出时，pcntl\_waitpid()返回终止的子进程PID，然后把子进程的状态信息状态变量中（第二个参数）。

还有一种情况就是调用pcntl\_waitpid()的程序没有子进程在运行，则立即返回-1，状态变量不变。

函数的第一个变量必须是下面的值，

* < -1 等待任意进程组ID等于参数pid给定值的绝对值的进程。比如传入-2016，则等待进程号为2016的进程的子进程
* -1 等待任意子进程；与pcntl_wait函数行为一致。
* 0 等待任意与调用进程组ID相同的子进程，这是最常用的，就是等待当前进程号的子进程。
* > 0 	等待进程号等于参数pid值的子进程。比如传入2016，则等待进程号为2016的进程的子进程。

从子进程中返回一个值到父进程中需要用exit()函数而不是单纯的exit。父进程可以通过通过pcntl\_waitpid()的返回值判断子进程是否结束。

我们假设子进程是通过exit()退出的，那么我们可以通过pcntl\_wexitstatus()函数，

下面是代码，

```php
for ($i = 1; $i <= 5; ++$i) {
    $pid = pcntl_fork();

    if (!$pid) {
        sleep(1);
        print "In child $i\n";
        exit($i);
    }
}

while (pcntl_waitpid(0, $status) != -1) {
    $status = pcntl_wexitstatus($status);
    echo "Child $status completed\n";
}
```

![运行结果](2.gif)

## fork进程的资源重复问题

## pcntl_waitpid()的第三个参数


# 相关参考
[使用 GDB 调试 CoreDump 文件](http://blog.ddup.us/2011/08/28/debug-cores-with-gdb/)
[进程，线程，超线程，并发，并行等概念](http://www.oschina.net/question/733118_80913?fromerr=XvkPjLE2)