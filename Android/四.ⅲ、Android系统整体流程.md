AMS是Android中最核心的服务，主要负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作。

  - 四大组件管理（activity,service,content provider,boardcast recever）。
  - 主要工作就是管理，记录，查询。
  - 四大组件进程通信的server端 四大组件属于client。
  - 属于系统进程的一部分。

相关类：
  - ActivityStack.java：其实是个管理类，管理activity的各种状态
  - ActivityRecord.java：ActivityStack的管理对象，每个Activity在AMS对应一个ActivityRecord，来记录Activity的状态以及其他的管理信息。其实就是服务器端的Activity对象的映像
  - ActivityThread.java：主线程，类中有main方法。
  - H.java：Handler子类。
  - Instrumentation.java：可以理解为ActivityThread的一个工具类，用来监控应用程序和系统的交互，对于生命周期的所有操作例如onCreate最终都是直接由它来执行的。对于hook和测试会用到这个类。
  - ApplicationThread.java:用来实现ActivityManagerService与ActivityThread之间的交互。在ActivityManagerService需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通讯。
  

#### [1、apk的安装流程。](https://blog.csdn.net/luoshengyang/article/details/6766010)
 
 Android系统在启动的过程中，会启动一个应用程序管理服务PackageManagerService，这个服务负责扫描系统中特定的目录，找到里面的应用程序文件，即以Apk为后缀的文件，然后对这些文件进解析，得到应用程序的相关信息，完成应用程序的安装过程.
 
 应用程序管理服务PackageManagerService安装应用程序的过程，其实就是解析析应用程序配置文件AndroidManifest.xml的过程，并从里面得到得到应用程序的相关信息，例如得到应用程序的组件Activity、Service、Broadcast Receiver和Content Provider等信息，有了这些信息后，通过ActivityManagerService这个服务，我们就可以在系统中正常地使用这些应用程序了。

 - ① SystemServer组件是由Zygote进程负责启动的，启动的时候就会调用它的main函数，系统服务的启动都是从main函数开始的。
 - ② 在这个调用链中会去创建一个ServerThread线程，PackageManagerService服务就是在这个线程中启动的。
 - ③ 在ServerThread的run方法中会去调用PackageManagerService的main函数，创建了一个PackageManagerService实例，并把它添加到ServiceManager中去，ServiceManager是Android系统Binder进程间通信机制的守护进程，负责管理系统中的Binder对象。
 - ④ PackageManagerService会去扫描移动设备上的五个目录（/system/framework、/system/app、/vendor/app、/data/app、/data/app-private）中的Apk文件，对每一个Apk文件进行解析和安装，并且把解析后的应用程序信息保存在PackageManagerService中。
 - ⑤ 这样就把前面解析应用程序得到的package、provider、service、receiver和activity等信息保存在了PackageManagerService服务中了。但是，这些应用程序只是相当于在PackageManagerService服务注册好了，如果我们想要在Android桌面上看到这些应用程序，还需要有一个Home应用程序，负责从PackageManagerService服务中把这些安装好的应用程序取出来，并以友好的方式在桌面上展现出来，例如以快捷图标的形式。
 
 
#### [2、Launcher启动过程](https://blog.csdn.net/luoshengyang/article/details/6767736)


#### [3、Android应用进程的启动过程](https://blog.csdn.net/luoshengyang/article/details/6747696)
  
  - Android应用程序进程的入口函数是`ActivityThread.main`，即进程创建完成之后，Android应用程序框架层就会在这个进程中将ActivityThread类加载进来，然后执行它的main函数，这个main函数就是进程执行消息循环的地方了。
  
  - Android应用进程天然支持Binder进程间通信。当我们在Android应用程序中实现Server组件的时候，我们并没有让进程进入一个循环中去等待Client组件的请求，然而，当Client组件得到这个Server组件的远程接口时，却可以顺利地和Server组件进行进程间通信，这就是因为Android应用程序进程在创建的时候就已经启动了一个线程池来支持Server组件和Binder驱动程序之间的交互了，这样，极大地方便了在Android应用程序中创建Server组件。


#### [4、Android应用程序的启动过程](https://blog.csdn.net/luoshengyang/article/details/6689748)

  - ① 论是通过Launcher来启动Activity，还是通过Activity内部调用startActivity方法来启动新的Activity，都通过Binder进程间通信进入到ActivityManagerService进程中，并且调用ActivityManagerService.startActivity方法。
  - ② 通过桌面快捷方式获取Intent，向Intent中添加`Intent.FLAG_ACATIVITY_NEW_TASK`，最终调用到ActivityManagerService.startActivity方法。
  - ③ 解析Intent内容，包括类名、启动模式等。
    - 如果启动模式为singleTask，就回去查找是否存在Activity相关联的Task没有就从新创建；
    - 如果启动模式为singleInstance，就会去查找是否存在要启动的Activity实例，如果没有，就创建一个新的Task。
    - 新创建的Task被添加到了ActivityManagerService中。
  - ④ 获取栈顶的Activity是否为即将激动的Activity，如果是，就不会在重新创建一个Activity。否则就将要启动的Activity的ActivityRecord存入栈中。
  - ⑤ 继续判断要启动的Activity是否为Resumed状态，如果是的话就不做任何操作。
  - ⑥ 通过当前状态为Resumed的Activity所在的进程的ApplicationThread对象，来通知Resumed状态的Activity（即Launcher）进入Paused状态。
  - ⑦ 根据清单文件的Application的process属性，系统默认使用package的名称，去查找进程是否存在，由于是第一次启动程序，所以会创建一个新的进程。并且执行到ActivityThread的mian函数，创建一个ActivityThread实例，每一个应用程序都有一个ActivityThread实例与之对应。
  - ⑧ 通知ActivityManagerService来正真启动一个Activity，并且回调对应的生命周期函数，在客户端创建一个正真的Activity实例（前面栈中保存的都是ActivityRecord），MainActivity启动起来了就表示整个应用程序也启动起来了。
  
  
