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
