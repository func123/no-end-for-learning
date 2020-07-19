[TOC]

# channel

## 什么是channel

channel和goroutine是go并发的两大基石。goroutine用于执行并发任务，而channel用于goroutine之间的同步、通信。它在goroutine之间架起一条管道，在管道中传递数据，它是线程安全的、具有“先进先出“的特性，以及能够影响goroutine的阻塞和唤醒

## channel的实现

channel分为两种：带缓冲，不带缓冲。

对不带缓冲的channel可以看成是同步模式，发送方和接收方都必须要在同步就绪的情况下，数据才能在两者间传输。否则任意先进行操作的一方都将被挂起，等待另一方出现才能被唤醒。

而对带缓冲的channel可以看成是异步模式，在有缓冲可用的情况下，就是对发送数据到channel中来说，缓冲中必须要有空余容量，对从channel中接收数据来说，缓冲中必须要有数据，这样的情况下发送和接受的操作就都能顺利进行，否则，操作的一方会被挂起。

当操作阻塞时，channel会把当前操作的goroutine封装在一个叫sudog的结构体中，放入到对应操作的阻塞队列（双端循环链表）后，调用gopark 函数，更改goroutine的状态为waiting，将其挂起，等待合适的时机再唤醒。

在发送和接收操作中，也会唤醒goroutine执行。例如在发送操作中，会从接收阻塞队列中取出粗赛的goroutine放入到P的可运行队列中，调用goready函数将其唤醒，把状态从waiting变成runnable，当被调度后，状态变成running，就能继续执行接下来的代码。

## channel数据的传输

在channel中，数据传输是通过将数据从发送者的栈拷贝到接受者的栈上实现的。sudog 通过 `elem` 字段绑定待发送元素或待接收元素的地址

## channel关闭

从关闭的channel中读数据，接受者会收到一个相应类型的零值，而向一个关闭的channel写数据会发生panic。在关闭函数中，会把阻塞队列中的goroutine中全部唤醒，此时被唤醒的发送者检测到 channel 已经关闭就会发生panic

## channel发生panic的情况

1. 向一个关闭的channel进行写操作
2. 关闭一个nil channel
3. 重复关闭一个channel

## channel应用

1. 停止信号。通过关闭channel或者向channel发送一个元素，使得接收方知道此信息

2. 任务定时。与timer结合，实现超时控制，定期执行

3. 控制并发数

   ```go
   var limit = make(chan int,3)
   func main() {  
   // …………  
   for _, w := range work {
           go func() {
               limit <- 1
               w()
               <-limit
           }()
       }    
   // …………
   }
   ```

   



