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
- [Condition](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#condition)
- [并发工具类](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#并发工具类)
- [CAS](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md#cas)  
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
## Condition
## 并发工具类
## CAS  
