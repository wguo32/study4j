# 多线程总结

## 名称解释
- 重入：synchronized关键字加锁的方法，默认允许同一线程重新获取锁。[查看相关代码]()
- 指令重排：
    - 概念：JVM(编译器，处理器，运行时)提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能地提高并行度。
    - 问题：因此在没有同步的情况下，编译器、处理器以及运行时都可能对操作的执行顺序进行意想不到的调整，所以在非线程安全的类或者方法中，内存的操作顺序是无法得到保证的。
- 内存可见性：
    - 概念：JVM内存模型分为：主内存和线程独立的工作内存。Java内存模型规定，对于多个线程共享的变量，存储在主内存当中，每个线程都有自己独立的工作内存（比如CPU的寄存器），线程只能访问自己的工作内存，不可以访问其它线程的工作内存。
    ```
    如何保证多个线程操作主内存的数据完整性是一个难题，Java内存模型也规定了工作内存与主内存之间交互的协议，定义了8种原子操作：
    (1) lock:将主内存中的变量锁定，为一个线程所独占
    (2) unclock:将lock加的锁定解除，此时其它的线程可以有机会访问此变量
    (3) read:将主内存中的变量值读到工作内存当中
    (4) load:将read读取的值保存到工作内存中的变量副本中。
    (5) use:将值传递给线程的代码执行引擎
    (6) assign:将执行引擎处理返回的值重新赋值给变量副本
    (7) store:将变量副本的值存储到主内存中。
    (8) write:将store存储的值写入到主内存的共享变量当中。
    ```
    - 问题：通过上面Java内存模型的概述，我们会注意到这么一个问题，每个线程在获取锁之后会在自己的工作内存来操作共享变量，
           操作完成之后将工作内存中的副本回写到主内存，并且在其它线程从主内存将变量同步回自己的工作内存之前，共享变量的改变对其是不可见的。
           即其他线程的本地内存中的变量已经是过时的，并不是更新后的值。
- 非原子的64位操作和最低安全性
    - 概念：最低安全性：当线程在没有同步的情况下读取变量时，可能会得到一个失效值，但至少这个值是由之前某个线程设置的值，而不是一个随机值。这种安全性保证被称为最低安全性。
           非原子的64位操作：例外情况，非`volatile`声明的long和double类型变量，JVM允许将64位的读/写操作分解为两个32位的操作，从而会带来问题。
    - 问题：当线程A读取long变量时，线程B同时在写入，就有可能读取到刚写入的高32位和原来的低32位。
- 对象的发布和逸出:[详见代码]()
- 线程封闭
    - 概念：多线程环境中，要对共享变量进行同步，另一个简单的方法是，不提供共享变量，即所有数据仅对当前线程可见。即线程封闭
           常见示例：Swing，JDBC
    - 线程封闭的方式：
        1. ad-hoc线程限制(GUI单线程化)，
        2. 局部变量（栈封闭），
        3. ThreadLocal
- 不可变性(@Immutable)：不可变对象永远是线程安全的。
    - 不可变对象定义：对象的状态在创建后不能再被修改，它的所有域都是final类型，并且对象时正确创建的（没有发生this逸出）。
    - 任何线程可以在不需要额外同步的情况下安全的访问一个不可变对象。即使该对象发布时未用同步也没有关系。
- 事实不可变对象：若一个对象在安全发布后，程序中确定不会去修改它，那么也可以当做一个不可变对象使用。
    - 在没有额外同步的情况下，任何线程都可安全地使用被安全发布的事实不可变对象
- 实例封闭 
    - 将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程在访问数据时总能持有正确的锁。
    - 封闭机制更容易构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全性时就不需检查整个程序
- 客户端加锁机制
- 隐藏迭代器：如容器类的`toString()`，`hashCode`,`equals`等方法会迭代容器。
- 同步容器：带有`synchronize`关键字的容器，如：Vector，HashTable，Collection.synchronizedXXX。
          同步容器所有的操作都加了锁，将所有操作变成串行方式，严重降低了并发性，所以多线程环境下，性能会比较糟糕。
- 并发容器：Java5.0提供的`concureent`包下的容器。可以解决同步容器并发性能差的问题。
	- Concurrent-HashMap替代Hashtable
	- CopyOnWriteArrayList替代同步List
	- ConcurrentMap新增了一些对复合操作的支持，如：`若没有则添加，替换以及有条件删除等`
	- Queue和BlockingQueue

