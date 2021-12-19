[TOC]

# Java

## 解决哈希冲突的常用方法分析

## 1.基本概念

**哈希算法**：根据设定的哈希函数H（key）和处理冲突方法将一组关键字映象到一个有限的地址区间上的算法。也称为散列算法、杂凑算法。 

**哈希表**：数据经过哈希算法之后得到的集合。这样关键字和数据在集合中的位置存在一定的关系，可以根据这种关系快速查询。 非哈希表：与哈希表相对应，集合中的 数据和其存放位置没任何关联关系的集合。

由此可见，哈希算法是一种特殊的算法，能将任意数据散列后映射到有限的空间上，通常计算机软件中用作快速查找或加密使用。

**哈希冲突**：由于哈希算法被计算的数据是无限的，而计算后的结果范围有限，因此总会存在不同的数据经过计算后得到的值相同，这就是哈希冲突。

## 2.解决哈希冲突的方法

解决哈希冲突的方法一般有：开放定址法、链地址法（拉链法）、再哈希法、建立公共溢出区等方法。

### 2.1 开放定址法

从发生冲突的那个单元起，按照一定的次序，从哈希表中找到一个空闲的单元。然后把发生冲突的元素存入到该单元的一种方法。开放定址法需要的表长度要大于等于所需要存放的元素。 在开放定址法中解决冲突的方法有：线行探查法、平方探查法、双散列函数探查法。 开放定址法的缺点在于删除元素的时候不能真的删除，否则会引起查找错误，只能做一个特殊标记。只到有下个元素插入才能真正删除该元素。

#### 2.1.1 线行探查法

线行探查法是开放定址法中最简单的冲突处理方法，它从发生冲突的单元起，依次判断下一个单元是否为空，当达到最后一个单元时，再从表首依次判断。直到碰到空闲的单元或者探查完全部单元为止。

#### 2.1.2 平方探查法

平方探查法即是发生冲突时，用发生冲突的单元d[i], 加上 1²、 2²等。即d[i] + 1²，d[i] + 2², d[i] + 3²…直到找到空闲单元。 在实际操作中，平方探查法不能探查到全部剩余的单元。不过在实际应用中，能探查到一半单元也就可以了。若探查到一半单元仍找不到一个空闲单元，表明此散列表太满，应该重新建立。

#### 2.1.3 双散列函数探查法

这种方法使用两个散列函数hl和h2。其中hl和前面的h一样，以关键字为自变量，产生一个0至m—l之间的数作为散列地址；h2也以关键字为自变量，产生一个l至m—1之间的、并和m互素的数(即m不能被该数整除)作为探查序列的地址增量(即步长)，探查序列的步长值是固定值l；对于平方探查法，探查序列的步长值是探查次数i的两倍减l；对于双散列函数探查法，其探查序列的步长值是同一关键字的另一散列函数的值。

### 2.2 链地址法（拉链法）

链接地址法的思路是将哈希值相同的元素构成一个同义词的单链表，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。 如下一组数字,(32、40、36、53、16、46、71、27、42、24、49、64)哈希表长度为13，哈希函数为H(key)=key%13,则链表法结果如下：

```javascript
0       
1  -> 40 -> 27 -> 53 
2
3  -> 16 -> 42
4
5
6  -> 32 -> 71
7  -> 46
8
9
10 -> 36 -> 49
11 -> 24
12 -> 64
```

注：在java中，链接地址法也是HashMap解决哈希冲突的方法之一，jdk1.7完全采用单链表来存储同义词，jdk1.8则采用了一种混合模式，对于链表长度大于8的，会转换为红黑树存储。

### 2.3 再哈希法

就是同时构造多个不同的哈希函数： Hi = RHi(key)   i= 1,2,3 … k; 当H1 = RH1(key)  发生冲突时，再用H2 = RH2(key) 进行计算，直到冲突不再产生，这种方法不易产生聚集，但是增加了计算时间。

### 2.4 建立公共溢出区

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

## JUC

在 `java.util.concurrent` 包（JUC）中，有各式各样的并发控制工具。

## synchronized原理

synchronized是java提供的原子性内置锁这种内置的并且使用者看不到的锁也被称为**监视器锁**，使用synchronized之后，会在编译之后在同步的代码块前后加上monitorenter和monitorexit字节码指令，他依赖操作系统底层互斥锁实现。他的作用主要就是实现原子性操作和解决共享变量的内存可见性问题。

