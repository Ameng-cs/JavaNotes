- # JUC(java.util.concurrent)
- ## 三个包 : 
    - **java.util.concurrent**
    - **java.util.concurrent.atomic**
    - **java.util.concurrent.locks**
- ## 锁:
    - ### 对象锁
        - 一个类里面如果有多个非静态synchronized方法,某一个时刻内,只能有一个线程访问该`对象`一个的synchronized方法.因为当某一线程访问该`对象`的synchronized方法的时候会`为该对象上锁`,上锁后其他线程就都不能进入该对象了.(因为对象被锁了嘛)
        - 一个类中有synchronized方法和普通方法,当一个线程访问synchronized方法后并为该对象加上了锁,其他线程访问`该对象的普通方法`将不受限制,因为普通方法并`不是临界资源`,所以其他线程调用普通方法并不会跟之前的线程构成竞争关系
        - 当有两个线程分别调用两个对象的synchronized方法时,他们会为各自的对象加锁,这两个对象互不竞争.(因为两把锁嘛,争的都不是一个东西)
    - ### 全局锁
        - 所有的静态同步方法用的是同一把锁--`类本身`.而非静态同步方法用的是对应的`对象锁`,这两把锁是不同的锁,所有静态同步方法和非静态同步方法之间是不会构成竞争的
        - 一旦一个静态同步方法获得锁后,其他的静态同步方法都必须等待之前的锁释放后才能获取锁,(不管他们是否是同一个对象)因为这个锁不是针对对象的而是针对该静态方法所属类的
    - ### 公平锁和非公平锁
        - **公平锁**:多个线程按照申请锁的顺序来获取锁
        - **非公平锁**:多个线程获取锁的顺序并不是按照申请锁的顺序有可能后申请锁的线程先拿到锁.在`高并发的情况下可能会造成优先级翻转或者是饥饿现象`
        - 两种锁的区别:
            - 公平锁:在并发环境中,先判断当前AQS的state是否等于0(0表示该锁没被占用),再判断此锁维护的等待队列中,本线程是否在第一个(在第一表示前面没有线程等待),如果在第一个则获得锁,如果不是第一个就会将本线程排到队尾
            - 非公平锁:先判断当前AQS的state是否等于0(0表示该锁没被占用),不管锁的等待队列,直接获取锁,如果获取失败再采用公平锁的方式
            - 非公平锁的吞吐量比公平锁大
        - ReentrantLock构造函数默认是非公平锁
        - Synchronized也是非公平锁
    - ### 可重入锁(递归锁)
        同一线程外层函数获得锁之后,内层递归函数仍能获得该锁的代码,在同一个线程外层方法获取锁的时候,在进入内层方法会自动获取锁.也就是说:`线程可以进入任何一个他已经拥有的锁所同步着的代码块`
        - 当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁
        - ReentrantLock/synchronized 是典型的可重入锁
        - 可重入锁的最大作用就是避免死锁
        - 使用 ReentrantLock的时候一定要`手动释放锁，并且加锁次数和释放次数要一样`,不然加锁和解锁的次数不一样会造成死锁
        - AQS通过控制status状态来判断锁的状态，对于非可重入锁状态不是0则去阻塞；对于可重入锁如果是0则执行，非0则判断当前线程是否是获取到这个锁的线程，是的话把status状态＋1，释放的时候，只有status为0，才将锁释放
    - ### 自旋锁
        指尝试获取锁的线程不会立即阻塞,而是`采用循环的方式去尝试获取锁`,这样的好处就是减少线程上下文切换的消耗,缺点是循环会消耗CPU
    - ### 独占锁(写锁)和共享锁(读锁)和互斥锁
        - 独占锁:只该锁一次只能被一个线程所持有.(ReentrantLock和synchronized都是独占锁)
        - 共享锁:该锁可以被多个线程所持有
        - 
