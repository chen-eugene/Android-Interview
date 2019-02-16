#### [1、什么是线程，什么是进程，能不能只用进程。](https://segmentfault.com/a/1190000005884656)
  - 进程（Process）：是系统进行资源分配和调度的基本单位，是操作系统结构的基础。一个进程可以看作一个程序或者一个应用    
  - 线程(thread)：是程序执行的最小单元。线程是在进程中执行的一个任务。

引入线程的操作系统中，通常都是把进程作为分配资源的基本单位，而把线程作为独立运行和独立调度的基本单位。

区别：
  - ① 地址空间和其它资源（如打开文件）：进程间相互独立，同一进程的各线程间共享。某进程内的线程在其它进程不可见。
  - ② 通信：进程间通信IPC，线程间可以直接读写进程数据段（如全局变量）来进行通信——需要进程同步和互斥手段的辅助，以保证数据的一致性。
  - ③ 调度和切换：线程上下文切换比进程上下文切换要快得多。
  - ④ 在多线程OS中，进程不是一个可执行的实体。
  
#### [2、多线程编程的好处是什么。](https://www.cnblogs.com/dolphin0520/p/3910667.html)

  多个线程被并发的执行可以提高程序的效率，CPU不会因为某个线程需要等待资源而进入空闲状态。线程的创建只需要较少的资源，因此创建多个线程去执行任务比创建多个进程更好。
  
#### [3、一个线程包含了哪些状态。](https://www.cnblogs.com/dolphin0520/p/3920357.html)

