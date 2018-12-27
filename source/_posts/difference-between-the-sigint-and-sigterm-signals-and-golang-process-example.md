---
title: 系统信号SIGINT和SIGTERM的区别以及GOLANG中信号处理
categories:
  - 编程语言
  - Go
tags:
  - Signal
---

Signal是Linux，类Unix和其它POSIX兼容的操作系统中用来进程间通讯的一种方式。一个信号就是一个异步的通知，发送给某个进程，或者同进程的某个线程，告诉它们某个事件发生了。

当信号发送到某个进程中时，操作系统会中断该进程的正常流程，并进入相应的信号处理函数执行操作，完成后再回到中断的地方继续执行。果目标进程先前注册了某个信号的处理程序(signal handler),则此处理程序会被调用，否则缺省的处理程序被调用。

<!--more-->

下面代码是Go演示的信号处理部分代码，

```go
import (
	"os"
    "os/signal"
)

func main() {
    // ...
    sc := make(chan os.Signal, 1)
    signal.Notify(sc,
		syscall.SIGINT,
		syscall.SIGTERM,
		syscall.SIGQUIT,
		syscall.SIGPIPE,
		syscall.SIGUSR1,
    )
    
    go func() {
        for {
            sig := <-sc
            if sig == syscall.SIGINT || sig == syscall.SIGTERM || sig == syscall.SIGQUIT {
            	// 关闭连接，保存数据等操作
            } else if sig == syscall.SIGPIPE {
                // 忽略
            } else if sig == syscall.SIGUSR1 {
                // 用户自定义的信号，可以用来做一些进程内操作的动作，例如更新配置、重启服务等
            }
        }
    }
    // ...
}
```

有两种信号不能被拦截和处理，SIGKILL和SIGSTOP。

其中SIGTERM和SIGKILL的区别是，如果有信号处理程序，通过SIGTERM信号监听可以优雅的退出程序，我们可以通过下面的漫画了解。

![cartoon](https://jack-images.wilead.net/2018-12-27-033950.png)

最后，附上几个重要（守护进程或者服务需要处理）的Signal列表，

| 信号    | 值       | 动作      | 说明                                                    |
| ------- | -------- | --------- | ------------------------------------------------------- |
| SIGINT  | 1        | Terminate | 用户发送INTR字符(Ctrl+C)触发                            |
| SIGQUIT | 3        | Core      | 用户发送QUIT字符(Ctrl+/)触发                            |
| SIGKILL | 9        | Terminate | 无条件结束程序(不能被捕获、阻塞或忽略)                  |
| SIGPIPE | 13       | Terminate | 消息管道损坏(FIFO/Socket通信时，管道未打开而进行写操作) |
| SIGTERM | 15       | Terminate | 结束程序(可以被捕获、阻塞或忽略)                        |
| SIGUSR1 | 30,10,16 | Terminate | 用户保留                                                |
| SIGUSR2 | 31,12,17 | Terminate | 用户保留                                                |
| SIGSTOP | 17,19,23 | Stop      | 停止进程(不能被捕获、阻塞或忽略)                        |

详细可以[点击这里](http://man7.org/linux/man-pages/man7/signal.7.html)查看。

综上，我们可以通过SIGUSR1或SIGUSR2实现程序的平滑重启，可以通过捕获信号SIGTERM来“温柔”地终止程序（让程序可以安全地退出）。

关于实现程序的平滑重启，我会另外开博文，这里先安利下一个库 [fvbock/endless](https://github.com/fvbock/endless)。

现在你知道为什么不要轻易 kill -9 某个进程了吧？