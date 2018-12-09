AMS是Android中最核心的服务，主要负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作。

  - 四大组件管理（activity,service,content provider,boardcast recever）。
  - 主要工作就是管理，记录，查询。
  - 四大组件进程通信的server端 四大组件属于client。
  - 属于系统进程的一部分。

相关类：
  - ActivityStack.java:其实是个管理类，管理activity的各种状态
  - ActivityRecord.java:ActivityStack的管理对象，每个Activity在AMS对应一个ActivityRecord，来记录Activity的状态以及其他的管理信息。其实就是服务器端的Activity对象的映像
  - ActivityThread.java:主线程，类中有main方法。
  - H.java:Handler子类。
  - Instrumentation.java:这个东西我把它理解为ActivityThread的一个工具类，也算是一个劳动者吧，对于生命周期的所有操作例如onCreate最终都是直接由它来执行的。对于hook和测试会用到这个类。
  - ApplicationThread.java:用来实现ActivityManagerService与ActivityThread之间的交互。在ActivityManagerService需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通讯。