![线程状态](https://github.com/chen-eugene/Interview/blob/master/image/061046391107893.jpg)

  - 可运行(runnable)：线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取cpu的使用权。
  - 运行(running)：可运行状态(runnable)的线程获得了cpu时间片（timeslice），执行程序代码。
  - 阻塞(block)：阻塞状态是指线程因为某种原因放弃了cpu使用权，也即让出了cpu时间片，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu 时间片转到运行(running)状态。阻塞的情况分三种： 
    - ① 等待阻塞：运行(running)的线程执行o.wait()方法，JVM会把该线程放入等待队列(waitting queue)中。 
    - ② 同步阻塞：运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。 
    - ③ 其他阻塞: 运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。
  - 死亡(dead)：线程run()、main()方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。
  
  
#### 4、wait()和sleep()的区别。
- sleep：让线程睡眠，交出CPU，让CPU去执行其他的任务。但是有一点要非常注意，sleep方法不会释放锁，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。
- wait：wait方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出CPU执行权限。
- yield：调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。
  注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。
- join：调用join方法是调用了Object的wait方法，由于wait方法会让线程释放对象锁，所以join方法同样会让线程释放对一个对象持有的锁。
  
  
#### 5、有三个线程T1，T2，T3，怎么确保它们按顺序执行？
  
  可以用线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。为了确保三个线程的顺序你应该先启动最后一个(T3调用T2，T2调用T1)，这样T1就会先完成而T3最后完成。
 
  
#### 6、如何停止一个线程。

  ![线程状态](https://github.com/chen-eugene/Interview/blob/master/image/db150002205d9bc30b8f.jpg)
  
  早期的jdk版本中提供了退出的方法：hread.stop, Thread.suspend, Thread.resume和Runtime.runFinalizersOnExit ，因为操作不安全，可能会出现不可控的结果，已不推荐使用。
  
  - ① 等线程自己执行完结束（这种最优雅，但是也没有讨论的意义了）。
  
  - ② 调用中断方法，判断中断标识。但是waiting状态下会抛异常，不能算作优雅。
  
 intercept：单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；另外，通过interrupt方法和isInterrupted()方法来停止正在运行的线程。直接调用interrupt方法不能中断正在运行中的线程。
  
```
public class Test {
     
    public static void main(String[] args) throws IOException  {
        Test test = new Test();
        MyThread thread = test.new MyThread();
        thread.start();
        try {
            Thread.currentThread().sleep(2000);
        } catch (InterruptedException e) {
             
        }
        thread.interrupt();
    } 
     
    class MyThread extends Thread{
        @Override
        public void run() {
            int i = 0;
            while(!isInterrupted() && i<Integer.MAX_VALUE){
                System.out.println(i+" while循环");
                i++;
            }
        }
    }
}
```
  - ③ 自己加中断状态标识。
    - waiting状态：此状态下退出只能调用中断方法。
    - blocked状态：分两种，一种可中断，调用中断方法退出；一种不可中断，只能等running后退出。
    - ready状态：等cpu调度，不可干预。
    - running状态：自己加共享状态标识。
  
 ```
 private volatile boolean exit = false;
 
 public void run(){
   //先判断状态
   try{
     while(!exit && !Thread.currentThread().isInterrupted()){
      System.out.prinln("running...");
      Thread.sleep(1000);
     }
   }catch(InterruptedException e){
    //中断了
   }finally{
    //退出后记得释放资源
   }
 }  
 ```


#### [7、一个线程发生异常会怎么样（Thread.UncaughtExceptionHandler原理）。](https://juejin.im/post/5b598e7a5188251aaa2d3901)

  线程出现异常如果没有使用`try-catch`进行捕获，JVM将调用Thread中的`Thread.dispatchUncaughtException`方法将异常传递给线程的为`UncaughtExceptionHandler`捕获异常处理器进行处理。
  
  Thread中存在两个UncaughtExceptionHandler。
    - 一个是静态的defaultUncaughtExceptionHandler：设置一个静态的默认的UncaughtExceptionHandler。来自所有线程中的Exception在抛出并且未捕获的情况下，都会从此路过。管辖范围为整个进程。
    - 另一个是非静态uncaughtExceptionHandler：为单个线程设置一个属于线程自己的uncaughtExceptionHandler，辖范围比较小。
  
  如果没有设置uncaughtExceptioniHander，将使用线程所在的线程组来处理这个未捕获异常。线程组ThreadGroup实现了UncaughtExceptionHandler，所以可以用来处理未捕获异常。
  ```
  public class ThreadGroup implements Thread.UncaughtExceptionHandler
  ```
  处理未捕获异常的逻辑：
   - 当出现未捕获异常时，JVM会调用`Thread.dispatchUncaughtException`方法。
   - 如果没有设置线程自己的uncaughtExceptioniHander，将会使用线程所在的ThreadGroup来处理这个异常，调用`ThreadGroup.uncaughtException`方法。
   - 如果存在父线程组，将异常消息通知给父线程组，
   - 如果父线程组不存在，然后尝试利用一个默认的defaultUncaughtExceptionHandler来处理异常。
   - 如果没有设置默认的defaultUncaughtExceptionHandler，则将异常信息输出到`System.err`。


#### 8、什么是守护线程。
  
  setDaemon和isDaemon用来设置线程是否成为守护线程和判断线程是否是守护线程。

  守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。


#### [9、ThreadLocal的设计理念与作用，ThreadPool用法与优势（AsyncTask底层有使用）。](https://www.cnblogs.com/hadoop-dev/p/7092935.html)

  - 首先，ThreadLocal 不是用来解决共享对象的多线程访问问题的，一般情况下，通过ThreadLocal.set() 到线程中的对象是该线程自己使用的对象，其他线程是不需要访问的，也访问不到的。各个线程中访问的是不同的对象。 
  - ThreadLocal使得各线程能够保持各自独立的一个对象,通过ThreadLocal.set()将对象的引用保存到各线程的自己的一个map中，每个线程都有这样一个map，执行ThreadLocal.get()时，各线程从自己的map中取出放进去的对象，因此取出来的是各自自己线程中的对象，ThreadLocal实例是作为map的key来使用的。 
  
  好处：每个线程都有自己的ThreadLocalMap类对象，线程只能访问自己的存放的对象，互不干扰，避免了线程之间的竞争。


#### 10、什么是线程安全，Vector是线程安全的吗。
  
  如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。
  
  一个线程安全的计数器类的同一个实例对象在被多个线程使用的情况下也不会出现计算失误。很显然你可以将集合类分成两组，线程安全和非线程安全的。Vector 是用同步方法来实现线程安全的, 而和它相似的ArrayList不是线程安全的。


#### [11、如何保证线程安全（如何保证多线程读写文件的安全）。](http://www.jasongj.com/java/thread_safe/)

  Java中提供了两种方式来解决多线程并发问题：锁(Lock)和同步(Synchronized)。
  
  使用锁(Lock)和同步(Synchronized)可以保证Java操作的原子性。还有一种保证原子性的方法，是同Java提供原子类，如AtomicInteger。
  
  无论使用锁还是synchronized，本质都是一样，通过锁来实现资源的排它性，从而实际目标代码段同一时间只会被一个线程执行，进而保证了目标代码段的原子性。这是一种以牺牲性能为代价的方法。


#### 12、说一说自己对于 synchronized 关键字的了解。

   synchronized关键字解决了多线程之间访问资源的同步性，可以保证被修饰的方法或者代码块在任意时刻只有一个线程执行。
   
   在早期的Java版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。
   
   在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。
   
   
#### 13、说说自己是怎么使用 synchronized 关键字，在项目中用到了吗。

  synchronized使用场景分析：
 - 当一个线程正在访问一个对象的synchronized方法，那么其他线程不能访问该对象的其他synchronized方法。这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他synchronized方法。

- 当一个线程正在访问一个对象的synchronized方法，那么其他线程能访问该对象的非synchronized方法。这个原因很简单，访问非synchronized方法不需要获得该对象的锁，假如一个方法没用synchronized关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，

- 如果一个线程A需要访问对象object1的synchronized方法fun1，另外一个线程B需要访问对象object2的synchronized方法fun1，即使object1和object2是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。


 synchronized关键字最主要的三种使用方式：  
   - 修饰方法：给当前的对象加锁，默认获取当前对象的锁。  
   - 修饰静态方法：给当前的类加锁会，作用于类的所有对象实例，默认获取的是当前类的类对象的锁。
   
   因为静态成员不属于任何一个实例对象（ static 表明这是该类的一个静态资源，不管new了多少个对象，只有一份，所以对该类的所有对象都加了锁）。
   
   所以如果一个线程A调用一个实例对象的非静态 synchronized 方法，而线程B需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象，因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁。 
   
   - 修饰代码块：需要指定锁对象，进入同步代码块前要获得指定对象的锁。 这里再提一下：synchronized关键字加到非 static 静态方法上是给对象实例上锁。另外需要注意的是：尽量不要使用 synchronized(String a) 因为JVM中，字符串常量池具有缓冲功能！
    
   **synchronized(this)代码块也是锁定的当前对象，与 synchronized 修饰方法一样。**
   **synchronized(class)代码块上是给 Class 类上锁，与 synchronized 修饰 static 静态方法一样。 **
   
   synchronized的一个使用案例：双锁的单例模式
   ```
   public class Singleton {
      //采用volatile修饰很有必要，在一定程度上避免了指令重排序
      private volatile static Singleton uniqueInstance;
  
      private Singleton() {}

      public static Singleton getUniqueInstance() {
         //先判断对象是否已经实例过，没有实例化过才进入加锁代码
         synchronized(this){
            if (uniqueInstance == null) {
               synchronized (this) {
                  if (uniqueInstance == null) {
                     uniqueInstance = new Singleton();
                  }
               }
            }
         }
        return uniqueInstance;
      }
   }
   ```
 采用双重锁的原因：`uniqueInstance = new Singleton()`并不是一个原子操作，赋值分为三个步骤：
   - ① 为uniqueInstance分配内存空间。
   - ② 初始化uniqueInstance。
   - ③ 将uniqueInstance指向分配的内存空间。
    
  其中使用volatile关键字是很有必要的，因为JVM存在指令重排序，所以整个赋值过程有可能是①->③->②，在多线程环境下会出现uniqueInstance没有初始化的情况。而volatile关键在在一定程度上禁止了JVM的这种指令重排序。
   
   
#### [14、讲一讲synchronized关键字的底层原理。](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247484539&idx=1&sn=3500cdcd5188bdc253fb19a1bfa805e6&chksm=fd98521acaefdb0c5167247a1fa903a1a53bb4e050b558da574f894f9feda5378ec9d0fa1ac7&token=1604028915&lang=zh_CN#rd)
   
   synchronized关键字底层原理属于JVM层面，问的有点深了。
   
   - synchronized代码块：synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。 
   
   当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。
   
   - synchronized方法：synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。
   
   
#### [15、讲讲 JDK1.6 之后的synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗?](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247484539&idx=1&sn=3500cdcd5188bdc253fb19a1bfa805e6&chksm=fd98521acaefdb0c5167247a1fa903a1a53bb4e050b558da574f894f9feda5378ec9d0fa1ac7&token=1604028915&lang=zh_CN#rd)
   
   这个问题也问的比较深。
   
   JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

   锁主要存在四中状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。
   
   
#### 16、谈谈synchronized和ReenTrantLock的区别。

  - ① 两者都是可重入锁，相比于synchronized，Lock不是Java语言内置的，Lock是一个接口，ReenTrantLock实现了Lock。
  
    可重入锁：自己可以再次获得自己已经获的对象锁，比如一个线程访问了某个对象的同步方法fun1，此时这个线程又去访问这个对象的另一个同步方法fun2，这个线程可以再次获取这个对象的锁，如果是不可重入锁的话，将会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。
    
  - ② synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API。
  - ③ 使用synchronized时不需要去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
  - ⑤ synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
  - ⑤ 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。
  
 相比于synchronized，ReenTrantLock所具有的高级特性：
  - 等待可中断：ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。使用synchronized时，等待的线程会一直等待下去，不能够响应中断。
  - 可实现公平锁：ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。
    - 公平锁：多个线程按照申请锁的顺序来获取锁，所谓的公平锁就是先等待的线程先获得锁。
    - 非公平锁：多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能会造成优先级反转或者饥饿现象。
  - 可实现选择性通知（锁可以绑定多个条件）：
    - synchronized关键字与wait()和notify/notifyAll()方法相结合可以实现等待/通知机制。
    - ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。
    
    Condition是JDK1.5之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 
    
    在使用notify/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知” ，这个功能非常重要，而且是Condition接口默认提供的。
    
    而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。
    Codition.await方法将会释放掉锁。
    
    
#### [17、什么是线程死锁，线程死锁的四个条件。](https://www.ibm.com/developerworks/cn/java/j-lo-deadlock/index.html)

  **死锁是指多线程环境中，线程之间循环等待其它线程所占用的资源而无限期等待下去的局面。**
  
  出现死锁的条件：
   - 互斥条件：一个资源每次只能被一个线程占有。
   - 请求和保持条件：线程至少已经占有一个资源，但又申请新的资源；由于该资源已被另外线程占有，此时该线程阻塞；但是，它在等待新资源之时，仍继续占用已占有的资源。
   - 不剥夺条件：进程所获得的资源在未使用完毕之前，资源申请者不能强行地从资源占有者手中夺取资源，而只能由该资源的占有者进程自行释放。
   - 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。
  
  [死锁的场景和如何避免死锁：](https://www.cnblogs.com/xiaoxi/p/8311034.html)
   - 加锁顺序（线程按照一定的顺序加锁）。
   - 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）。
  

#### [18、锁的等级：方法锁、对象锁、类锁。](https://blog.csdn.net/qq_35181209/article/details/72627363)
  
  Java的所有对象都含有一个互斥锁，这个锁由JVM自动获取和释放。
  
  - 方法锁：synchronized修饰方法时，默认的是当前的对象作为锁对象。
  - 对象锁：修饰代码块时，需要一个锁对象。
  - 修饰代码块：需要指定锁对象，进入同步代码块前要获得指定对象的锁。

  **方法锁也是一种对象锁。**
  

#### [19、Java锁的分类。](https://www.cnblogs.com/qifengshi/p/6831055.html)

- **公平锁/非公平锁**：
  - 公平锁：多个线程按照申请锁的顺序来获取锁。           
  - 非公平锁：多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。

   ReentrantLock：默认是非公平锁，可以通过构造函数参数设置成公平锁。Synchronized：是一种非公平锁。

- **可重入锁**：同一个线程在访问某个对象加锁的方法时获取了该对象的锁，在访问这个过程中又去访问其他加锁的方法，将会自动获取锁。

  synchronized和ReentrantLock都是可重入锁。    
                                              
- **独享锁/共享锁**：
  - 独享锁：指该锁一次只能被一个线程所持有。
  - 共享锁：指该锁可被多个线程所持有。

  ReentrantLock和Synchronized都是独享锁，ReadWriteLock的读锁是共享锁，写锁是独享锁。

- **互斥锁/读写锁**：独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
  - 互斥锁在Java中的具体实现就是ReentrantLock，Synchronized。
  - 读写锁在Java中的具体实现就是ReadWriteLock。

- **乐观锁/悲观锁**：乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
  - 悲观锁：对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
  - 乐观锁：对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的更新数据。乐观的认为，不加锁的并发操作是没有事情的。
  
  使用悲观锁和独占锁(其实是一个意思)的方式好处就是简单安全，但是挂起线程和恢复线程都需要转入内核态进行，这样做会带来很大的性能开销。悲观锁的代表是 synchronized。然而在真是的环境中，大部分时候都不会产生冲突。悲观锁会造成很大的浪费。
  
  而乐观锁不一样，它假设不会产生冲突，先去尝试执行某项操作，失败了再进行其他处理（一般都是不断循环重试）。这种锁不会阻塞其他的线程，也不涉及上下文切换，性能开销小。代表实现是 CAS。

  悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景。

- **分段锁**：分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。

#### [20、sychronized、CAS和AQS的区别，什么是CAS和AQS](https://juejin.im/post/5c37377351882525ec200f9e)

  **CAS：**
  CAS 是 compare and swap 的简写，即比较并交换。它是指一种操作机制，而不是某个具体的类或方法。是乐观锁的一种实现方式。在 Java 平台上对这种操作进行了包装。在 Unsafe 类中，调用代码如下：
  ```
  unsafe.compareAndSwapInt(this, valueOffset, expect, update);
  ```
 - valueOffset：内存位置
 - expect：期望值
 - update：新值
 
  它需要三个参数，分别是内存位置 V，旧的预期值 A 和新的值 B。操作时，先从内存位置读取到值，然后和预期值A比较。如果相等，则将此内存位置的值改为新值 B，返回 true。如果不相等，说明和其他线程冲突了，则不做任何改变，返回 false。
  
  这种机制在不阻塞其他线程的情况下避免了并发冲突，比独占锁的性能高很多。 CAS 在 Java 的原子类和并发包中有大量使用。

  Java中原子类的实现原理就是采用CAS机制来解决并发，参见AtomicInteger的代码。

  CAS 就是通过比较和交换操作的原子性的。值得注意的是， CAS 只是保证了操作的原子性，并不保证变量的可见性，因此变量需要加上 volatile 关键字。

  **AQS：以ReentranLock实现非公平锁来看AQS。**
  
  AQS 全称 AbstractQueuedSynchronizer。是Java提供的一套基于等待队列实现锁的框架。AQS 中有两个重要的成员：
  - 成员变量 state：state 为0表示没有任何线程持有这个锁，线程持有该锁后将 state 加1，释放时减1。多次持有释放则多次加减。实现可重入锁。
  - 还有一个双向链表，链表除了头结点外，每一个节点都记录了线程的信息，代表一个等待线程。这是一个 FIFO 的链表。
  
  锁请求的步骤：
  - 1. 如果没有线程持有锁，则请求成功，当前线程直接获取到锁。
  - 2. 如果当前线程已经持有锁，则使用 CAS 将 state 值加1，表示自己再次申请了锁，释放锁时减1。这就是可重入性的实现。
  - 3. 如果由其他线程持有锁，那么将自己添加进等待队列。
  ```
  ReentranLock源码
  final void lock() {
    if (compareAndSetState(0, 1))   
        setExclusiveOwnerThread(Thread.currentThread()); //没有线程持有锁时，直接获取锁，对应情况1
    else
        acquire(1);
  }

  public final void acquire(int arg) {
    if (!tryAcquire(arg) && //在此方法中会判断当前持有线程是否等于自己，对应情况2
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) //将自己加入队列中，对应情况3
        selfInterrupt();
  }
  ```
  非公平锁的实现：在获取锁失败之后，加入到等待队列中，当持有锁的线程释放掉锁的时候，还是会按照等待队列的顺序来获取锁。
  公平锁的实现：在获取锁的时候，先判断等待队列中是否有等待线程，如果有则直接加入到等待队列中，这样就保证了先等待的线程先获取到锁。 


#### [21、Java中volatile变量是什么。](https://www.cnblogs.com/dolphin0520/p/3920373.html)
- 原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
- 可见性：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
- 有序性：即程序执行的顺序按照代码的先后顺序执行。
- 指令重排序：一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

 处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令Instruction 2必须用到Instruction 1的结果，那么处理器会保证Instruction 1会在Instruction 2之前执行。
指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。

**要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。**

volatile：一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：
- 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。（保证可见性）
- 禁止进行指令重排序。（一定程度上保证有序性）

volatile关键字禁止指令重排序有两层意思：
 - 当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；

 - 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

```
x = 2;        //语句1
y = 0;        //语句2
volatile flag = true;  //语句3
x = 4;         //语句4
y = -1;       //语句5
```
　由于flag变量为volatile变量，那么在进行指令重排序的过程的时候，不会将语句3放到语句1、语句2前面，也不会讲语句3放到语句4、语句5后面。但是要注意语句1和语句2的顺序、语句4和语句5的顺序是不作任何保证的。

　　并且volatile关键字能保证，执行到语句3时，语句1和语句2必定是执行完毕了的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的。
　　
**volatile能够保证程序的可见性，在一定程度上保证有序性，但不能保证程序的原子性。**

使用volatile必须具备以下2个条件：

- 对变量的写操作不依赖于当前值

- 该变量没有包含在具有其他变量的不变式中


#### 22、为什么要使用线程池。

  线程池提供了一种限制和资源的原理（包括执行一个任务），每个线程池还维护一些基本统计信息，例如已完成任务的数量。
  
  使用线程池的好处：
   - 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
   - 提高相应速度：当任务到达时，任务可以不需要的等到线程创建就能立即执行。
   - 提高线程的客观理性：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
  
 
#### 23、Java默认提供了几种线程池。

- 核心线程数量(corePoolSize)：默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当。当任务队列满了并且线程数小于maximumPoolSize，那么会创建新的线程来执行任务。
- 最大线程数(maximumPoolSize)：表示在线程池中最多能创建多少个线程。
- 空闲线程存活时间(keepAliveTime)：表示线程没有任务执行时最多保持多久时间会终止。
  
  默认情况下只作用于最大线程，直到线程池中的线程数不大于corePoolSize，核心线程将会一直存在。
  但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0。
 
   **Java通过Executors提供四种线程池**
   
- newFixedThreadPool：corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；

  创建一个线程数量固定的线程池，当有任务来时，有空闲的线程就立即执行，否则就会将任务存入暂存的任务队列中，等待空闲的线程。

- newSingleThreadExecutor：corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；

  创建一个只有一个线程的线程池，若没有空闲线程，则先将任务保存到任务队列中去，等待空闲线程来执行。

- newCachedThreadPool：将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue。

  可以根据实际情况调节线程数量的线程池，线程池数量不确定，如有空闲的线程，则优先使用空闲的新城，否则来了任务就创建新的线程，当线程空闲超过60秒，就销毁线程。

- newScheduledThreadPool：corePoolSize和maximumPoolSize值相等，使用DelayedWorkQueue。

  DelayedWorkQueue是一个无界队列，它能按一定的顺序对工作队列中的元素进行排列。可以定期的执行任务。
  
  ![线程池结构图](https://github.com/chen-eugene/Interview/blob/master/image/20170406230435886.jpg)
  
  
  #### [24、什么时阻塞队列。](https://www.cnblogs.com/dolphin0520/p/3932906.html)
  
    当一个线程去获取一个阻塞队列的元素时，若队列为空，那么就会当前队列就会被阻塞，当队列有了元素之后，被阻塞的线程会被自动唤醒。
    
    阻塞队列的好处：非阻塞队列不会对当前线程产生阻塞，那么在面对类似消费者-生产者的模型时，就必须额外地实现同步策略以及线程间唤醒策略，这个实现起来就非常麻烦。如果使用阻塞队列就非常方便。
  
    Java提供的几种阻塞队列：
    - ArrayBlockingQueue：基于数组实现的一个阻塞队列，在创建ArrayBlockingQueue对象时必须制定容量大小。并且可以指定公平性与非公平性，默认情况下为非公平的，即不保证等待时间最长的队列最优先能够访问队列。
    - LinkedBlockingQueue：基于链表实现的一个阻塞队列，在创建LinkedBlockingQueue对象时如果不指定容量大小，则默认大小为Integer.MAX_VALUE。
    - SynchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
    - PriorityBlockingQueue：以上2种队列都是先进先出队列，而PriorityBlockingQueue却不是，它会按照元素的优先级对元素进行排序，按照优先级顺序出队，每次出队的元素都是优先级最高的元素。注意，此阻塞队列为无界阻塞队列，即容量没有上限（通过源码就可以知道，它没有容器满的信号标志），前面2种都是有界队列。
    - DelayedWorkQueue：基于PriorityQueue，一种延时阻塞队列，DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue也是一个无界队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。
      
  
 #### [25、如果提交任务时，线程池队列已满，会发生什么。](https://blog.csdn.net/qq_25806863/article/details/71172823)
  
 当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：
   - ThreadPoolExecutor.AbortPolicy：拒绝执行新的任务，直接抛出RejectedExecutionException异常。（默认的拒绝策略）
   - ThreadPoolExecutor.DiscardPolicy：不会执行新的任务，也不会抛出异常。
   - ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）。
   - ThreadPoolExecutor.CallerRunsPolicy：会调用当前线程池所在的线程去执行被拒绝的任务。

#### [26、实现Runnable接口和Callable接口的区别。](https://www.cnblogs.com/dolphin0520/p/3949310.html)

  Runnable和Callable都表示线程池要执行的任务，两者的区别在于Runnable不会返回线程的执行结果，而Callable可以返回线程的结果。执行结果使用Futrue来接收。
  ```
  public void test(){
    
    ExecutorService executor = Executors.newCachedThreadPool();
    Future<Integer> result = executor.submit(new Callable<Integer>{      
                                public Integer call() throw Exception{
                                  return 1 + 1;  
                                }
                              });   
  }
  ```
  Future：
  - get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
  - get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。
  
#### 27、执行execute()方法和submit()方法的区别是什么呢？
    
  - execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行情况。
  - submit()方法用于提交有返回值的任务，线程池返回一个Future对象，通过这个Future对象可以判断任务的执行情况。
  
  
#### 28、线程池需要销毁吗，如何进行销毁。

  线程池创建之后通常情况下是不需要销毁的，因为无法判断程序的执行过程中还有没有任务需要执行，这个并不是绝对的，需要根据具体的执行任务来判断。Java中在程序退出的时候需要将线程池销毁；而在Android中，线程池的生命周期通常和应用的生命周期相同，所以不需要纪进行销毁。
  
  销毁线程池的方法：
  - shutDown()：调用此方法之后，线程池立即进入SHUTDOWN状态，此时不能向线程池中添加新的任务，否则抛出RejectedExecutionException异常，线程不会立即退出，知道线程池中的任务执行完毕之后才会退出。
  - shutDownNow()：调用该方法之后，线程池立即进入STOP状态，并尝试停止所有正在执行的线程，不再处理还在等待队列中等待被执行的任务，并将等待被执行的任务返回。
  
     它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的(阻塞状态下，调用Thread.interrupt()将会抛出异常)。所以，ShutdownNow()并不代表线程池就一定能立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

  
  


   
 

  
  

 
  



  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  