执行monitorenter指令时会尝试获取对象锁，如果对象没有被锁定或者已经获得了锁，锁的计数器+1。此时其他竞争锁的线程则会进入等待队列中。

执行monitorexit指令时则会把计数器-1，当计数器值为0时，则锁释放，处于等待队列中的线程再继续竞争锁。

synchronized是排它锁，当一个线程获得锁之后，其他线程必须等待该线程释放锁后才能获得锁，而且由于Java中的线程和操作系统原生线程是一一对应的，线程被阻塞或者唤醒时时会从用户态切换到内核态，这种转换非常消耗性能。

从内存语义来说，加锁的过程会清除工作内存中的共享变量，再从主内存读取，而释放锁的过程则是将工作内存中的共享变量写回主内存。

*实际上大部分时候我认为说到monitorenter就行了，但是为了更清楚的描述，还是再具体一点*。

如果再深入到源码来说，synchronized实际上有两个队列waitSet和entryList。

1. 当多个线程进入同步代码块时，首先进入entryList
2. 有一个线程获取到monitor锁后，就赋值给当前线程，并且计数器+1
3. 如果线程调用wait方法，将释放锁，当前线程置为null，计数器-1，同时进入waitSet等待被唤醒，调用notify或者notifyAll之后又会进入entryList竞争锁
4. 如果线程执行完毕，同样释放锁，计数器-1，当前线程置为null

### synchronized优化机制

从JDK1.6版本之后，synchronized本身也在不断优化锁的机制，有些情况下他并不会是一个很重量级的锁了。优化机制包括自适应锁、自旋锁、锁消除、锁粗化、轻量级锁和偏向锁。

锁的状态从低到高依次为**无锁->偏向锁->轻量级锁->重量级锁**，升级的过程就是从低到高，降级在一定条件也是有可能发生的。

#### 自旋锁

由于大部分时候，锁被占用的时间很短，共享变量的锁定时间也很短，所有没有必要挂起线程，用户态和内核态的来回上下文切换严重影响性能。自旋的概念就是让线程执行一个空循环，可以理解为就是啥也不干，防止从用户态转入内核态，自旋锁可以通过设置-XX:+UseSpining来开启，自旋的默认次数是10次，可以使用-XX:PreBlockSpin设置。

#### 自适应锁

自适应锁就是自适应的自旋锁，自旋的时间不是固定时间，而是由前一次在同一个锁上的自旋时间和锁的持有者状态来决定。

#### **锁消除**

锁消除指的是JVM检测到一些同步的代码块，完全不存在数据竞争的场景，也就是不需要加锁，就会进行锁消除。

#### **锁粗化**

锁粗化指的是有很多操作都是对同一个对象进行加锁，就会把锁的同步范围扩展到整个操作序列之外。

#### **偏向锁**

当线程访问同步块获取锁时，会在对象头和栈帧中的锁记录里存储偏向锁的线程ID，之后这个线程再次进入同步块时都不需要CAS来加锁和解锁了，偏向锁会永远偏向第一个获得锁的线程，如果后续没有其他线程获得过这个锁，持有锁的线程就永远不需要进行同步，反之，当有其他线程竞争偏向锁时，持有偏向锁的线程就会释放偏向锁。可以用过设置-XX:+UseBiasedLocking开启偏向锁。

#### **轻量级锁**

JVM的对象的对象头中包含有一些锁的标志位，代码进入同步块的时候，JVM将会使用CAS方式来尝试获取锁，如果更新成功则会把对象头中的状态位标记为轻量级锁，如果更新失败，当前线程就尝试自旋来获得锁。

整个锁升级的过程非常复杂，我尽力去除一些无用的环节，简单来描述整个升级的机制。

简单点说，偏向锁就是通过对象头的偏向线程ID来对比，甚至都不需要CAS了，而轻量级锁主要就是通过CAS修改对象头锁记录和自旋来实现，重量级锁则是除了拥有锁的线程其他全部阻塞。

## 对象头具体都包含哪些内容

相比于synchronized，ReentrantLock需要显式的获取锁和释放锁，相对现在基本都是用JDK7和JDK8的版本，ReentrantLock的效率和synchronized区别基本可以持平了。他们的主要区别有以下几点：