## 常见问题

### 哪些类是线程安全的？
- 无状态的对象一定是线程安全的：因为其并不包含对于其它类的引用，也就不涉及共享变量的问题。
- 不可变对象一定是线程安全的。

### 哪些类不是线程安全的？
- 竞态条件（Race Condition）：即当某个计算结果取决于多个线程的交替执行时序时，就会发生竞态条件。
  如：i++；i的结果取决于其自身的值，取决于线程调用过的次数，则i++这个操作会发生竞态条件
- 数据竞争（Data Race）：当多个线程同时读取一个非final共享字段时，会发生该问题？
- 先检查后执行操作：对象延迟初始化。

### 如何创建一个不可变类?
[详见代码]()。

### 如何安全的发布对象？
要安全地发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。一个正确构造的对象可以通过以下方法来安全地发布：
- 在静态初始化函数中初始化一个对象引用。
- 将对象的引用保存在volatile类型的域或者AtomicReference对象中。
- 将对象的引用保存在某个正确构造对象的final类型域中,(或者本身就是一个不可变对象)
- 将对象的引用保存在一个由锁保护的域中，(包括将对象放入线程安全的容器类中)

但是安全发布的对象，虽然保证了内存可见性，但是不能保证其是不可变的。对象的发布需求取决于它的可变性：
- 不可变对象可以通过任意机制来发布。
- 事实不可变对象必须通过安全方式来发布。
- 可变对象必须通过安全方式来发布，并且必须是线程安全的或者由某个锁保护起来。
    
### 如何安全的共享对象？
并发程序中使用和共享对象时，一些实用策略：
- 线程封闭。线程封闭的对象只能由一个线程拥有，对象被封闭在该线程中，并且只能由这个线程修改。
- 只读共享。在没有额外同步的情况下，共享的只读变量可以由多个线程并发访问，但任何线程都不能修改它。共享的只读变量包括不可变对象和事实不可变对象。
- 线程安全共享。线程安全的对象在其内部实现同步，因此多个线程可以通过对象的公有接口来进行访问而不需要进一步的同步。
- 保护对象。被保护的对象只能通过持有特定的锁来访问。保护对象包括封装在其他线程安全对象中的对象，以及已发布的并且由某个特定锁保护的对象。

### 如何设计一个线程安全类？
设计线程安全类的3个基本要素：
- 找出构成对象状态的所有变量。
- 找出约束状态变量的不变性条件。
- 建立对象状态的并发访问管理策略
要满足状态变量的有效值或状态转换上的各种约束条件，就需要借助于原子性与封装性。

### 如何扩展现有的线程安全类？
- 扩展现有的类，如继承Vector类。
- 客户端加锁（注意要使用正确的锁！）[参考代码](ListHelper)

### 有哪些常见的非线程安全例子？
- 多线程环境下，对线程安全的容器进行不加锁的复合操作，如`先检查再删除`等？
- 多线程环境下，多线程安全的容器进行迭代操作，解决办法是：客户端加锁（持有锁时间过长，有可能引发死锁等），更好的解决办法是复制容器，再迭代。

### 在多线程环境下，有哪些值得注意的问题？
- 多个同步的原子性操作组合成的复合操作，也有可能出现同步问题。
- 当执行时间较长的计算或者无法快速完成的操作时（如：I/O，网络连接，数据库访问）时，不要持有锁，否则会非常影响性能，以及可能带来死锁。
- 同步可变的变量时，不仅需要在修改它的地方同步，在访问它的地方也要实现同步，避免发生线程A持有锁在修改，而线程B同时在读取，不能保证读取的是线程A更新后的状态。
- 对于服务器端项目，在启动JVM时，一定要加上-server。因为-server模式会比-client做更多的优化，如将循环中不变的值提前到循环外部，这种行为模式的不同，可能会引发死循环。

### 多线程编程主要规则如下：
- 所有的并发问题都可以归结为如何协调对并发状态的访问，可变状态越少，就越容易确保线程安全性。
- 尽量将域声明为final类型，除非需要它们是可变的。
- 不可变对象一定是线程安全的。（不可变对象能极大地降低并发编程的复杂性，它们更为简单而且安全，可以任意共享而无需使用加锁或保护性复制等机制）。
- 封装有助于管理复杂性
- 用锁来保护每个可变变量
- 当保护同一个不变性条件中的所有变量时，需要使用同一个锁
- 在执行复合操作期间，要持有锁。
- 如果从多个线程中访问一个可变变量时没有同步机制，那么会出现问题
- 不要故作聪明地推断出不需要使用同步
- 在设计过程中考虑线程安全，或者在文档中明确地指出它不是线程安全的。
- 将同步策略文档化


