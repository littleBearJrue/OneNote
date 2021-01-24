# 面试之go基础

***************************************************************************基础线*************************************************************************

#### 1. go的变量是被分配在栈中还是堆中的？

答：go的全局变量是放在堆中的，函数内的局部变量是分配在栈中的，同时go的编译器会对内存做逃逸分析，发生内存跃迁时，之前分配在栈中的内存会被分配到堆中。所以是编译器自行决定了变量是分配在堆或者栈中，以保证程序的正确性

#### 2.for...range遍历里改变slice的值会怎么样？

答：go中for...range遍历中，slice拿到的是赋值的值拷贝，遍历过程中的任何改动对原本的切片不受影响

#### 3.for...range遍历里取value地址的时候会发生什么？

答：for...range过程中value的地址始终不变，改变的只是他所指向的值，所以最终取到的值都是同一个且相同

#### 4. make和new的区别是什么？

答：new返回是一个指针类型，并初始化给类型分配了零值。make是返回类型的引用。new多用于数组，指针。make适用于切片，map和通道

#### 5. 对nil的理解

答：nil是预先声明的标识符，可以标识切片，map，指针，通道，函数和接口类型



*************************************************************************map相关*************************************************************************

#### 1. map复制到新的map是深拷贝还是浅拷贝的？

答: go中map的赋值是浅拷贝的，新map的地址指向的是同个内存空间

#### 2. map是线程安全的吗？

答：map不是线程安全，在查找，赋值，遍历和删除中都会检测写标志，一旦发现写标志位等于1则直接panic。所以基于线程不安全的问题，在map的操作中一般都是要加锁，用读写锁的方式实现对map的操作，但这会极度影响了map的性能，尤其是在高并发环境下频繁的读写锁的操作会导致频繁的阻塞。官方提供了syncMap的接口，可以比较高效的解决map线程安全的问题。内部使用了read和dirty表来维护map的数据。以空间换时间的方式，读请求的话，首先会读read表，如果读取不到数据则从dirty表中加锁读取，如果从read表中读取不到key值的次数达到阈值，则将dirty表中的数据提升到read中；写表的话只写dirty表。但这样也会有一个问题就是当写请求远远大于读请求的时候，会频繁的触发写dirty表，而dirty是直接加锁处理的，这又会到了跟普通map表没啥区别了。还有另外的问题是syncMap还是对整个map表做读写锁的，粒度很大，所以尝试引入一种分级表的方式将map的粒度降低，定义一个区间范围，使用hash的方式选定一个所在区，将数据所对应的map保存在当前key中，缩小map的范围，使其在加锁粒度上更小。

#### 3. map可以直接比较大小吗？

答：map是不能直接比较大小的，同slice一样是不可比较类型大小的字段类型，只能通过遍历map的每个元素做比较判断两个map是否相等；或者使用反射reflect.deepEqual进行比较

#### 4. map是无序遍历的吗？为啥要做成无序遍历？

答：map的遍历是无序的，遍历无序的原因是因为map在做扩容的时候后，会发生key的搬迁，key可能会移动到别的bucket中，位置发生了变化导致遍历本身做不到有序输出；当然在不改变map表的时候做遍历也是无序的原因是因为这会形成不一致性，可能是为了理解的方便，导致底层实现中对map的遍历做了随机种子，所以遍历是无序的

#### 5. map是怎么扩容的？

答：map表的数据一旦达到阈值时，会触发扩容机制，刚开始会以2倍的方式进行，直到1024后，开始以1.25倍的方式进行。但底层实现了内存对齐的内存策略，所以不是严格上按照这个比例进行的



***********************************************************************channel相关*************************************************************************

#### 1. channel的两种模式适用于哪些场合？

答：Channel分为有缓冲类型和无缓冲类型。无缓冲类型时，发送了消息到channel的线程会阻塞，直到有其他线程从channel中接收消息。有缓冲类型，是可以指定缓冲的消息数量，当消息数量小于指定值时，不会出现阻塞，超过之后才会阻塞，需要等待其他线程去接收channel处理。channel可实现生产者消费者模式，也可以实现超时处理