1. 等待可中断，当持有锁的线程长时间不释放锁的时候，等待中的线程可以选择放弃等待，转而处理其他的任务。
2. 公平锁：synchronized和ReentrantLock默认都是非公平锁，但是ReentrantLock可以通过构造函数传参改变。只不过使用公平锁的话会导致性能急剧下降。
3. 绑定多个条件：ReentrantLock可以同时绑定多个Condition条件对象。

ReentrantLock基于AQS(**AbstractQueuedSynchronizer 抽象队列同步器**)实现。

AQS原理：AQS内部维护一个state状态位，尝试加锁的时候通过CAS(CompareAndSwap)修改值，如果成功设置为1，并且把当前线程ID赋值，则代表加锁成功，一旦获取到锁，其他的线程将会被阻塞进入阻塞队列自旋，获得锁的线程释放锁的时候将会唤醒阻塞队列中的线程，释放锁的时候则会把state重新置为0，同时当前线程ID置为空。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ibBMVuDfkZUljPgPC9h7FmEyOSbttvPehjaoTsjWvmlr4VwFnX8ZHGh8xUPt87pI4iaYBOoltaT7zWibDqrO1HouA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## CAS的原理

CAS叫做CompareAndSwap，比较并交换，主要是通过处理器的指令来保证操作的原子性，它包含三个操作数：

1. 变量内存地址，V表示
2. 旧的预期值，A表示
3. 准备设置的新值，B表示

当执行CAS指令时，只有当V等于A时，才会用B去更新V的值，否则就不会执行更新操作。

### CAS缺点

**ABA问题**：ABA的问题指的是在CAS更新的过程中，当读取到的值是A，然后准备赋值的时候仍然是A，但是实际上有可能A的值被改成了B，然后又被改回了A，这个CAS更新的漏洞就叫做ABA。只是ABA的问题大部分场景下都不影响并发的最终效果。

Java中有AtomicStampedReference来解决这个问题，他加入了预期标志和更新后标志两个字段，更新时不光检查值，还要检查当前的标志是否等于预期标志，全部相等的话才会更新。

**循环时间长开销大**：自旋CAS的方式如果长时间不成功，会给CPU带来很大的开销。

**只能保证一个共享变量的原子操作**：只对一个共享变量操作可以保证原子性，但是多个则不行，多个可以通过AtomicReference来处理或者使用锁synchronized实现。

## 双亲委派模型

### 什么是双亲委派模型？

#### JVM的类加载器

![](E:\workSpace\docx\material_docx\java\img\java-1.png)

JVM自带的类加载器包括启动类加载器、扩展类加载器和应用程序类加载器，此外还可以自定义类加载器。每个类加载器的作用都不一样。

**启动类加载器（Bootstrap ClassLoader）**

负责加载 JAVA_HOME\lib 目录下的类库，也就是JDK核心类库。

**扩展类加载器（Extension ClassLoader）**

负责加载 JAVA_HOME\lib\ext 这个子目录下的类库。

**应用程序类加载器（Application ClassLoader）**

负责加载用户类路径 Classpath 上的类库。

**自定义类加载器**

通过继承 java.lang.ClassLoader，根据不同的需求来实现自定义的类加载器，可以用来加载用户指定目录下的Class。在实际开发中，通过继承JDK中的类或者实现JDK中的接口来实现自定义类加载器。

#### 类加载器工作流程：双亲委派

**双亲委派模型（Parent Delegation Model)**，双亲委派模型包括两个过程，向上委派和向下委派。

![](E:\workSpace\docx\material_docx\java\img\java-2.png)

**向上委派**

一个类在收到类加载请求后，不会自己马上加载这个类，而是把这个类的加载请求委派给它的父类去完成。父类收到这个请求后又会继续向上委派给父类的父类。以此类推，直到所有的请求委派到**Bootstrap ClassLoader**。

**向下委派**

当**Bootstrap ClassLoader**在接收到类加载请求后，如果发现自己无法加载这个类，比如这个类的Class文件不在加载路径中，那么父类就会向下委派子类加载器来加载这个类，直到这个请求被成功加载。

如果一直到最底层的自定义加载器都没有找到这个类的Class文件，那么JVM就会抛出**ClassNotFound**异常。

#### 为什么采用双亲委派模型？

