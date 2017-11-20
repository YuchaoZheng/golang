***sync.Mutex***

保证在每个时刻，只有一个goroutine能访问一个共享的变量从而避免冲突。

通过使用互斥锁(mutex)来提供这个限制。

Go 标准库中提供了 sync.Mutex 类型及其两个方法：
> Lock
> 
> Unlock

我们可以通过在代码前调用Lock方法，在代码后调用Unlock方法来保证一段代码的互斥执行。

我们也可以用defer语句来保证互斥锁一定会被解锁。

***sync.WaitGroup***

sync包中的WaitGroup实现了一个类似任务队列的结构，你可以向队列中加入任务，任务完成后就把任务从队列中移除，如果队列中的任务没有全部完成，队列就会触发阻塞以阻止程序继续运行。

某个地方需要创建多个goroutine，并且一定要等它们都执行完毕后再继续执行接下来的操作。

WaitGroup最大的优点就是.Wait()可以阻塞到队列中的任务都完毕后才解除阻塞。

```go
package main

import (
	"fmt"
	"sync"
)

var waitgroup sync.WaitGroup

func Afunction(shownum int) {
	fmt.Println(shownum)
	waitgroup.Done() //任务完成，将任务队列中的任务数量-1，其实.Done就是.Add(-1)
}

func main() {
	for i := 0; i < 10; i++ {
		waitgroup.Add(1) //每创建一个goroutine，就把任务队列中任务的数量+1
		go Afunction(i)
	}
	waitgroup.Wait() //.Wait()这里会发生阻塞，直到队列中所有的任务结束就会解除阻塞
}
```

***goroutine***

goroutine是由Go运行时环境管理的轻量级线程。

>go f(x,y,z) 表示开启一个新的goroutine执行 f(x,y,z)

goroutine是在相同的地址空间中运行，因此访问共享内存必须同步。

***channel***

channel是有类型的管道，可以用channel操作符 <- 对其发送或接收值。

>ch <- v //将 v 送入channel ch。
>v := <-ch  //从 ch 接收，并且赋值给 v。
>(“箭头”就是数据流的方向。)

 默认情况下，在另一端准备好之前，发送和接收都会阻塞。这使得 goroutine 可以在没有明确的锁或竞态变量的情况下进行同步。

***缓冲channel***

channel是可以**带缓冲**的。
向带缓冲的 channel 发送数据的时候，只有在缓冲区满的时候才会阻塞。 而当缓冲区为空的时候接收操作会阻塞。

***range和close***

发送者可以 close 一个 channel 来表示再没有值会被发送了。
循环 `for i := range c` 会不断从 channel 接收值，直到它被关闭。 

只有发送者才能关闭 channel，而不是接收者。向一个已经关闭的 channel 发送数据会引起 panic。
channel 与文件不同；通常情况下无需关闭它们。只有在需要告诉接收者没有更多的数据的时候才有必要进行关闭，例如中断一个range

``` go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

***select***

 select 语句使得一个 goroutine 在多个通讯操作上等待。

select 会阻塞，直到条件分支中的某个可以继续执行，这时就会执行那个条件分支。当多个都准备好的时候，会随机选择一个。

``` go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

```

