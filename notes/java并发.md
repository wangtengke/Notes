# Java并发
## [关键字](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#关键字-1)
- [synchronized](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#synchronized)
- [volatile](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#volatile)

## [Java内存模型](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#java内存模型-1)
- [Java内存模型介绍](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#java内存模型介绍)
- [happens-before](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#happens-before)
- [重排序](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#重排序)
- [从JMM角度分析DCL](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#从jmm角度分析dcl)
- [总结](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#总结)

## [J.U.C](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#juc-1)
- [AQS](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#aqs)
- [重入锁与读写锁](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#重入锁与读写锁)
- [并发工具类](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#并发工具类)
- [非阻塞同步](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#非阻塞同步)  
## [线程池](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#线程池-1)
- [线程池原理](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#线程池原理)
- [java Executors类四种线程池](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#java-executors类四种线程池)
# 关键字
## synchronized
**实现原理**

synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性

Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：

1. 普通同步方法，锁是当前实例对象
2. 静态同步方法，锁是这个类
3. 同步方法块，锁是括号里面的对象
4. 同步一个类，锁是这个类

当一个线程访问同步代码块时，它首先是需要得到锁才能执行同步代码，当退出或者抛出异常时必须要释放锁。

**锁优化**
- **自旋锁**

线程的阻塞和唤醒需要CPU从用户态转为核心态，频繁的阻塞和唤醒对CPU来说是一件负担很重的工作，势必会给系统的并发性能带来很大的压力。同时我们发现在许多应用上面，对象锁的锁状态只会持续很短一段时间，为了这一段很短的时间频繁地阻塞和唤醒线程是非常不值得的。所以引入自旋锁。

何谓自旋锁？ \
所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。怎么等待呢？执行一段无意义的循环即可（自旋）。

自旋等待不能替代阻塞，先不说对处理器数量的要求（多核，貌似现在没有单核的处理器了），虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，典型的占着茅坑不拉屎，这样反而会带来性能上的浪费。所以说，自旋等待的时间（自旋的次数）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。

自旋锁在JDK 1.4.2中引入，默认关闭，但是可以使用-XX:+UseSpinning开启，在JDK1.6中默认开启。同时自旋的默认次数为10次，可以通过参数-XX:PreBlockSpin来调整；

如果通过参数-XX:preBlockSpin来调整自旋锁的自旋次数，会带来诸多不便。假如我将参数调整为10，但是系统很多线程都是等你刚刚退出的时候就释放了锁（假如你多自旋一两次就可以获取锁），你是不是很尴尬。于是JDK1.6引入自适应的自旋锁，让虚拟机会变得越来越聪明。
- **锁消除**

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：
```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```
String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：
```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```
每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

- **锁粗化**

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

- **轻量级锁**

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

以下是 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。除了 marked for gc 状态，其它四个状态已经在前面介绍过了。

![lock1](https://github.com/wangtengke/Notes/blob/master/imgs/lock1.png)

下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。

![lock2](https://github.com/wangtengke/Notes/blob/master/imgs/lock2.png)

轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。

![lock3](https://github.com/wangtengke/Notes/blob/master/imgs/lock3.png)

- **偏向锁**

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

![lock4](https://github.com/wangtengke/Notes/blob/master/imgs/lock4.jpg)
## volatile
**定义**

Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。

通俗点讲就是说一个变量如果用volatile修饰了，则Java可以确保所有线程看到这个变量的值是一致的，如果某个线程对volatile修饰的共享变量进行更新，那么其他线程可以立马看到这个更新，这就是所谓的线程可见性。

**原子性**

volatile是无法保证复合操作的原子性。

**可见性**

Java提供了volatile来保证可见性。

当一个变量被volatile修饰后，当一个线程的变量被修改后，其他线程的该变量的本地缓存失效，需要从主内存中读取，因此就保证了变量的变化，所有的线程都能感知到，保证了可见性。

**有序性**

有序性：即程序执行的顺序按照代码的先后顺序执行。

在Java内存模型中，为了效率是允许编译器和处理器对指令进行重排序，当然重排序它不会影响单线程的运行结果，但是对多线程会有影响。

Java提供volatile来保证一定的有序性。最著名的例子就是单例模式里面的DCL（双重检查锁）。

**剖析volatile原理**

volatile可以保证线程可见性且提供了一定的有序性，但是无法保证原子性。在JVM底层volatile是采用“内存屏障”来实现的。

**volataile的内存语义及其实现**

在JMM中，线程之间的通信采用共享内存来实现的。volatile的内存语义是：
- 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新到主内存中。 
- 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，直接从主内存中读取共享变量。

所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取。 那么volatile的内存语义是如何实现的呢？对于一般的变量则会被重排序，而对于volatile则不能，这样会影响其内存语义，所以为了实现volatile的内存语义JMM会限制重排序。其重排序规则如下：

1. 如果第一个操作为volatile读，则不管第二个操作是啥，都不能重排序。这个操作确保volatile读之后的操作不会被编译器重排序到volatile读之前；
2. 当第二个操作为volatile写是，则不管第一个操作是啥，都不能重排序。这个操作确保volatile写之前的操作不会被编译器重排序到volatile写之后；
3. 当第一个操作volatile写，第二操作为volatile读时，不能重排序。

volatile的底层实现是通过插入内存屏障，但是对于编译器来说，发现一个最优布置来最小化插入内存屏障的总数几乎是不可能的，所以，JMM采用了保守策略。如下：
- 在每一个volatile写操作前面插入一个StoreStore屏障
- 在每一个volatile写操作后面插入一个StoreLoad屏障
- 在每一个volatile读操作后面插入一个LoadLoad屏障
- 在每一个volatile读操作后面插入一个LoadStore屏障

StoreStore屏障可以保证在volatile写之前，其前面的所有普通写操作都已经刷新到主内存中。

StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序。

LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。

LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序。

下面我们就上面那个VolatileTest例子分析下：
```java
public class VolatileTest {
    int i = 0;
    volatile boolean flag = false;
    public void write(){
        i = 2;
        flag = true;
    }

    public void read(){
        if(flag){
            System.out.println("---i = " + i); 
        }
    }
}
```
![volatile语义](https://github.com/wangtengke/Notes/blob/master/imgs/volatile%E8%AF%AD%E4%B9%89.jpg)

上面通过一个例子稍微演示了volatile指令的内存屏障图例。

volatile的内存屏障插入策略非常保守，其实在实际中，只要不改变volatile写-读得内存语义，编译器可以根据具体情况优化，省略不必要的屏障。如下（摘自方腾飞 《Java并发编程的艺术》）：
```java
public class VolatileBarrierExample {
    int a = 0;
    volatile int v1 = 1;
    volatile int v2 = 2;
    
    void readAndWrite(){
        int i = v1;     //volatile读
        int j = v2;     //volatile读
        a = i + j;      //普通读
        v1 = i + 1;     //volatile写
        v2 = j * 2;     //volatile写
    }
}
```
没有优化的示例图如下：

![volatile语义1](https://github.com/wangtengke/Notes/blob/master/imgs/volatile%E8%AF%AD%E4%B9%891.jpg)

我们来分析上图有哪些内存屏障指令是多余的
1. 这个肯定要保留了
2. 禁止下面所有的普通写与上面的volatile读重排序，但是由于存在第二个volatile读，那个普通的读根本无法越过第二个volatile读。所以可以省略。
3. 下面已经不存在普通读了，可以省略。
4. 保留
5. 保留
6. 下面跟着一个volatile写，所以可以省略
7. 保留
8. 保留

所以2、3、6可以省略，其示意图如下：

![volatile语义2](https://github.com/wangtengke/Notes/blob/master/imgs/volatile%E8%AF%AD%E4%B9%892.jpg)

**总结**

volatile看起来简单，但是要想理解它还是比较难的，这里只是对其进行基本的了解。volatile相对于synchronized稍微轻量些，在某些场合它可以替代synchronized，但是又不能完全取代synchronized，只有在某些场合才能够使用volatile。使用它必须满足如下两个条件：
1. 对变量的写操作不依赖当前值；
2. 该变量没有包含在具有其他变量的不变式中。

volatile经常用于两个两个场景：状态标记量、double check

# Java内存模型
## Java内存模型介绍
Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

**主内存与工作内存**
处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

![memory1](https://github.com/wangtengke/Notes/blob/master/imgs/memory1.png)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![memory2](https://github.com/wangtengke/Notes/blob/master/imgs/memory2.png)

**内存间交互操作**

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![memory3](https://github.com/wangtengke/Notes/blob/master/imgs/memory3.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock：作用与主内存的变量

## happens-before
在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。

happens-before原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要依据，依靠这个原则，我们解决在并发环境下两操作之间是否可能存在冲突的所有问题。下面我们就一个简单的例子稍微了解下happens-before：
```java
i = 1;       //线程A执行
j = i ;      //线程B执行
```
j 是否等于1呢？假定线程A的操作（i = 1）happens-before线程B的操作（j = i）,那么可以确定线程B执行后j = 1 一定成立，如果他们不存在 happens-before 原则，那么 j = 1 不一定成立。这就是 happens-before 原则的威力。

happens-before原则定义如下：

1. **如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。**

2. **两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。**

下面是happens-before原则规则：

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作；

2. 锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作；

3. volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作；

4. 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；

5. 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作；

6. 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；

7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；

8. 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始；

我们来详细看看上面每条规则（摘自《深入理解Java虚拟机第12章》）：

- 程序次序规则：一段代码在单线程中执行的结果是有序的。注意是执行结果，因为虚拟机、处理器会对指令进行重排序（重排序后面会详细介绍）。虽然重排序了，但是并不会影响程序的执行结果，所以程序最终执行的结果与顺序执行的结果是一致的。故而这个规则只对单线程有效，在多线程环境下无法保证正确性。

- 锁定规则：这个规则比较好理解，无论是在单线程环境还是多线程环境，一个锁处于被锁定状态，那么必须先执行unlock操作后面才能进行lock操作。

- volatile变量规则：这是一条比较重要的规则，它标志着volatile保证了线程可见性。通俗点讲就是如果一个线程先去写一个volatile变量，然后一个线程去读这个变量，那么这个写操作一定是happens-before读操作的。

- 传递规则：提现了happens-before原则具有传递性，即A happens-before B , B happens-before C，那么A happens-before C

- 线程启动规则：假定线程A在执行过程中，通过执行ThreadB.start()来启动线程B，那么线程A对共享变量的修改在接下来线程B开始执行后确保对线程B可见。

- 线程终结规则：假定线程A在执行的过程中，通过制定ThreadB.join()等待线程B终止，那么线程B在终止之前对共享变量的修改在线程A等待返回后可见。

上面八条是原生Java满足Happens-before关系的规则，但是我们可以对他们进行推导出其他满足happens-before的规则：

1. 将一个元素放入一个线程安全的队列的操作Happens-Before从队列中取出这个元素的操作

2. 将一个元素放入一个线程安全容器的操作Happens-Before从容器中取出这个元素的操作

3. 在CountDownLatch上的倒数操作Happens-Before CountDownLatch#await()操作
4. 释放Semaphore许可的操作Happens-Before获得许可操作
5. Future表示的任务的所有操作Happens-Before Future#get()操作
6. 向Executor提交一个Runnable或Callable的操作Happens-Before任务开始执行操作

这里再说一遍happens-before的概念：**如果两个操作不存在上述（前面8条 + 后面6条）任一一个happens-before规则，那么这两个操作就没有顺序的保障，JVM可以对这两个操作进行重排序。如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的。**

下面就用一个简单的例子来描述下happens-before原则：
```java
private int i = 0;

public void write(int j ){
	i = j;
}

public int read(){
	return i;
}
```
我们约定线程A执行write()，线程B执行read()，且线程A优先于线程B执行，那么线程B获得结果是什么？；我们就这段简单的代码一次分析happens-before的规则（规则5、6、7、8 + 推导的6条可以忽略，因为他们和这段代码毫无关系）：

1. 由于两个方法是由不同的线程调用，所以肯定不满足程序次序规则；
2. 两个方法都没有使用锁，所以不满足锁定规则；
3. 变量i不是用volatile修饰的，所以volatile变量规则不满足；
4. 传递规则肯定不满足；

所以我们无法通过happens-before原则推导出线程A happens-before线程B，虽然可以确认在时间上线程A优先于线程B指定，但是就是无法确认线程B获得的结果是什么，所以这段代码不是线程安全的。那么怎么修复这段代码呢？满足规则2、3任一即可。

**happen-before原则是JMM中非常重要的原则，它是判断数据是否存在竞争、线程是否安全的主要依据，保证了多线程环境下的可见性。**

下图是happens-before与JMM的关系图（摘自《Java并发编程的艺术》）

![happens-before](https://github.com/wangtengke/Notes/blob/master/imgs/jmm-happens-before.png)
## 重排序
在执行程序时，为了提供性能，处理器和编译器常常会对指令进行重排序，但是不能随意重排序，不是你想怎么排序就怎么排序，它需要满足以下两个条件：

1. 在单线程环境下不能改变程序运行的结果；
2. 存在数据依赖关系的不允许重排序

其实这两点可以归结于一点：无法通过happens-before原则推导出来的，JMM允许任意的排序。

**as-if-serial语义**

as-if-serial语义的意思是，所有的操作均可以为了优化而被重排序，但是你必须要保证重排序后执行的结果不能被改变，编译器、runtime、处理器都必须遵守as-if-serial语义。注意as-if-serial只保证单线程环境，多线程环境下无效。

下面我们用一个简单的示例来说明：
```java
int a = 1 ;      //A
int b = 2 ;      //B
int c = a + b;   //C
```
A、B、C三个操作存在如下关系：A、B不存在数据依赖关系，A和C、B和C存在数据依赖关系，因此在进行重排序的时候，A、B可以随意排序，但是必须位于C的前面，执行顺序可以是A –> B –> C或者B –> A –> C。但是无论是何种执行顺序最终的结果C总是等于3。

as-if-serail语义把单线程程序保护起来了，它可以保证在重排序的前提下程序的最终结果始终都是一致的。

其实对于上段代码，他们存在这样的happen-before关系：
1. A happens-before B
2. B happens-before C
3. A happens-before C

1、2是程序顺序次序规则，3是传递性。但是，不是说通过重排序，B可能会排在A之前执行么，为何还会存在存在A happens-beforeB呢？这里再次申明A happens-before B不是A一定会在B之前执行，而是A的对B可见，但是相对于这个程序A的执行结果不需要对B可见，且他们重排序后不会影响结果，所以JMM不会认为这种重排序非法。

我们需要明白这点：在不改变程序执行结果的前提下，尽可能提高程序的运行效率。

下面我们在看一段有意思的代码：
```java
public class RecordExample1 {
    public static void main(String[] args){
        int a = 1;
        int b = 2;

        try {
            a = 3;           //A
            b = 1 / 0;       //B
        } catch (Exception e) {
            
        } finally {
            System.out.println("a = " + a);
        }
    }
}
```
按照重排序的规则，操作A与操作B有可能会进行重排序，如果重排序了，B会抛出异常（ / by zero），此时A语句一定会执行不到，那么a还会等于3么？如果按照as-if-serial原则它就改变了程序的结果。其实JVM对异常做了一种特殊的处理，为了保证as-if-serial语义，Java异常处理机制对重排序做了一种特殊的处理：JIT在重排序时会在catch语句中插入错误代偿代码（a = 3）,这样做虽然会导致cathc里面的逻辑变得复杂，但是JIT优化原则是：尽可能地优化程序正常运行下的逻辑，哪怕以catch块逻辑变得复杂为代价。

**重排序对多线程的影响**

在单线程环境下由于as-if-serial语义，重排序无法影响最终的结果，但是对于多线程环境呢？

如下代码（volatile的经典用法）：
```java
public class RecordExample2 {
    int a = 0;
    boolean flag = false;

    /**
     * A线程执行
     */
    public void writer(){
        a = 1;                  // 1
        flag = true;            // 2
    }

    /**
     * B线程执行
     */
    public void read(){
        if(flag){                  // 3
           int i = a + a;          // 4
        }
    }

}
```
A线程执行writer()，线程B执行read()，线程B在执行时能否读到 a = 1 呢？答案是不一定（注：X86CPU不支持写写重排序，如果是在x86上面操作，这个一定会是a=1）。

由于操作1 和操作2 之间没有数据依赖性，所以可以进行重排序处理，操作3 和操作4 之间也没有数据依赖性，他们亦可以进行重排序，但是操作3 和操作4 之间存在控制依赖性。假如操作1 和操作2 之间重排序：

![重排序](https://github.com/wangtengke/Notes/blob/master/imgs/%E9%87%8D%E6%8E%92%E5%BA%8F.jpg)

按照这种执行顺序线程B肯定读不到线程A设置的a值，在这里多线程的语义就已经被重排序破坏了。

操作3 和操作4 之间也可以重排序，这里就不阐述了。但是他们之间存在一个控制依赖的关系，因为只有操作3 成立操作4 才会执行。当代码中存在控制依赖性时，会影响指令序列的执行的并行度，所以编译器和处理器会采用猜测执行来克服控制依赖对并行度的影响。假如操作3 和操作4重排序了，操作4 先执行，则先会把计算结果临时保存到重排序缓冲中，当操作3 为真时才会将计算结果写入变量i中

通过上面的分析，**重排序不会影响单线程环境的执行结果，但是会破坏多线程的执行语义**。

## 从JMM角度分析DCL
在早期的 JVM 中，synchronized（甚至是无竞争的 synchronized）存在这巨大的性能开销。因此，人们想出了一个“聪明”的技巧：双重检查锁定（double-checked locking）。人们想通过双重检查锁定来降低同步的开销。下面是使用双重检查锁定来实现延迟初始化的示例代码：
```java
ublic class DoubleCheckedLocking {                 //1
    private static Instance instance;                    //2

    public static Instance getInstance() {               //3
        if (instance == null) {                          //4: 第一次检查 
            synchronized (DoubleCheckedLocking.class) {  //5: 加锁 
                if (instance == null)                    //6: 第二次检查 
                    instance = new Instance();           //7: 问题的根源出在这里 
            }                                            //8
        }                                                //9
        return instance;                                 //10
    }                                                    //11
}                                                        //12
```
如上面代码所示，如果第一次检查 instance 不为 null，那么就不需要执行下面的加锁和初始化操作。因此可以大幅降低 synchronized 带来的性能开销。上面代码表面上看起来，似乎两全其美：

- 在多个线程试图在同一时间创建对象时，会通过加锁来保证只有一个线程能创建对象。
- 在对象创建好之后，执行 getInstance() 将不需要获取锁，直接返回已创建好的对象。

双重检查锁定看起来似乎很完美，但这是一个错误的优化！在线程执行到第 4 行代码读取到 instance 不为 null 时，instance 引用的对象有可能还没有完成初始化。

**问题的根源**

前面的双重检查锁定示例代码的第 7 行（instance = new Singleton();）创建一个对象。这一行代码可以分解为如下的三行伪代码：
```java
memory = allocate();   //1：分配对象的内存空间 
ctorInstance(memory);  //2：初始化对象 
instance = memory;     //3：设置 instance 指向刚分配的内存地址
```
上面三行伪代码中的 2 和 3 之间，可能会被重排序（在一些 JIT 编译器上，这种重排序是真实发生的，2 和 3 之间重排序之后的执行时序如下：
```java
memory = allocate();   //1：分配对象的内存空间 
instance = memory;     //3：设置 instance 指向刚分配的内存地址 
                       // 注意，此时对象还没有被初始化！
ctorInstance(memory);  //2：初始化对象
```
请看下面的示意图（多线程下 2 和 3 重排的情况）：

![muliThread重排序](https://github.com/wangtengke/Notes/blob/master/imgs/muliThread%E9%87%8D%E6%8E%92%E5%BA%8F.png)

DoubleCheckedLocking 示例代码的第 7 行（instance = new Singleton();）如果发生重排序，另一个并发执行的线程 B 就有可能在第 4 行判断 instance 不为 null。线程 B 接下来将访问 instance 所引用的对象，但此时这个对象可能还没有被 A 线程初始化！

**基于 volatile 的双重检查锁定的解决方案**

对于前面的基于双重检查锁定来实现延迟初始化的方案（指 DoubleCheckedLocking 示例代码），我们只需要做一点小的修改（把 instance 声明为 volatile 型），就可以实现线程安全的延迟初始化。请看下面的示例代码：
```java
public class SafeDoubleCheckedLocking {
    private volatile static Instance instance;

    public static Instance getInstance() {
        if (instance == null) {
            synchronized (SafeDoubleCheckedLocking.class) {
                if (instance == null)
                    instance = new Instance();//instance 为 volatile，现在没问题了 
            }
        }
        return instance;
    }
}
```
当声明对象的引用为 volatile 后，“问题的根源”的三行伪代码中的 2 和 3 之间的重排序，在多线程环境中将会被禁止。上面示例代码将按如下的时序执行：

![volatile重排序](https://github.com/wangtengke/Notes/blob/master/imgs/volatile%E9%87%8D%E6%8E%92%E5%BA%8F.png)
## 总结
JMM规定了线程的工作内存和主内存的交互关系，以及线程之间的可见性和程序的执行顺序。一方面，要为程序员提供足够强的内存可见性保证；另一方面，对编译器和处理器的限制要尽可能地放松。JMM对程序员屏蔽了CPU以及OS内存的使用问题，能够使程序在不同的CPU和OS内存上都能够达到预期的效果。

Java采用内存共享的模式来实现线程之间的通信。编译器和处理器可以对程序进行重排序优化处理，但是需要遵守一些规则，不能随意重排序。

- 原子性：一个操作或者多个操作要么全部执行要么全部不执行；

- 可见性：当多个线程同时访问一个共享变量时，如果其中某个线程更改了该共享变量，其他线程应该可以立刻看到这个改变；

- 有序性：程序的执行要按照代码的先后顺序执行；

在并发编程模式中，势必会遇到上面三个概念，JMM对原子性并没有提供确切的解决方案，但是JMM解决了可见性和有序性，至于原子性则需要通过锁或者Synchronized来解决了。

如果一个操作A的操作结果需要对操作B可见，那么我们就认为操作A和操作B之间存在happens-before关系，即A happens-before B。

happens-before原则是JMM中非常重要的一个原则，它是判断数据是否存在竞争、线程是否安全的主要依据，依靠这个原则，我们可以解决在并发环境下两个操作之间是否存在冲突的所有问题。JMM规定，两个操作存在happens-before关系并不一定要A操作先于B操作执行，只要A操作的结果对B操作可见即可。

在程序运行过程中，为了执行的效率，编译器和处理器是可以对程序进行一定的重排序，但是他们必须要满足两个条件：1 执行的结果保持不变，2 存在数据依赖的不能重排序。重排序是引起多线程不安全的一个重要因素。

同时顺序一致性是一个比较理想化的参考模型，它为我们提供了强大而又有力的内存可见性保证，他主要有两个特征：1 一个线程中的所有操作必须按照程序的顺序来执行；2 所有线程都只能看到一个单一的操作执行顺序，在顺序一致性模型中，每个操作都必须原则执行且立刻对所有线程可见。
# J.U.C
## AQS
## 重入锁与读写锁
### 重入锁 ReentrantLock
ReentrantLock，可重入锁，是一种递归无阻塞的同步机制。它可以等同于synchronized的使用，但是ReentrantLock提供了比synchronized更强大、灵活的锁机制，可以减少死锁发生的概率。
API介绍如下：

一个可重入的互斥锁定 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 isHeldByCurrentThread() 和 getHoldCount() 方法来检查此情况是否发生。

ReentrantLock还提供了公平锁也非公平锁的选择，构造方法接受一个可选的公平参数（默认非公平锁），当设置为true时，表示公平锁，否则为非公平锁。公平锁与非公平锁的区别在于公平锁的锁获取是有顺序的。但是公平锁的效率往往没有非公平锁的效率高，在许多线程访问的情况下，公平锁表现出较低的吞吐量。

**ReentrantLock与synchronized的区别**
1. 与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。
2. ReentrantLock还提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更加适合（以后会阐述Condition）。
3. ReentrantLock提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized则一旦进入锁请求要么成功要么阻塞，所以相比synchronized而言，ReentrantLock会不容易产生死锁些。
4. ReentrantLock支持更加灵活的同步代码块，但是使用synchronized时，只能在同一个synchronized块结构中获取和释放。注：ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。
5. ReentrantLock支持中断处理，且性能较synchronized会好些。


### 读写锁 ReentrantReadWriteLock
重入锁ReentrantLock是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

读写锁的主要特性：

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁

**写锁**：写锁就是一个支持可重入的排他锁。

**读锁**：读锁为一个可重入的共享锁，它能够被多个线程同时持有，在没有其他写线程访问时，读锁总是或获取成功。

**锁降级**：锁降级就意味着写锁是可以降级为读锁的，但是需要遵循先获取写锁、获取读锁在释放写锁的次序。注意如果当前线程先获取写锁，然后释放写锁，再获取读锁这个过程不能称之为锁降级，锁降级一定要遵循那个次序。
## 并发工具类
### CountdownLatch
用来控制一个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

![CountdownLatch](https://github.com/wangtengke/Notes/blob/master/imgs/CountdownLatch.png)

示例仍然使用开会案例。老板进入会议室等待5个人全部到达会议室才会开会。所以这里有两个线程老板等待开会线程、员工到达会议室：
```java
public class CountDownLatchTest {
    private static CountDownLatch countDownLatch = new CountDownLatch(5);

    /**
     * Boss线程，等待员工到达开会
     */
    static class BossThread extends Thread{
        @Override
        public void run() {
            System.out.println("Boss在会议室等待，总共有" + countDownLatch.getCount() + "个人开会...");
            try {
                //Boss等待
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("所有人都已经到齐了，开会吧...");
        }
    }

    //员工到达会议室
    static class EmpleoyeeThread  extends Thread{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "，到达会议室....");
            //员工到达会议室 count - 1
            countDownLatch.countDown();
        }
    }

    public static void main(String[] args){
        //Boss线程启动
        new BossThread().start();

        for(int i = 0 ; i < countDownLatch.getCount() ; i++){
            new EmpleoyeeThread().start();
        }
    }
}
```
运行结果：
```
Boss在会议室等待，总共有5个人开会...
Thread-3，到达会议室....
Thread-1，到达会议室....
Thread-2，到达会议室....
Thread-5，到达会议室....
Thread-4，到达会议室....
所有人都已经到齐了，开会吧...
```
### CyclicBarrier
用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。
```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

![CyclicBarrier](https://github.com/wangtengke/Notes/blob/master/imgs/CyclicBarrier.png)

例子如下：

```java
public class CyclicBarrierTest  {
        private static CyclicBarrier cyclicBarrier;

        static class CyclicBarrierThread extends Thread{
            public void run() {
                System.out.println(Thread.currentThread().getName() + "到了");
                //等待
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

        public static void main(String[] args){
            cyclicBarrier = new CyclicBarrier(5, ()->{
                    System.out.println("人都齐了，开会吧....");
                }
            );

            for(int i = 0 ; i < 5 ; i++){
                new CyclicBarrierThread().start();
            }
        }
}
```
运行结果：
```
Thread-0到了
Thread-2到了
Thread-3到了
Thread-1到了
Thread-4到了
人都齐了，开会吧....
```
### Semaphore

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

![Semaphore](https://github.com/wangtengke/Notes/blob/master/imgs/Semaphore.png)

我们已停车为示例：
```java
public class SemaphoreTest {

    static class Parking{
        //信号量
        private Semaphore semaphore;

        Parking(int count){
            semaphore = new Semaphore(count);
        }

        public void park(){
            try {
                //获取信号量
                semaphore.acquire();
                long time = (long) (Math.random() * 10);
                System.out.println(Thread.currentThread().getName() + "进入停车场，停车" + time + "秒..." );
                Thread.sleep(time);
                System.out.println(Thread.currentThread().getName() + "开出停车场...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
        }
    }


    static class Car extends Thread {
        Parking parking ;

        Car(Parking parking){
            this.parking = parking;
        }

        @Override
        public void run() {
            parking.park();     //进入停车场
        }
    }

    public static void main(String[] args){
        Parking parking = new Parking(3);

        for(int i = 0 ; i < 5 ; i++){
            new Car(parking).start();
        }
    }
}
```
运行结果：
```
Thread-2进入停车场，停车3秒...
Thread-0进入停车场，停车6秒...
Thread-1进入停车场，停车2秒...
Thread-1开出停车场...
Thread-3进入停车场，停车7秒...
Thread-2开出停车场...
Thread-4进入停车场，停车3秒...
Thread-0开出停车场...
Thread-4开出停车场...
Thread-3开出停车场...
```

## 非阻塞同步
互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

### 1. CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。
```java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```
以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。
```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```
以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```
### 3. ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。  

# 线程池
## 线程池原理
线程池好处：
- 降低资源消耗
- 提高响应速度
- 提高线程的可管理性

线程池的实现原理：
1. 判断线程池（corePoolSize）里的核心线程是否都在执行任务，如果不是（核心线程空闲或者还有核心线程没有被创建）则创建一个新的工作线程来执行任务。如果核心线程都在执行任务，则进入下个流程。

2. 线程池判断工作队列（BlockingQueue）是否已满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。

3. 判断线程池（maximumPoolSize）里的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略（调用handle的rejectedExecution()方法）来处理这个任务。

![线程池原理](https://github.com/wangtengke/Notes/blob/master/imgs/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%8E%9F%E7%90%86.jpg)

线程池构造方法
```java
ThreadPoolExecutor(int corePoolSize,

int maximumPoolSize,

long keepAliveTime,

TimeUnit unit,

BlockingQueue<Runnable> workQueue,

ThreadFactory threadFactory,

RejectedExecutionHandler handler)
```
- **corePoolSize**  线程池核心线程数，当提交任务时发现线程数小于这个数，将直接创建新线程执行任务，即使有空余线程也会创建新的，如果调用了线程池的prestartAllCoreThreads()将提前创建好指定个数的线程（执行这一步需要获得全局锁）
- **maximumPoolSize** 线程池最大线程数
- **keepAliveTime** 线程空闲后，存活时间
- **unit** 时间单位
- **workQueue** 任务队列，如果传入的是无界队列，那maximumPoolSize无效
- **threadFactory** 用于设置创建线程的工厂，可以用来给线程设置有意义的名字，如：```new ThreadFactoryBuilder().setNameFormat("xx-task-%d").build();```
- **handler** 当线程和队列都满了后的处理策略，jdk的实现类：
   - AbortPolicy：直接抛出异常
   - CallerRunsPolicy：只用调用者所在线程来运行任务
   - DiscardOldestPolicy：丢弃队列最近的一个任务，并执行当前任务
   - Discardpolicy：不处理，丢弃掉

线程池执行Runnable有两个方法：execute(Runnable task)和submit(Runnable task)两个方法，区别是submit会返回一个Feature对象，Feature对象可以调用get()获取返回值，会阻塞当前线程直到任务完成，同时提供了get(long timeout, TimeUnit unit)超时抛出TimeOutException脱离阻塞。
## java Executors类四种线程池
