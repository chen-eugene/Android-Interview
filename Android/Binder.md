#### 1、Android进程的优先级。

  - ① 前台进程：一般前台进程都不会因为内存不足被杀死。
  
    - 拥有正在与用户交互的activity(已调用onResume)。
    - 拥有某个绑定到用户正在交互的Activity的Service。
    - 拥有正在“前台”运行的Service（调用了startFofeground）。
    - 拥有正在整形一个生命周期回调的Service（onCreate、onStart、onDestroy）。
    - 拥有正在执行其onReceive方法的BroadcastReceiver。
    
  - ② 可见进程：没有任何前台组件，但仍会影响用户在屏幕上所见内容的进程。除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。
  
    - 拥有不在前台、但仍对用户可见的 Activity（已调用 onPause()）。
    - 拥有绑定到可见（或前台）Activity 的 Service。
    
  - ③ 服务进程：尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。
  
    - 正在运行 startService() 方法启动的服务，且不属于上述两个更高类别进程的进程。
    
  - ④ 后台进程：后台进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。
  
    - 对用户不可见的 Activity 的进程（已调用 Activity的onStop() 方法）。
    
  - ⑥ 空进程：保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。
  
    - 不含任何活动应用组件的进程。

#### [2、Binder是什么？它是如何实现跨进程通信的？（详细解释Binder机制。）](https://blog.csdn.net/carson_ho/article/details/73560642)

  - 从机制、模型角度来说：Binder是一种跨进程(IPC)的方式，即Binder机制模型。
  - 从模型的结构、组成来说：Binder是一种虚拟的物理设备驱动，即Binder驱动。连接Server进程、Client进程和ServerManager进程。
  - Android源码角度：Binder是一个类，实现了IBinder接口。

   相比较于其他的IPC方式，如管道、SystemV、Socket等：
   - Binder相对于传统的Socket方式，更加高效。Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，共享内存方式一次内存拷贝都不需要，但实现方式又比较复杂。
   - 传统的进程通信方式对于通信双方的身份并没有做出严格的验证，比如Socket通信的IP地址是客户端手动填入，很容易进行伪造。然而，Binder机制从协议本身就支持对通信双方做身份校检，从而大大提升了安全性。
   
   ![binder](https://github.com/chen-eugene/Interview/blob/master/image/20181206225009.png)
   
   Binder架构包括Server、Client和Binder驱动三个部分。
   - Binder服务端：使用服务的进程。
   
   - Binder客户端：提供服务的进程。
   
   - Binder驱动：连接Server进程、Client进程和ServerManager进程的桥梁。
    - 传递进程间的数据：通过内存映射的方式。
    - 实现线程控制：采用Binder的线程池，并由Binder驱动自身进行管理。
   
   - Service Manager：是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力。
   
   **Binder驱动工作流程：**
   - ① Binder驱动创建一块接受缓存区。
   - ② 实现地址映射关系：即根据需要映射的接收进程信息，实现内核缓存区和接受进程用户空间地址同时映射到同一个共享接收缓存区中。
   - ③ 发送进程通过系统调用copy_from_user发送数据到虚拟内存区域（数据拷贝1次）。
   - ④ 由于内核缓存区和接收进程的用户空间地址存在映射关系（同时映射Binder创建的接收缓存区中），故相当于也发送到了接收进程的用户空间地址，即实现了跨进程通信。
   
   **优点：**
   - 传输效率：数据拷贝次数少（1次）、用户控件和内核空间可直接通过共享对象直接交互。
   - 为接收进程飞陪了不确定大小的接收缓存区。
   
   **Binder请求的线程管理：**
   - Server进程会创建很多线程来处理Binder请求。
   - Binder模型的线程管理采用Binder驱动的线程池，并由Binder驱动自身进程管理，而不是由Server进程来管理的。
   - 一个进程的Binder线程数默认最大是16，超过的请求会被阻塞等待空闲的Binder线程。
    所以，在进程间通信时处理并发问题时，如使用ContentProvider时，它的CRUD（创建、检索、更新和删除）方法只能同时有16个线程同时工作。

   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