双亲委派模型的主要作用是保证JVM加载类的一致性，尤其是一些JDK中最基础的类。

如果没有使用双亲委派模型，由各个类加载器自行加载的话，如果用户自己编写了一个称为 java.lang.Object 的类，并放在程序的ClassPath中，那系统将会出现多个不同的 Object 类，这样会导致Java类型的混乱。

双亲委派模型可以保证对于 Java.lang.Object 类，无论哪个类加载器要加载它，最终都会委派给Bootstrap ClassLoader，一定会加载 JAVA_HOME/lib 中的 Object 类。

另外双亲委派模型也可避免同一个类被加载多次，防止内存中出现多份同样的字节码。

**面试中经常会问到的一个问题：可以不可以自己写个String类？**

如果我们定义的完全类名是 java.lang.String, 我们知道基于双亲委派模型，会通过父加载器加载，那么最终到 Bootstrap ClassLoader 加载的就是标准JDK中的 String 类。其他加载器看到 String 类已经被加载，也就不会再重新加载 String 类了。

但是如果我们自定义的 String 类，包名不同的话，是可以重写的，在JVM看来，这其实是两个不同的类。

不过需要注意的是，双亲委派模型只是一种建议，而并非强制，根据场景需要是可以被破坏的，比如JDBC、JNDI就破坏了双亲委派模型的类加载顺序。

### 为什么要打破双亲委派模型

#### 双亲委派模型

![](E:\workSpace\docx\material_docx\java\img\java-3.png)

双亲委派模型就是将类加载器进行分层。在触发类加载的时候，当前类加载器会从低层向上委托父类加载器去加载。每层类加载器在加载时会判断该类是否已经被加载过，如果已经加载过，就不再加载了。这种设计能够避免重复加载类、核心类被篡改等情况发生。

双亲委派模型很好地解决了各个类加载器的基础类的统一问题，越基础的类由越上层的加载器进行加载。Java基础类一般都是被用户代码所调用的，那么是否存在基础类需要调用用户代码的情况呢？

主要场景就是JDK自身只提供接口规范，然后让第三方厂商去提供接口的具体实现，这样就会出现基础类调用厂商代码的情况，比如JDBC驱动类的加载，这类接口统称为服务提供者接口（SPI）。

**服务提供者接口（SPI）**

我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块、XML解析模块、JDBC模块等。为了实现模块实现的可插拔，在模块装配的时候就需要一种服务发现机制。Java SPI就是一种为某个接口寻找实现的机制。

Java 提供了很多**服务提供者接口（SPI：Service Provider Interface）**，允许第三方为这些接口提供实现。常见的 SPI 包括 JDBC、JCE、JNDI、JAXP 和 JBI 等。

SPI机制允许Java程序在运行时才来加载具体的接口实现类，使用者只需要按照SPI规范指定具体的接口实现即可。通过这种方式，就实现策略模式和热拔插效果。

这些 SPI 接口在Java 核心库中，而实现代码则是在类路径（ClassPath）下的Jar包中。核心库中涉及到SPI接口的代码需要加载接口实现类。

#### 为何要破坏双亲委派模型？

以JDBC为例，它的代码在rt.jar中，由启动类加载器去加载，但它需要调用厂商实现的SPI代码，这些代码部署在ClassPath下面。

根据双亲委派模型，启动类加载器无法直接委派应用程序类加载器（Application ClassLoader）来加载SPI的实现代码。那么启动类加载器如何找到这些代码呢？

JDK引入了**线程上下文类加载器（TCCL：Thread Context ClassLoader）**，线程上下文类加载器破坏了“双亲委派模型”，可以在执行线程中抛弃双亲委派加载链，利用线程上下文类加载器去加载所需要的SPI代码。

TCCL是从JDK1.2开始引入的，可以通过 java.lang.Thread 类中的 getContextClassLoader()和 setContextClassLoader(ClassLoader cl) 方法来获取和设置线程的上下文类加载器。如果没有手动设置上下文类加载器，线程将继承其父线程的上下文类加载器，初始线程的默认上下文类加载器是 Application ClassLoader。

![](E:\workSpace\docx\material_docx\java\img\java-4.png)

连接Mysql数据库时需要加载Mysql的JDBC驱动 com.mysql.jdbc.Driver。

