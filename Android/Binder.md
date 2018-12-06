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

#### 2、Binder是什么？它是如何实现跨进程通信的？（详细解释Binder机制。）

   相比较于其他的IPC方式，如管道、SystemV、Socket等：
   - Binder相对于传统的Socket方式，更加高效。Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，共享内存方式一次内存拷贝都不需要，但实现方式又比较复杂。
   - 传统的进程通信方式对于通信双方的身份并没有做出严格的验证，比如Socket通信的IP地址是客户端手动填入，很容易进行伪造。然而，Binder机制从协议本身就支持对通信双方做身份校检，从而大大提升了安全性。
   
   ![binder](https://github.com/chen-eugene/Interview/blob/master/image/20160220005120410.png)
   
   Binder架构包括Server、Client和Binder驱动三个部分。
   
   - Binder服务端：一个Binder服务端实际上就是Binder类的对象，该对象一旦创建，内部则会启动一个隐藏线程，会接收Binder驱动发送的消息，收到消息后，会执行Binder对象中的onTransact()函数，并按照该函数的参数执行不同的服务器端代码。onTransact函数的参数是客户端调用transact函数的输入。
   
   - Binder客户端：获取远程服务在Binder驱动中对应的mRemote引用，然后调用它的transact方法即可向服务端发送消息。
   
   - Binder驱动：任意一个服务端Binder对象被创建时，同时会在Binder驱动中创建一个mRemote对象，该对象也是一个Binder类。客户端访问远程服务端都是通过该mRemote对象。
   
   - Service Manager：是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力。
   
   **Binder客户端如何获取远程服务在Binder中的mRemote**
   
   - 系统服务是在系统启动的时候在SystemServer进程的init2函数中启动ServerThread线程，在这个线程中启动了各种服务，并且通过调用ServerManager.addService(String name, IBinder service)将其加入保存起来。ServerManager就相当于DNS服务器，在查找某个服务时通过调用ServerManager.getService(String name)函数就可以获得远程服务的Binder，至于它的具体细节可以查看Android启动相关的源代码。
   
   - 自定义的服务必须通过Service来实现。
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