- ## volatile
    - ### JMM java内存模型
    - JMM关于同步的规定
        - 线程解锁前,必须吧共享变量的值刷新回主内存
        - 线程加锁前,必须读取主内存的最新值到自己的工作内存
        - 加锁解锁是同一把锁
    - jvm为每个线程都会创造一个属于线程自己的工作内存,`每个线程的工作内存都是相互不可见的`.
    java规定所有的变量都存在主内存中,主内存是共享的,`各个线程如果操作变量的值,不能直接在主内存中修改,而是先要将变量从主内存拷贝到自己的工作内存,然后再将操作工作内存中的拷贝,回写给主内存`.
    - 造成的问题:
        一个线程对主存中的变量进行了修改,而另一个线程还在继续使用它拷贝到自己工作线程中的未修改的值,这就会造成数据不一致.
            ![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E6%95%B0%E6%8D%AE%E4%B8%8D%E4%B8%80%E8%87%B4.png)
    - 解决:
        把变量声明为`volatile`,这就会指示JVM,这个变量是不稳定的,每次使用都会到主存中进行读取.<br>说白了， volatile 关键字的主要作用就是保证变量的可见性然后还有一个作用是防止指令重排序
        ![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/volatile%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E5%8F%AF%E8%A7%81%E6%80%A7.png)
    - ### volatile是轻量级的同步指令
        - 保证可见性
            - 线程修改了volatile修饰的变量后,jvm会通知其他线程该值已经修改,让其重新拿值
        - 不保证原子性
            - 当多个线程争抢共享资源并都对其进行修改的时候,当线程将要将修改值写回主内存时,是有可能被挂起的,这样就会导致重复写的一个情况,如果保证原子性的话,他写回的过程是不会被打断的,要么修改成功,要么就不修改
            - 解决方法:<br>
                加sync:代价太大<br>
                使用Atomic原子类
        - 禁止指令重排
            - 计算机为了提高性能,编译器和处理器往往会对指令做重排.
            - 处理器在进行重排的时候必须要考虑指令之间的数据依懒性
            - 多线程环境中线程交替执行,由于编译器优化重排的存在,两个线程中使用的变量能否保证一致性是无法确定的,结果无法预测
- ## DCL单例模式
    - DCL: Double Check Lock:在加锁前和加锁后都进行一次判读
    - 普通的单例模式在多线程条件下会失效
    - DCL:添加了双重判断后,如果instance不加volatile关键字的话,由于允许指令重排,所以有几率会发生单例失效
    - 具体原因:
        - instance= newSingletonDemo()这个过程在底层实现是话是有三步的(1.分配内存2.加载类3.地址指向类),但是这三步都没有数据依赖性,所有编译器会进行指令重排,所以重排后先将地址指向了类,这是instance就不为null了,但是此时类还没有加载完也就是指向的地址不为空,但是对象还没加载出来是空的,这样就产生了错误

```java

package com.JUC;
//DCL(Double Check Lock)单例模式
public class SingletonDemo {
    //加volatile放在指令重排
    private  volatile static SingletonDemo instance = null;

    private SingletonDemo(){
        System.out.println(Thread.currentThread().getName()+":我是构造方法");
    }
    public static SingletonDemo getInstance(){
        //双端检索
        if (instance==null)
            synchronized (SingletonDemo.class){
                if(instance==null){
                    instance = new SingletonDemo();
                }
            }
        return instance;
    }

    public static void main(String[] args) {
        for (int i = 0; i <20; i++) {
            new Thread(()->{
                SingletonDemo.getInstance();
            },String.valueOf(i)).start();
        }
    }
}
```
- ## CAS（Compare And Swap）算法
- #### 定义：
     Compare And Swap（比较与交换），是一种有名的无锁算法。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）
    - #### CAS算法涉及三个操作数： 
    - 需要读写的内存值 **V**
    - 进行比较的值 **A**
    - 拟写入的新值 **B**
    

