***sync.Mutex***

保证在每个时刻，只有一个goroutine能访问一个共享的变量从而避免冲突。

通过使用互斥锁(mutex)来提供这个限制。

Go 标准库中提供了 sync.Mutex 类型及其两个方法：
> Lock
> Unlock

我们可以通过在代码前调用Lock方法，在代码后调用Unlock方法来保证一段代码的互斥执行。

我们也可以用defer语句来保证互斥锁一定会被解锁。