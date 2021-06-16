---
title: "聊聊 “协程” Coroutine"
date: 2021-06-16T22:24:34+08:00
tags: ["theme"]
categories: ["starting"]
banner: "img/banners/banner-1.jpg"
---

> 最近问团队的小伙伴希望我写一些什么类型的文章给团队内学习的时候. 有人提到了golang的协程(因为现在很多小伙伴可能是php+golang).

在聊协程之前, 我们还是得先聊聊 `并行(parallel)` 跟 `并发(concurrent)`.  那么什么是并发计算呢?

我们用gif图来解释. 方块变成紫色就表示cpu在处理一个子程序. 当我们的程序没有并发的时候. 它像这样. 每一秒只有当一个方块变成紫色后

另一个方块才能变成紫色. 就如同我们的CPU在单核的时候. 同一个时刻. 只能运行一个子程序. 这个时候我们会觉得. 程序好像很卡的样子. 每个方块等待的时间都很长.

这就是串行执行。

![](https://other.wkcoding.com/2.gif)

然后呢, 我们调整我们的程序. 让每一个方块在亮一刻之后. 将亮的权力交接给另一个方块. 两个不断的让出权力. 使我们觉得CPU好像可以同时把两个方块点亮.

这就是并发执行. 当然, 现实的计算机例子实现并发细节还有很多, 这里只是用简单的模型让大家理解

![](https://other.wkcoding.com/1.gif)

那么什么是并行呢, 现在的CPU大部分都是双核, 四核, 或者八核.
那前面我们也说了.

> 我们的CPU在单核的时候. 同一个时刻. 只能运行一个子程序. 

所以当我们的CPU只有单核的时候, 我的理解是. 不存在并行执行. 那当我们的CPU是双核的时候. 我们就可以像下面的gif图一样.
每次可以亮起两个紫色方块。(截图就可以看到了)

这个就是并行执行。

![](https://other.wkcoding.com/3.gif)

ok, 大概理解了 `并行(parallel)` 跟 `并发(concurrent)` 后. 我们就可以聊聊. 我们今天讨论的主角. `"协程"` 了。

协程的概念很早就提出来了，但火起来, 我记得是因为golang火导致的. 其实golang中的goroutine跟很多语言里面的协程, 比如 php，javascript(新版v8引擎) 等这些语言的协程都不太一样. 所以golang的协程叫`goroutine`, 而不是 `coroutine`. 很多文章阐述 `coroutine` 跟 `goroutine` 的区别我个人觉得都不太正确. 今天就来聊聊我理解的协程.  `非抢占式协程`

对于php协程而言. [Cooperative multitasking using coroutines (in PHP!)](http://nikic.github.io/2012/12/22/Cooperative-multitasking-using-coroutines-in-PHP.html)) 这篇文章就使用了php5的新特性 yield 实现了一个协程调度器.

[sabre-io/event](https://github.com/sabre-io/event) 也利用yield实现的协程调度器.

而比较新的v8源码里面也能找到对应的协程支持.

[runtime-generator.cc](https://github.com/v8/v8/blob/0fd6ef88d5fd3dcfa30b4e337288dd30a0dad882/src/runtime/runtime-generator.cc#L15)

[js-generator.h](https://github.com/v8/v8/blob/29d08f1cd8ad3b3069bff726712f62f6826acddb/src/objects/js-generator.h#L16)

那么我们再来看看go的goroutine. 非常简单的示例代码.

> 我关注了一下最新golang的release notes. 发现 go1.14 以后对 goroutine scheduler进行了重构. 改成了抢占式调度. 所以本文的golang代码使用的是go1.13版本的. 推荐大家使用gvm进行golang版本管理

```golang
package main

func main() {
	for i := 0; i < 5; i++ {
		go println(i)
	}
	for {
	}
}

// terminal output
➜  blog_temp go run ./test.go
1
0
2
4
3
```

大部分前端跟php开发小伙伴的疑惑可能都是. 为什么nodejs, php的协程. 输出还是按顺序的。 而golang的协程我们可以看到. 是不固定顺序的.

我们把golang代码改成这样再试试
> runtime.GOMAXPROCS(1) 将golang使用的CPU核数改成了1. 也就是跟nodejs, php 一样的单核
```golang
package main

import (
	"runtime"
)

func main() {
	runtime.GOMAXPROCS(1)
	for i := 0; i < 5; i++ {
		go println(i)
	}
	for {
		// 
	}
}

// terminal output
➜  blog_temp go run ./test.go
```

我们会发现. 没有一点输出. 这是因为执行到 `for` 循环的时候. 主程进入了 无限循环. 由于是 非抢占式协程的原因. 我们在 `for` 循环里面没有主动让出程序执行权. 导致子协程无法被调度。要解决这一点。我们只需要在 `for` 循环里面进行一些io操作让出执行权即可.

```golang
package main

import (
	"runtime"
	"time"
)

func main() {
	runtime.GOMAXPROCS(1)
	for i := 0; i < 5; i++ {
		go println(i)
	}
	for {
		time.Sleep(time.Second)
	}
}

// terminal output
➜  blog_temp go run ./test.go
0
1
2
3
4
```

现在可以看到. 输出跟nodejs, php中的协程一样. 都是按顺序执行的。

所以协程其实就是子程序之间通过一系列方式切换执行权. 使我们 『看起来好像多个任务在同时执行』。

所以, 在CPU是单核的情况下. 协程对于性能来说. 其实是没有什么作用的. 因为协程不能像 `进程` 一样缩短运行时间.

那么协程为什么可以提高并发能力呢. 我们假设我们有一个API Server. 单进程, 单线程的情况下. 在不考虑所有因素, API 整体处理完需要50ms, (不考虑网络情况.) 单指API处理的时间. 然后每次固定有5个请求同时进来. 

没有协程版本运行情况应该如图

![image](https://user-images.githubusercontent.com/20190812/91273805-5ab08680-e7b0-11ea-9a98-228aeaa5d988.png)

第一个请求会非常快. 但是到后面每一个请求的响应时间都是叠加的. 这样会导致后面的请求响应时间越来越高. 以至于最后一个请求的用户, 会有系统好卡的感觉。

但是我们改造成协程版本. 将API拆成每个10ms的子程序. 交替运行. 

![image](https://user-images.githubusercontent.com/20190812/91274321-1376c580-e7b1-11ea-990c-56cd55134439.png)

给到每个用户的感觉延迟都非常平均. 非常线性. 所以这样就大大提高了我们的并发能力. 非协程版本. 第5个用户就已经到了600ms的延迟. 那第6个就是1200ms. 而协程版本将延迟平均到每个用户.到并发6个的时候大家也只有300ms的延迟.

### Reference
- http://nikic.github.io/2012/12/22/Cooperative-multitasking-using-coroutines-in-PHP.html
- https://yourbasic.org/golang/goroutines-explained/