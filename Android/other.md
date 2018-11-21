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
   
   
   
   
   