#### 2. channel的底层数据结构：

``` go
type hchan struct {
    // chan 里元素数量
    qcount   uint
    // chan 底层循环数组的长度
    dataqsiz uint
    // 指向底层循环数组的指针
    // 只针对有缓冲的 channel
    buf      unsafe.Pointer
    // chan 中元素大小
    elemsize uint16
    // chan 是否被关闭的标志
    closed   uint32
    // chan 中元素类型
    elemtype *_type // element type
    // 已发送元素在循环数组中的索引
    sendx    uint   // send index
    // 已接收元素在循环数组中的索引
    recvx    uint   // receive index
    // 等待接收的 goroutine 队列
    recvq    waitq  // list of recv waiters
    // 等待发送的 goroutine 队列
    sendq    waitq  // list of send waiters

    // 保护 hchan 中所有字段
    lock mutex
}
```

特意说下recvq和sendq，一个是等待发送的go队列，一个是等待接收的go队列。

其中waitq是sudog的一个双向链表，而sudog实际上是对goroutine的一个封装。

#### 3. 什么情况下操作channel会出现死锁的可能？

答：

1. 向未声明的channel读写数据
2. 向已经声明的无缓冲的channel读数据

#### 4. 什么情况下操作channel会出现panic的可能？

答：

1. 关闭一个未初始化（nil）的channel会产生panic
2. 重复关闭同一个channel会产生panic
3. 向已经关闭的channel发送消息会产生panic

#### 5. channel中select是怎么使用的？

答：

1. select可以监听多个channel的写入或者读取
2. 执行select，若只有一个case通过，则执行此case
3. 若有多个case通过，则随机挑选一个case执行
4. 若所有的case均阻塞，且定义了default模块，则执行default模块。若无定义，则select语句阻塞，直到有case调用唤醒
5. 使用break可以跳出select循环块

#### 6. 如何较为优雅的关闭channel，不会导致panic或者产生死锁？

答：这个问题要从sender和receiver的数量来分别考虑使用的方案策略

1. 一个sender，一个receiver的情况下：直接从sender端发送完关闭即可，因为关闭发送端后，接收端还能从已关闭的channel读取剩余的数据

2. 一个sender，多个receiver的情况下：同上直接从sender端发送完关闭即可

3. 多个sender，一个receiver的情况下：增加一个传递关闭信号的channel，receiver通过信号channel下达关闭数据的channel数据，sender监听到关闭信号后，停止发送数据。

   ``` go
   func main() {
       rand.Seed(time.Now().UnixNano())
   
       const Max = 100000
       const NumSenders = 1000
   
       dataCh := make(chan int, 100)
       stopCh := make(chan struct{})
   
       // senders
       for i := 0; i < NumSenders; i++ {
           go func() {
               for {
                   select {
                   case <- stopCh:
                       return
                   case dataCh <- rand.Intn(Max):
                   }
               }
           }()
       }
   
       // the receiver
       go func() {
           for value := range dataCh {
               if value == Max-1 {
                   fmt.Println("send stop signal to senders.")
                   close(stopCh)
                   return
               }
   
               fmt.Println(value)
           }
       }()
   
       select {
       case <- time.After(time.Hour):
       }
   }
   ```



   ```bash
   注意: 
   对于一个 channel，如果最终没有任何 goroutine 引用它，不管 channel 有没有被关闭，最终都会被 gc 回收。所以，在这种情形下，所谓的优雅地关闭 channel 就是不关闭 channel，让 gc 代劳
   ```