当且仅当 **V==A** 时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断的重试
- #### 过程演示：
1. 假如有两个线程 a 和 b 这两个线程都想去修改 x 的值 ，设 x == 56即 **V** == 56<br>
2. 线程在修改前都会先读取该变量x ，把读到的x的值 完全拷贝到自己的内存空间，这时线程a和b 对x都有一个预期的值56 该值也就是上面说的值 **A**
3. 假设a线程在线程竞争中得到了资源，能去更新x的值，他在更新之前先会检查内存中x的值是否等于自己对x的期望值 也就是判断是否有V==A，若相等，则代表该数据没有被修改，没发生冲突，然后线程 a 就会将新值 **B**（设为B  == 57）赋给x,写入内存,此时内存中x ==57,即V == 57。
4. b在线程竞争中失败了，但b线程不会被挂起，而只是被宣告这次线程竞争失败，而且可以再次尝试修改，若在a修改后，b得到了资源，他也会先检查此时内存中的值V 和他的期望值A是否相等，即判断是否有A==V，已知此时V已经被a线程修改为57 而B线程的期望值A仍为56 即A!=V 此时b的修改操作就会失败
* **注意：** **这其中的比较和替换是原子操作**，也就是说不会存在比较后成立但是还没替换时时间片轮转结束，而进入其他线程的情况，也即判断可修改但是没有修改的情况。

* ### CAS的局限性：
    - **ABA问题：**<br>
    如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 "ABA"问题。
    - **循环时间长，开销大：**<br>
    自旋CAS（也就是不成功就一直循环执行直到成功）**如果长时间不成功，会给CPU带来非常大的执行开销**。
    - **只能保证一个共享变量的原子操作：**<br>
    CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。
* ### ABA问题的解决
    * #### 原子引用
        * 包装一个自定义类为原子类,可以对这个类进行原子修改操作
        * class AtomicReference<T>
    * #### 时间戳原子引用
        * 为原子类增加一个版本号(时间戳),每修改一次,版本号(时间戳  )就加一.这样当产生ABA问题时就可以检测到原子类的版本号的差异,从而判断修改失败.
        * Class AtomicStampedReference<T> 

- ### CAS与synchronized的使用情景
    - **对于资源竞争较少（线程冲突较轻）的情况**，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而**CAS**基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能
    - **对于资源竞争严重（线程冲突严重）的情况**，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于**synchronized**。
- ### Unsafe类
    - 是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，UnSafe相当于一个后门，`基于该类可以直接操作特点的内存数据`。Unsafe存在于sun.misc包中，其内部方法操作可以像C的指针一样直接操作内存，因为Java中CAS操作的执行依赖于Unsafe类的方法
    - `注意Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务`
    - 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的
* ### CAS的实质:
    * CAS的全称为Compare And Swap ,`它是一条CPU并发原语`
    * 它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值，这个过程是原子的
    * CAS并发原语体现在JAVA语言中就是sun.misc.Unsafe类中的各个方法，调用UnSafe类中的CAS方法，JVM会帮我们实现出CAS汇编指令。这是一种完全依赖于硬件的功能，通过它实现了原子操作。
    由于CAS是一种系列原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且`原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU原子指令，不会造成所谓的数据不一致问题`
    * 也就是说CAS能保证原子性的原因是UnSafe类的方法调用的是cpu原语,而原语执行时保证原子性的
- ### 集合类的线程不安全
    - ArrayList,HashMap,HashSet,LinkedList这些集合类都是线程不安全的.
    - 解决方案:
        - 使用Collections里的工具类
        - 使用JUC下的包装类
            - CopyOnWriteArrayList
            - CopyOnWriteArraySet(底层还是CopyOnWriteArrayList) 
            - ConcurrentHashMap 
    - #### CopyOnWrite: 写时复制
        思想就是线程不在原数组上操作而是要操作数组的时候新建一个拷贝数组,在拷贝数组上操作,再将引用改为自己操作后的拷贝数组
        
        -  好处:多线程对容器进行并发的读的时候不会对线程加锁,因为容器不会被修改.