DriverManager调用 getConnection() 连接数据库时，会触发 ServiceLoader.load(Driver.class)。

load() 函数会通过 Thread.currentThread().getContextClassLoader(）获得当前线程的上下文类加载器，完成驱动类加载。

![](E:\workSpace\docx\material_docx\java\img\java-5.png)

接下来在Classpath的jar包中查找，如存在META-INF/services/java.sql.Driver 文件，则加载其实现类，比如mysql-connector-java-5.1.44.jar。从上图的调用链中，可以知道此时使用的类加载器是线程上下文类加载器，这就是SPI机制的核心原理。

Spring框架也用到了**线程上下文类加载器（TCCL）**，来加载WEB-INF下的用户代码，同样也是打破了双亲委派模型的加载链。

## 并发的三大特性

### 可见性

当一个线程修改了共享变量的值，其他线程能够看到修改的值。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方法来实现可见性的。

如线程 A 修改一个普通变量的值，然后向主内存进行回写，另外一条线程 B 在线程 A 回写完成了之后再从主内存进行读取操作，新变量的值才会对线程 B 可见。

#### 如何保证可见性

- 通过 **volatile 关键字**标记内存屏障保证可见性。
- 通过 **synchronized 关键字**定义同步代码块或者同步方法保障可见性。
- 通过 **Lock 接口**保障可见性。
- 通过 **Atomic 类型**保障可见性。
- 通过 **final 关键字**保障可见性。

### 原子性

一个或多个操作，要么全部执行且在执行过程中不被任何因素打断，要么全部不执行。在 Java 中，对**基本数据类型的变量**的**读取**和**赋值**操作是原子性操作。

#### 如何保证原子性

- 通过 **synchronized 关键字**定义同步代码块或者同步方法保障原子性。
- 通过 **Lock 接口**保障原子性。
- 通过 **Atomic 类型**保障原子性。

### 有序性

即程序执行的顺序按照代码的先后顺序执行，JVM 存在**指令重排**，所以存在有序性问题。

#### 如何保证有序性

- 通过 **synchronized关键字** 定义同步代码块或者同步方法保障可见性。
- 通过 **Lock接口** 保障可见性。

## Volatile原理

### 概述

要了解并发编程，首先就需要了解并发编程的三大特性：**可见性、原子性和有序性。**volatile保证了**可见性和有序性**，但是**不保证原子性**。

### volatile保证可见性

![](E:\workSpace\docx\material_docx\java\img\20211219171704.png)

**volatile通过cpu的总线嗅探机制，将其他也正在使用该变量的线程的数据失效掉，使得这些线程要重新读取主内存中的值**，最后线程A就发现flag的值被改变了。

### Volatile保证有序性

Volatile通过内存屏障禁止指令的重排序，从而保证执行的有序性。

### Volatile不保证原子性

Volatile不保证原子性

### Volatile的使用场景

双重校验单例：

```
public class Singleton {
    private static Singleton Instance;
    private Singleton(){};
    public static Singleton getInstance(){
        if (Instance==null){
            synchronized (Singleton.class){
                Instance=new Singleton();
            }
        }
        return Instance;
    }
}
```

单例模式最经典的一段代码，获取实例对象时如果为空就初始化，如果不为空就返回实例，看着没有问题，但是在高并发环境下这段代码是会出问题的。
Instance=new Singleton(); 实例化一个对象时，需要经历三步：

![](E:\workSpace\docx\material_docx\java\img\20201203215824482.png)

**这三步是有可能发生指令重排序的**，因此有可能是先申请内存空间，再把对象赋值到内存里，最后实例化对象。**第一步->第三步->第二步**的方式来执行。

当此时有两个线程A和B同时申请对象的时候，当线程A执行到重排序后的第二步时

![](E:\workSpace\docx\material_docx\java\img\2020120321584594.png)

线程B执行了if (Instance==null)这行代码，因为此时instance已经赋值到内存里了，所以会直接return Instance; 但是！这个对象并没有被实例化，因此线程B调用该实例时，就报错了。

这个问题的解决办法就是volatile禁止重排序

```
private static volatile Singleton Instance; 
```

### Volatile可能会导致的问题

如果一个程序中用了大量的volatile，就有可能会导致总线风暴，所谓总线风暴，就是指当volatile修饰的变量发生改变时，总线嗅探机制机会将其他内存中的这个变量失效掉，如果volatile很多，无效交互会导致总线带宽达到峰值。因此对volatile的使用也需要适度。

## AQS简介

AQS（AbstractQueuedSynchronizer）为抽象队列同步器，简单的说AQS就是一个抽象类，抽象类AbstractQueuedSynchronizer，没有实现任何的接口，仅仅定义了同步状态（state）的获取和释放的方法。它提供了一个FIFO队列，多线程竞争资源的时候，没有竞争到的线程就会进入队列中进行等待，并且定义了一套多线程访问共享资源的同步框架。

在AQS中的锁类型有两种：分别是Exclusive(独占锁)和Share(共享锁)。

独占锁：就是每次都只有一个线程运行，例如ReentrantLock。

共享锁：就是同时可以多个线程运行，如Semaphore、CountDownLatch、ReentrantReadWriteLock。

### AQS源码分析

AQS的源码可以看到对于**state共享变量**，使用**volatile关键字**进行修饰，从而**保证了可见性**

![](E:\workSpace\docx\material_docx\java\img\20211219173040.png)

从上面的源码中可以看出，对于state的修改操作提供了setState和compareAndSetState，那么为什么要提供这两个对state的修改呢？

因为compareAndSetState方法通常使用在获取到锁之前，当前线程不是锁持有者，对于state的修改可能存在线程安全问题，所以需要保证对state修改的原子性操作。

而setState方法通常用于当前正持有锁的线程对state共享变量进行修改，因为不存在竞争，是线程安全的，所以没必要使用CAS操作。

分析了AQS的源码的实现，接下来我们看看AQS的实现的原理。

### AQS实现原理

AQS中维护了一个**FIFO队列**，并且**该队列式一个双向链表**，链表中的每一个节点为**Node节点**，**Node类是AbstractQueuedSynchronizer中的一个内部类**。

AQS中Node内部类的源码

```
static final class Node {
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;
        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;
        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

可以看到上面的Node类比较简单，只是对于每个Node节点拥有的属性进行维护，在Node内部类中最重要的基本构成就是这几个：

```
volatile Node prev;
volatile Node next;
volatile Thread thread;
```

在FIFO队列中，头节点占有锁，也就是头节点才是锁的持有者，尾指针指向队列的最后一个等待线程节点，除了头节点和尾节点，节点之间都有前驱指针和后继指针。

在AQS中维护了一个共享变量state，标识当前的资源是否被线程持有，多线程竞争的时候，会去判断state是否为0，尝试的去把state修改为1。

## ReentrantLock

### Semaphore

### CountDownLatch

### ReentrantReadWriteLock

## CAS

### 简介

在分析ReentrantLock的具体实现的源码中，可以看出所有涉及设置共享变量的操作，都会指向CAS操作，保证原子性操作。CAS(compare and swap)原语理解就是比较并交换的意思，CAS是一种乐观锁的实现。在CAS的算法实现中有三个值：更新的变量、旧的值、新值。在修改共享资源时候，会与原值进行比较，若是等于原值，就修改为新值。于是在这里的算法实现下，即使不加锁，也能保证数据的可见性，即使的发现数据是否被更改，若是数据已经被更新则写操作失败。但是CAS也会引发ABA的问题。

### ABA问题

ABA问题就是假如有两个线程，同一时间读取一个共享变量state=1，此时两个线程都已经将state的副本赋值到自己的工作内存中。当线程一对state修改state=state+1，并且写入到主存中，然后线程一又对state=state-1写入到主存，此时主存的state是变化了两次，只不过又变回了原来的值。那么此时线程二修改state的时候就会修改成功，这就是ABA问题。对于ABA问题的解决方案就是加版本号（version），每次进行比较的时候，也会比较版本号。因为版本版是只增不减，比如以时间作为版本号，每一时刻的时间都不一样，这样就能避免ABA的问题。

### 性能分析

相对于synchronized的阻塞算法的实现，CAS采用的是乐观锁的非阻塞算法的实现，一般CPU在进行线程的上下文切换的时间比执行CPU的指令集的时间长，所以CAS操作在性能上也有了很大的提升。但是所有的算法都是没有最完美的，在执行CAS的操作中，没有更新成功的就会自旋，这样也会消耗CPU的资源，对于CPU来说是不友好的。