4. 多个sender，多个receiver的情况下：增加一个中间人，M个receiver都向它发送关闭的请求，中间人收到请求后，就会下达关闭的通道，这样就不会出现重复关闭的问题。此时sender和receiver都监听关闭的channel，一旦收到关闭请求则return

   ``` go
   func main() {
       rand.Seed(time.Now().UnixNano())
   
       const Max = 100000
       const NumReceivers = 10
       const NumSenders = 1000
   
       dataCh := make(chan int, 100)
       stopCh := make(chan struct{})
   
       // It must be a buffered channel.
       toStop := make(chan string, 1)
   
       var stoppedBy string
   
       // moderator
       go func() {
           stoppedBy = <-toStop
           close(stopCh)
       }()
   
       // senders
       for i := 0; i < NumSenders; i++ {
           go func(id string) {
               for {
                   value := rand.Intn(Max)
                   if value == 0 {
                       select {
                       case toStop <- "sender#" + id:
                       default:
                       }
                       return
                   }
   
                   select {
                   case <- stopCh:
                       return
                   case dataCh <- value:
                   }
               }
           }(strconv.Itoa(i))
       }
   
       // receivers
       for i := 0; i < NumReceivers; i++ {
           go func(id string) {
               for {
                   select {
                   case <- stopCh:
                       return
                   case value := <-dataCh:
                       if value == Max-1 {
                           select {
                           case toStop <- "receiver#" + id:
                           default:
                           }
                           return
                       }
   
                       fmt.Println(value)
                   }
               }
           }(strconv.Itoa(i))
       }
   
       select {
       case <- time.After(time.Hour):
       }
   
   }
   ```

   #### 7. channel有哪些应用？

   答：

   1. 停止信号

   2. 任务定时

      ``` go
      // 超时控制
      select {
          case <-time.After(100 * time.Millisecond):
          case <-s.stopc:
              return false
      }
      
      // 定期执行某个任务
      func worker() {
          ticker := time.Tick(1 * time.Second)
          for {
              select {
              case <- ticker:
                  // 执行定时任务
                  fmt.Println("执行 1s 定时任务")
              }
          }
      }
      ```

   3.  解耦生成方和消费方

   4. 控制并发数（这里很巧妙！！！）

      ```go
      var limit = make(chan int, 3)
      
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



******************************************************************************GC相关*************************************************************************

#### 1. 根对象是什么？

答：也叫根集合，是垃圾回收器中在标记过程时最先检查的对象。

1. 全局变量
2. 执行栈：每个goroutine都包含自己的执行栈，执行栈上包含栈上的变量以及指向分配的堆内存区块地址
3. 寄存器:寄存器的值表示一个指针，参与计算这些指针可能指向某些赋值器分配的堆内存区块



#### 2. 常用的GC实现方式有哪些？

答：

1. 追踪式gc：
   * 标记清除
   * 标记整理
   * 分代式
2. 引用计数法

#### 3. go的gc是在堆中还是栈中进行的？

答：go的gc是在堆中进行的，堆中存的是全局变量还有内存发生逃逸后，从栈中分配到堆中的变量

#### 4. 怎么理解go的GC？

答：go的gc使用的是三色标记法。三色抽象规定了三种不同类型的对象，用白，灰，黑来表示：

1. 白色对象：指未被回收器访问到的对象。在回收开始阶段，所有对象均为白色；当回收结束后，白色对象均不可达。

2. 灰色对象：已被回收器访问的对象

3. 黑色对象：已被回收器访问到的对象，其中所有字段均已被扫描，黑色对象的任何一个指针都不可能直接指向白色对象

   工作流程：

   当回收开始时，所有对象均为白色对象。随着标记的过程的进行，白色对象开始开始着色为灰色，当灰色对象的所有子节点都完成扫描后，被着为黑色，其子节点对象被着为灰色。当整个堆被遍历完成后，只剩下可达的黑色对象和不可达的白色对象。此时白色对象即为可清理的对象

#### 5. 什么情况下会STW？

答：

1. 标记终止阶段，保证周期内标记任务的完成，停止写屏障
2. 清扫终止阶段，为下一阶段的并发标记做准备工作，此时启动写屏障

#### 6. 排查go gc的方式有哪些？

答：

1. GODEBUG=gctrace=1

2. go tool trace

   ```go
   func main() {
       f, _ := os.Create("trace.out")
       defer f.Close()
       trace.Start(f)
       defer trace.Stop()
       (...)
   }
   
   // 终端执行以下命令
   go tool trace trace.out
   ```

3. debug.ReadGCStats

4. runtime.ReadMemStats

#### 7. 在有gc存在的情况下，还有什么情况会出现内存泄露？

答：

1. 预期被快速释放的内存因被根对象（全局变量）引用而没有得到快速的释放
2. goroutine泄露：
   * 只有发送者没有接收者的情况下，go下的channel一直得不到释放

#### 8. 针对go，gc要怎么优化？

答：

1. 合理化内存分配的速度、提高赋值器的 CPU 利用率
2. 降低并复用已经申请的内存
3. 调整 GOGC大小

#### 9. go gc相关的api有哪些？分别起到什么作用？

答：

- runtime.GC：手动触发 GC
- runtime.ReadMemStats：读取内存相关的统计信息，其中包含部分 GC 相关的统计信息
- debug.FreeOSMemory：手动将内存归还给操作系统
- debug.ReadGCStats：读取关于 GC 的相关统计信息
- debug.SetGCPercent：设置 GOGC 调步变量
- debug.SetMaxHeap（尚未发布[10]）：设置 Go 程序堆的上限值
- 使用pprof查看





******************************************************************************go调度器相关********************************************************************

#### 1. go调度器的怎么理解？

答：go调度器简单理解即为GMP。其中G为goroutine，也就是我们日常用到协程，主要在栈中保存一些状态信息以及CPU的一些寄存器的值；M即为Machine，工作线程或者叫内核线程；P即为Processor,可理解为虚拟处理器，为M提供执行的上下文，保存M执行G时的一些资源，包括本地的G队列；三者的关系是，M需要P才能运行G，G需要M才能运行，M依赖P提供的资源，P则持有待运行的G队列;

工作的流程是这样的：

1. 从全局运行队列中寻找goroutine，为了保证调度的公平性，每个工作线程没经过61次调度就需要优先尝试从全局队列中找出一个g来运行，这样才能保证位于全局运行队列中的g得到调度的机会。
2. 未达到从全局队列中找g的次数时，则工作线程从本地队列中寻找g
3. 从本地队列中找不到则从其他的工作线程的运行队列中偷取g
4. 在进入休眠状态前会在一次尝试从全局队列中寻找g，如果找不到则进入network poller

#### 2. go的调度时机有哪些：

答：

1. 使用关键字go：会创建一个新的goroutine，go scheduler会优先考虑调度
2. GC：进行GC时，goroutine需要在M上运行
3. 系统调用：当 goroutine 进行系统调用时，会阻塞 M，所以它会被调度走，同时一个新的 goroutine 会被调度上
4. 内存同步访问：atomic，mutex，channel 操作等会使 goroutine 阻塞，因此会被调度走。等条件满足后（例如其他 goroutine 解锁了）还会被调度上来继续运行
5. time.sleep()

#### 3. go的调度是怎么的工作方式？

答：go的调度是协作式的调度，不会像线程那样，在时间片用完后，由CPU中断任务强行将其调度走。对于Go语言中运行时间过长的g, go scheduler有一个后台线程在持续监控，一旦发现go运行超过10s，会设置为go的"抢占标志位"，之后调度器会优先处理



#### 4. goroutine和thread的区别？

答：

1.  内存占用上：常见一个goroutine的栈内存消耗为2kb，实际运行过程中如果栈内存不够用，自动扩容；而一个thread则需要消耗栈内存2MB。
2. 创建和销毁的消耗上：goroutine由go runtime负责调度管理，是用户级别的，消耗很小；Thread的创建和销毁是要操作系统调用的，为内核级别的，因此消耗较大
3. 切换上：goroutine切换只需要保存三个寄存器，耗时很短；Thread切换需要保存各种寄存器的状态，以便后续恢复，耗时较长







 





