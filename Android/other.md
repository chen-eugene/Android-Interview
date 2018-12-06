#### 1、Android消息循环机制。

   消息机制主要包含：MessageQueue，Handler和Looper这三大部分，以及Message。
   
   **消息循环流程**：子线程执行完耗时操作，当Handler发送消息时，将会调用MessageQueue.enqueueMessage，向消息队列中添加消息。当通过Looper.loop开启循环后，会不断地从消息队列中读取消息，即调用MessageQueue.next，然后调用目标Handler（即发送该消息的Handler）的dispatchMessage方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用handleMessage方法，接收消息，处理消息。
   
   ![消息循环](https://github.com/chen-eugene/Interview/blob/master/image/3985563-d7da4f5ba49f6887.png)
   
   **MessageQueue，Handler和Looper三者之间的关系**：每个线程中只能存在一个Looper，Looper是保存在ThreadLocal中的。主线程（UI线程）已经创建了一个Looper，所以在主线程中不需要再创建Looper，但是在其他线程中需要创建Looper。每个线程中可以有多个Handler，即一个Looper可以处理来自多个Handler的消息。 Looper中维护一个MessageQueue，来维护消息队列，消息队列中的Message可以来自不同的Handler。
   
#### 2、关于Handler，在任何地方new Handler都是在什么线程下。

   ```
   public Handler(Callback callback, boolean async) {
       mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
   }
   ```

   通过Handler无参构造方法，默认获取的是当前线程ThreadLocal中的Looper对象，如果获取不到就会抛出异常，所以在子线程中，只要执行了`Looper.prepare()`方法，就可以有效的获取到Looper对象。
   
   在默认情况下，new Handler()获取的是当前线程的Looper，Handler对象就在当前线程中。当然也可以通过有参构造方法传入Looper对象，则Handler对象就属于Looper对象所在的线程中。

#### 3、HandlerThread的使用场景和原理。

   如果经常要开启线程和销毁线程，这无疑比较消耗性能，HandlerThread可以用来执行多个耗时操作，而不需要多次开启线程，里面是采用Handler+Looper实现的。
   
   ```
      public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
      }
   ```
   HandlerThread的run方法在新线程中创建Looper对象，并进行了关联。 在创建HandlerThread对象并调用其start方法之后，该HandlerThread线程就已经关联了looper对象（通过Looper.prepare()方法关联），并且该线程内部的消息队列循环了起来（通过Looper.loop()方法）。
  
#### 4、Android PendingIntent内部机制。


#### [5、Android ART机制，与dalvik的区别，JIT与AOT的区别是什么？ART GC有什么改善，还有什么缺点。](https://lrh1993.gitbooks.io/android_interview_guide/content/android/basis/dalvik-art.html)

  **Dalvik：**
  
  Dalvik是Google公司自己设计用于Android平台的Java虚拟机，支持dex格式（Dalvik Executable）的Java应用程序的运行。dex格式是专门为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统。
  
  Android2.2时添加了JIT，当APP运行时，JIT编译器就会对新类进行编译，经过编译后的代码，会被优化成相当精简的原生型指令码，这样在下次执行到相同逻辑的时候，速度就会更快。
  
  **Dalvik虚拟机和Java虚拟机的区别**
  
  - Java虚拟机运行的是Java字节码，Dalvik虚拟机运行的是Dalvik字节码。
  - Dalvik可执行文件体积小。Android SDK中有一个叫dx的工具负责将Java字节码转换为Dalvik字节码。
  - Java虚拟机与Dalvik虚拟机架构不同。这也是Dalvik与JVM之间最大的区别。Java虚拟机基于栈架构，Dalvik虚拟机基于寄存器架构。
  
  **优点：**
  
   - 安装速度快，占用空间小。
   
  **缺点：**
  
   - 运行时编译开销大，容易造成卡顿。
  
  
  **ART：**
  
  ART在应用安装时字节码就会被编译成指令码，使其成为真正的本地应用，在APP运行时不需要再进行编译，提高了启动速度和运行速度，这一机制叫Ahead-Of-Time (AOT）编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。
  
  **优点：**
  
   - 系统性能的显著提升。
   - 应用启动更快、运行更快、体验更流畅、触摸反馈更及时。
   - 更强的电池续航能力。
   - 支持更低的硬件。
   
  **缺点：**
  
   - 更大的存储空间占用，可能会增加10%-20%。
   - 更长的应用安装时间。
   
  

   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