```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    //写的时候上锁,避免被抢资源
    lock.lock();
    try {
        //写之前得到原数组及长度
        Object[] elements = getArray();
        int len = elements.length;
        //创建一新数组,复制原来的数组并将长度+1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //将要添加的元素加入到新数组
        newElements[len] = e;
        //更新引用
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

```
- ### CountDownLatch
    - 让一些线程阻塞直至另外一些线程完成一些列操作后才能被唤醒
    - 主要方法:
        - await():阻塞方法,直至其他线程运行完
        - countDown():倒数方法,这个方法将计数器减一,当计数器的值为0时,被await方法阻塞的线程就会被唤醒,继续执行

```java
CountDownLatch countDownLatch = new CountDownLatch(5);
CountDownLatch countDownLatch2 = new CountDownLatch(1);
for (int i = 0; i < 5; i++) {
    new Thread(()->{
        System.out.println(Thread.currentThread().getName()+" 同学走啦!");
        countDownLatch.countDown();
    },String.valueOf(i)).start();
}
new Thread(()->{
    try {
        countDownLatch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName()+"关灯");
    countDownLatch2.countDown();
},"学习委员").start();
try {
    countDownLatch2.await();
} catch (InterruptedException e) {
    e.printStackTrace();
}
System.out.println("班长关门");
```

- ### CyclicBarrier
    - 可循环屏障,他会让一组线程到达同步点时被阻塞,直到最后一个线程达到同步点的时候屏障才开门,所有被拦截的线程才会继续运行.
    - 主要方法:
        - await(),阻塞线程,直到最后一个线程执行完成

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
    System.out.println("7个条件达成啦!");
});
for (int i = 0; i < 7; i++) {
    new Thread(()->{
        System.out.println("条件"+Thread.currentThread().getName()+"完成了!");
        try {
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+"继续后序动作");
    },String.valueOf(i)).start();
}
```
- ### Semaphore
    - 用于多个共享资源的互斥使用
    - 用于并发线程数的控制

```java
//三个共享资源
Semaphore semaphore = new Semaphore(3);
//六个线程竞争
for (int i = 0; i < 6; i++) {
    new Thread(()->{
        try {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName()+"抢到资源啦!!!!!!!");
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            semaphore.release();
            System.out.println(Thread.currentThread().getName()+"释放资源啦");
        }
    },String.valueOf(i)).start();
}
```
- ## 阻塞队列
    - 能处理生产者消费者问题
        - 当阻塞队列为空时:从队列中获取元素的操作会被阻塞
        - 当阻塞队列是满时:往队列里添加元素的操作会被阻塞   
    - 种类:
        - **ArryBlockingQueue**:由`数组结构`组成的`有界`阻塞队列。
        - **LinkedBlockingQueue**:由`链表结构`组成的`有界`（但大小默认值为`integer.MAX_VALUE`）阻塞队列
        - **SynchronousQueue**:不存储元素的阻塞队列，也即`单个元素的队列`
        - PriorityBlockingQueue:支持优先级排序的无界阻塞队列
        - DelayQueue:使用优先级队列实现的延迟无界阻塞队列
        - LinkedTransferQueue:由链表组成的无界阻塞队列
        - LinkedBlockingDeque:由链表组成的双向阻塞队列
    - 核心方法:

        | 方法类型 | 抛出异常  | 特殊值   | 阻塞   | 超时               |      |
        | -------- | --------- | -------- | ------ | ------------------ | ---- |
        | 插入     | add(e)    | offer(e) | put(e) | offer(e,time,unit) |      |
        | 移除     | remove()  | poll()   | take() | poll(time,unit)    |      |
        | 检查     | element() | peek()   | 不可用 | 不可用             |      |
    - 其他
        - 抛出异常:
            - 当阻塞队列满时，再往队列里add插入元素会抛IllegalStateExceptionQueuefull
            - 当阻塞队列空时，再往队列里remove移除元素会抛NoSuchElementException
        - 特殊值:
            - 插入方法，成功true失败false，不抛异常
            - 移除方法，成功返回出队列的元素，队列里没有就返回null
        - 一直阻塞:
            - 当阻塞队列满时，生产者线程继续往队列里put元素，队列会一直阻塞生产者线程直到put数据or响应中断退出
            - 当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用
        - 超时退出
            - 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出