## 关键字
### synchronized
- 加锁：用来同步代码块，以实现原子性操作：即同时只能有一个线程执行该代码块。常见锁：内置锁，即锁是当前对象this。
- 锁的作用不仅是一种互斥行为，也可以保证内存的可见性，可以保证所有线程都可以看到共享变量的最新值。

### volatile
- 作用：
	- 可以保证内存可见性，可以声明long和double类型变量，避免发生非原子的64位操作。
    - 可以防止指令重排
    - 并不能保证操作的原子性
- 使用场景：当且仅当满足以下所有条件时，才应该使用volatile变量：
	- 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。
    - 该变量没有包含在具有其他变量的不变式中。
    - 在访问对象时，不需要加锁。
    
### final
- final数据：
    - 1、编译期常量，永远不可改变（必须是基本类型）（编译期就可确定值，编译器可直接参与运算，）
    - 2、运行期初始化时，我们希望它不会被改变（可以是引用，也可以是基本类型）（引用类型仅仅是引用不可变，但是引用指向的对象时可变的）
- final方法：该方法不可被继承，覆盖。
- final类：该类不可继承，如String，Integer等包装类
- final参数：表示方法不可改变该参数。[注意：在匿名内部类中，为了保持参数的一致性，若所在的方法的形参需要被内部类里面使用时，该形参必须为final](http://www.cnblogs.com/chenssy/p/3390871.html)
- 运用final可以创建线程安全的不可变类

### ThreadLocal对象
- 概念：将某个全局变量或者单例对象保存一个副本到当前的Thread上
- 应用：


## 常见的线程安全类使用

### Vector和CopyOnWriteArrayList
- 同步容器

### Hashtable和ConCurrentHashMap
- 同步容器

### Collecctions.synchronizXXX
- 同步容器

### ConCurrentMap
- 并发容器


## 任务执行

### Executor
>`Executor`是java类库中的任务抽象。其是基于生产者-消费者模式。提交任务的相当于生产者，执行任务的相当于消费者。

- 执行策略： 定义了任务执行的`What,Where,When,How`等。最佳执行策略取决于可用的计算资源以及对服务质量的需求。
	- 在什么线程中执行任务？
	- 任务按照什么顺序执行（FIFO,LIFO,优先级）？
	- 有多少个任务能并发执行？
	- 在队列中有多少个任务在等待执行？
	- 如果系统由于过载而需要拒绝一个任务，那么应该选择哪一个任务？以及如何通知应用程序有任务被拒绝？
	- 在执行一个任务之前或之后，应该进行哪些动作？
- 线程池：管理一组同构工作线程的资源池，常用的有:`newFixedThreadPool`，`newCachedThreadPool`，`newSingleThreadExecutor`，`newScheduledThreadPool`等。
- 任务的生命周期：若不能正确的关闭Executor的生命周期，JVM有可能没有办法正常退出。
	- 运行：创建时处于运行状态
	- 关闭：shutdown执行平缓的关闭过程：不再接受新的任务，同时等待已提交的任务执行完成。shutdownNow：粗暴关闭，立即结束任务。
	- 已终止
- 延迟任务与周期任务：使用`ScheduledThreadPoolExecutor`代替`Timer`。因为Timer是单线程且不会抛出异常。

### Callable与Future
> Executor框架使用Runnable作为其基本的任务表示形式。但是Runnable的run方法不能返回一个值或者抛出受检查的异常。在解决有延迟的计算时，使用Callable与Future会带来更好的性能提升。

- 异构任务并行化中存在的局限性：即任务A和任务B并行操作，有时候并不能带来显著的性能提升，反而有可能带来更高的复杂度。
- 任务的时限：若某个任务无法再指定时间内完成，将不再需要其结果，此时就可以放弃这个任务。可以利用Future.get实现，若超出指定的时间，则抛出Timeout异常即可。

总结：通过围绕任务执行来设计应用程序，可以简化开发过程，并有助于实现并发。Executor框架将任务提交与执行策略解耦开来，同时还支持多种不同类型的执行策略。
当需要创建线程来执行任务时，可以考虑使用Executor。要想在将应用程序分解为不同的任务时获得最大的好长，必须定义清晰的任务边界。
某些应用程序中存在着比较明显的任务边界，而在其他一些程序中，则需要进一步分析才能揭示出粒度更细的并行性。


## 任务的取消和关闭
> 健壮的程序可以很完善的处理失败、关闭和取消等过程。因此任务的正确关闭和取消很重要。

- 取消：一个可取消的任务必须拥有取消策略：如读取某个标志位等(但是若线程是阻塞的，则有可能永远读不到该标识位)。
- 线程中断：
```
线程中断是一种协作机制，线程可以通过这种机制来通知另一个线程，告诉它在合适的或者可能的情况下停止当前工作，并转而执行其他的工作。
每个线程都有一个boolean类型的中断状态。当中断线程时，这个线程的中断状态将被设置为true。Thread类中，interrupt方法能中断目标线程（设置中断状态）。
isInterrupted方法能返回目标线程的中断状态。静态的interrupted方法将清除当前线程的中断状态，并返回它之前的值，这是清除中断状态的唯一方法。
阻塞库方法，如Thread.sleep和Object.wait，join等，都会检查线程何时中断，并且发现中断时提前返回。
它们在响应中断时执行的操作包括：清除中断状态，抛出InterruptedException，表示阻塞操作由于中断而提前结束。
当线程在非阻塞状态下中断时，它的中断状态将被设置，然后根据将被取消的操作来检查中断状态以判断发生了中断。这样，如果不触发InterruptException，那么中断状态将一直保持，直到明确地清除中断状态。
如何正确理解中断？中断并不会真正地中断一个正在运行的线程，而只是发出中断请求，然后由线程在下一个合适的时刻中断自己。 在使用静态的interrupted时应该小心，因为它会清除当前线程的中断状态。
如果在调用interrupted时返回了true，那么除非你想屏蔽这个中断，否则必须对它进行处理—可以抛出InterruptedException，或通过再次调用interrupt来恢复中断状态。 通常，中断是实现取消的最合理方式。
```
- 中断策略：由于每个线程都拥有各自的中断策略，因此除非你知道中断对于该线程的含义，否则就不应该中断这个线程。
- 响应中断：
- 守护线程：线程分为：普通线程和守护线程两种。JVM启动时，除了主线程以外，其余线程均为守护线程（如垃圾回收器）。普通线程与守护线程的差异仅仅在于退出时的操作。


## 线程池的使用

### 线程饥饿死锁
在线程池中，如果任务A依赖相同线程池中其它的任务，那么可能产生死锁。
只要线程池中的任务需要无限期地等待一些必须由池中其他任务才能提供的资源或者条件，例如某个任务等待另一个任务的返回值或者执行结果，那么除非线程池足够大，否则将发生线程饥饿死锁。

### 运行时间较长的任务
当线程池中的任务运行时间都比较长时，可能会影响本来应该运行比较短时间的任务的运行。
解决方法是：
- 设置超时时间，若等待超时，则把任务标识为失败等。
- 增加线程池的大小

### 线程池的大小设置多大比较合适？
- 线程池大小设置需要考虑多方面的因素：如CPU个数，内存大小，任务的类型（I/O密集型还是计算密集型）等。
- 对于计算密集型的任务，在N个CPU的机器上，通常线程池设置为N+1效率会比较高。
- 对于I/O或者其它阻塞型密集型操作，线程池规模应该适当增大。
- Runtime.getRuntime().availableProcessors()可以获取可用CPU的数量。

### 线程池中线程何时创建和销毁？
- 线程池的基本大小(Core Pool Size)，最大大小(Maximum Pool Size)以及存活时间等因素共同负责线程的创建与销毁。
- 基本大小就是在没有任务执行时线程的默认大小，当基本大小的线程池满了时，会创建新的线程。
- 若某个线程的空闲时间超过了存活时间，会被标记为可回收的，并且当线程池的当前大小超过了基本大小时，这个线程将被终止。
- 所以可以通过调节线程池的基本大小和存活时间，来帮助线程池回收空闲线程占用的资源，从而使得这些资源可以用于执行其他工作。
- `newFixedThreadPool`工厂方法将线程池的基本大小和最大大小设置为参数中指定的值，并且线程不会超时
- `newCachedThreadPool`工厂方法将线程池的最大大小设置为Integer.MAX_VALUE。初始大小为0，超时为1分钟。

### 线程池如何管理待处理的任务队列
- 若新请求到达速率大于线程池的处理速率，那么新到来的请求将累积起来，在线程池中，这些请求会在一个由Executor管理的Runnable队列中等待，而不会像线程那样去竞争CPU资源。若Runnable过多，仍会耗尽资源。
- 任务排队方法：无界队列，有界队列和同步移交。队列的选择与配置参数有关，如线程池的大小。
- `newFixedThreadPool`和`newSingleThreadExecutor`在默认情况下使用无界的LinkedBlockingQueue。可以无限制的增加待处理任务。
- 更好的排队策略是使用有界队列：如:ArrayBlockingQueue,有界的LinkedBlockingQueue、PriorityBlockingQueue。
- `newCachedThreadPool`使用SynchronousQueue.可以避免排队，即同步移交。来任务A，直接将其移交至线程A。要想将新任务放到SynchronousQueue中，必须有一个线程在等待接受该任务。
- 只有当任务相互独立时，为线程池和工作队列设置界限才是合理的，若任务直接存在依赖性，那么有界的线程池可能会导致`饥饿`死锁问题。此时应该使用无界线程池，如：newCachedThreadPool

### 线程池饱和策略
- 当提交的任务达到排队任务的上界时，需要处理多余的请求，此时需要用到饱和策略来处理
- 中止策略(AbortPolicy)：默认的策略，直接抛出异常。
- 抛弃策略(DiscardPolicy)：抛弃该任务
- 抛弃策略(DiscardOldestPolicy)：抛弃最老的任务，不能和优先者队列结合使用
- 调用者运行策略(CallerRunsPolicy)：一种调节机制，如：若工作队列已经满了，新任务A来时，将A委托给主线程运行，此时主线程阻塞，不会再接受新的请求，将请求阻塞在TCP层，或者客户端。从而达到在高负载下服务性能的逐渐降低。

### 线程工厂
- 每当线程池需要创建一个线程时，都是通过线程工厂方法来完成的。可以继承`ThreadFactory`类来定制化线程工厂。

### ThreadPoolExecutor定制化
- 可以使用ThreadPoolExecutor.setXXX进行定制。
- 若不想暴露setXXX方法，可以使用unconfigurableExecutorService方法对现有的ExecutorService进行包装。

### ThreadPoolExecutor可扩展
- beforeExecute|afterExecute|terminated都是可扩展的，如可以使用这个实现日志记录，信息统计，释放资源等操作。

### 循环和递归可以使用线程池来提高运算效率

>总结：
>对于并发执行的任务，Executor框架是一种强大且灵活的框架。
>它提供了大量可调节的选型，例如创建和关闭线程的策略，处理队列任务的策略，处理过多任务的策略，并且提供了几个钩子方法来扩展它的行为。

## 活跃度危险

### 死锁
- 锁顺序死锁。分为：简单锁顺序死锁，动态锁顺序死锁（加锁对象为方法的参数）
    - 如果同时获取两个锁，但是在不同线程中获取锁的顺序不一样，就会发生死锁。如果所有线程已固定的顺序获取锁，那么程序中就不会出现锁顺序死锁问题。
    - 如果在持有锁的同时调用外部方法，则需要注意是否存在死锁，因为若外部方法也持有锁，则此时需要获取两个锁。
    - 要解决调用外部方法可能引起的死锁问题，则需要使用`开放调用`：即调用方法时，不需要获取锁
 
- 资源死锁。包括：线程饥饿死锁，同时对两个资源争夺导致的死锁
- 如何避免死锁
    - 每次只获取一个锁
    - 若获取多个锁，则需要保证在全局范围内，它们的获取顺序是一样的
    - 最好使用`开放调用`
    - 使用定时锁`tryLock`
- 通过线程转储信息来分析死锁
	- UNIX下向JVM发送SIGQUIT信号(kill -3)

### 饥饿
- 当线程由于无法访问它的资源而不能继续执行时，就发生了`饥饿`。如无限循环等。
- 应该避免使用线程优先级，因为这会增加平台依赖性，并可能导致活跃性问题。在大多数并发应用中，都可以使用线程的默认优先级。


### 糟糕的响应性
- 锁占用时间较长可能造成程序响应延迟

### 活锁
- 活锁是另一种形式的活跃性问题，该问题不会造成线程阻塞，但是也不能继续执行，因为线程会不断重复执行相同的操作，并且总是会失败。
- 活锁通常会发生在处理事务消息的应用程序中，如：事务A发生错误，造成了回滚，此时消息处理机制又将其放置队列首部，则又会执行事务A。从而导致失败。


>总结：
>活跃性故障是非常严重的问题，因为当活跃性故障发生时，只有重启应用。
>最常见的是锁顺序死锁。所以在设计时应该避免产生锁顺序死锁：确保线程在获取多个锁时采用一致的顺序。
>其中最好的解决办法是在程序中始终使用开放调用。这将大大减少需要同时持有多个锁的地方，也更容易发现这些地方。


## 性能与可伸缩性

可伸缩性指的是：当增加计算资源时(例如CPU，内存，存储容量或者I/O带宽)，程序的吞吐量或者处理能力能够相应的增加

不用过度担心非竞争同步带来的开销，因为这个基本的机制已经非常快乐，并且JVM能够进行额外的优化以进一步降低或者消除开销。因此，我们应该将优化重点放在那些发生锁竞争的地方。
发生锁竞争时会发生阻塞：JVM在实现阻塞行为时，可以采用#自旋等待（不断尝试，直到成功）#，或者通过操作系统挂起被阻塞的线程。选择哪种方式主要取决于阻塞的时间。

### 提高性能和可伸缩性的方式
- 减少锁的竞争
	- 减少锁的持有时间
	- 降低锁的请求频率
	- 使用带有协调机制的独占锁，这些机制允许更高的并发性。
- 缩小锁的范围：可以减少锁的持有时间
- 减小锁的粒度：如果一个锁同时保护多个相互独立的变量，可以考虑将其拆分成多个独立的锁。
- 锁分段：例如ConcurrentHashMap。将一个锁拆分成多个锁，每个锁只负责其中的一小块。可以提高并发量，适合锁竞争十分激烈的场景。
  如果采用锁分段技术，那么一定要表现出在锁上的竞争频率高于在锁保护的数据上发生竞争的频率。例如锁Y保护独立变量a和b。线程A访问a，线程b访问b，此时变量a和b并没有竞争需求，但是锁需要竞争，此时适合锁分段。
- 使用其它方式代替独占锁，如使用并发容器，原子变量，不可变对象或者读写锁等(ReadWriteLock)

### CPU利用率检测
检测手段：
- UNIX：vmstat或者mpstat
- Windows：perfmon

影响因素：
- 负载不充足
- I/O密集
- 外部限制
- 锁竞争

>总结：
>由于使用线程是为了充分利用多个处理器的计算能力，因此在并发程序性能的讨论中，通常更多地将侧重点防止吞吐量和可伸缩性上，而不是服务时间。
>Amdahl定律告诉我们，程序的可伸缩性取决于在所有代码中必须被串行执行的代码比例。
>因为Java程序中串行操作的主要来源是独占方式的资源锁，因此通常可以通过以下方式来提升可伸缩性：
	- 减少锁的持有时间
	- 降低锁的粒度
	- 采用非独占的锁或非阻塞锁代替独占锁



## 性能测试

### 性能测试的指标
- 吞吐量：指一组并发任务中已完成任务所占的比例
- 响应性：指请求从发出到完成之间的时间(延迟)
- 可伸缩性：指在增加更多资源的情况下(通常指CPU)，吞吐量(或者缓解短缺)的提升情况。


## 显示锁

- 与内置加锁(synchronized)机制不同的是，Lock提供了一种无条件的、可轮询的、定时的以及可中断的锁获取操作，所有加锁和解锁的方法都是显示的。

### ReentrantLock（重入锁）

- 锁的公平性：`ReentrantLock`的构造函数提供了两种公平性的选择：非公平的锁（默认）和公平性的锁。
			由于公平性的锁性能要比非公平性的锁低很多，所有大多数情况下，用默认的锁就好。造成性能底的主要原因是线程排队和切换带来的开销。
- `ReentrantLock`与`synchronized`相同点：
	- 互斥性
	- 内存可见性
	- 可重入性
- `ReentrantLock`与`synchronized`不同点：
	- `ReentrantLock`使用起来复杂一些，必须在finally块中释放锁，否则发生异常，该锁则永远无法释放。
	- `synchronized`的程序有可能会发生死锁。而`ReentrantLock`的`tryLock`可以实现轮询锁，从而避免发生死锁问题。
	- `ReentrantLock`的`tryLock`可实现带有时间限制的加锁。
	- `ReentrantLock`的`lockInterruptibly`可以实现可中断的锁。
	- `ReentrantLock`可实现非块结构的加锁：如锁分段技术，连锁式加锁或者锁耦合技术。
	- `ReentrantLock`可实现公平队列的锁竞争。

- `ReentrantLock`与`synchronized`该怎么选择?
   - `ReentrantLock`和`synchronized`的语义基本相同，且有更好的性能和更多的特性。
   - 但是`synchronized`更好理解且简洁紧凑。`ReentrantLock`需要手动管理锁的释放，危险性更高。
   - 在一些内置锁无法满足需求的情况下，`ReentrantLock`可作为一种高级工具。当需要一些高级功能时才应该使用`ReentrantLock`，这些功能包括：可定时的、可轮询的与可中断的锁获取操作，公平队列，以及非块结果的锁。否则还是应该优先使用`synchronized`。
   
### ReadWriteLock(读写锁)
`ReentrantLock`与`synchronized`都是一种标准的互斥锁，但是对于大多数情况来说，互斥锁通常是一种过于强硬的加锁规则，因此也就不必要地限制了并发性。
主要原因是由于：互斥锁不仅限制了`写/写`,`写/都`还限制了`读/读`。而大多数数据结构大部分都是读操作，因此若能够放宽读操作的加锁请求，就能够提升程序的性能。
   
- ReadWriteLock中提供了一些读取锁与写入锁直接的交互方式
	- 释放优先
	- 读线程插队
	- 重入性
	- 降级
	- 升级
	
- ReentrantReadWriteLock是ReadWriteLock的实现类。可以用来包装HashMap等，以提高其读写性能。


## 原子变量和非阻塞同步机制

java.util.concurrent包中许多类都提供了比synchronized机制更高的性能和可伸缩性，主要就是因为：原子变量和非阻塞的同步机制

- 锁的缺点：
	- 对于很多细粒度的操作，采用独占操作很浪费资源。
	- 有可能造成死锁等活跃性问题
	- 会造成线程阻塞，影响整体性能
	- volatile变量虽然可以解决锁独占的问题，但是其没有提供原子性的复合操作。
	
- 硬件对于并发操作的支持：CAS
	- CAS（Compare-and-Swap）：比较并交换，即处理器执行更新操作时，先将值V和希望的值A进行比较，若V==A，则将V更新为B，则更新成功。否则失败，并允许失败的线程重试。这样就避免了多线程问题。
	- CAS可以实现非阻塞的计数器等。通常CAS的性能要比锁高很多，因为其避免了很多的线程切换，系统调度等操作。
	- 可以实现非阻塞的链表，非阻塞的栈等
	- 存在ABA问题

- 原子变量类CAS
    - 原子变量可以代替volatile变量，并且可以实现复合的原子操作：读-改-写，自增等。
    - 原子变量类都实现了CAS
	- 共有12个原子变量类：分为标量类（AtomicInteger,AtomicLong,AtomicBoolean,AtomicReference）,更新器类，数组类，复合变量类。
	- 原子变量类是可变类，且没有实现hashCode，equals，所有不能作为hashcode的键值。

>总结：
>非阻塞算法通过底层的并发原语(例如比较并交换而不是锁)来维持线程的安全性。这些底层的原语通过原子变量类向外公开，这些类也用做一种"更好的volatile变量"，从而为整数和对象引用提供原子的更新操作
>非阻塞算法在设计及实现上比较困难，建议只使用已有的类库。


## Java的内存模型
- 多处理器的体系架构下：每个处理器都有自己的缓存器（寄存器等），无法保证程序的串行一致性。因此需要注意使用加锁机制。
- 重排序导致的问题
- 使用安全的初始化模式：加锁，原子性等。
- 不要使用`双重检查加锁`机制。
```java
@NotThreadSafa
public class DoubleCheckedLocking {
	private static Resource resource;
	public static Resource getInstance() {
		if (resource == null){//非同步操作，有可能此时Resource未初始化完成，则有可能获取到一个失效的值。
			synchronized (DoubleCheckLocking.class) {
				if (resource == null) {
				    resource = new Resource();
				    //do something 
				}
			}
		}
		
		return resource;
	}
}
```























