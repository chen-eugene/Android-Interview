#### [1、apk的安装流程。](https://blog.csdn.net/luoshengyang/article/details/6766010)
 
 Android系统在启动的过程中，会启动一个应用程序管理服务PackageManagerService，这个服务负责扫描系统中特定的目录，找到里面的应用程序文件，即以Apk为后缀的文件，然后对这些文件进解析，得到应用程序的相关信息，完成应用程序的安装过程.
 
 应用程序管理服务PackageManagerService安装应用程序的过程，其实就是解析析应用程序配置文件AndroidManifest.xml的过程，并从里面得到得到应用程序的相关信息，例如得到应用程序的组件Activity、Service、Broadcast Receiver和Content Provider等信息，有了这些信息后，通过ActivityManagerService这个服务，我们就可以在系统中正常地使用这些应用程序了。

 - ① SystemServer组件是由Zygote进程负责启动的，启动的时候就会调用它的main函数，系统服务的启动都是从main函数开始的。
 - ② 在这个调用链中会去创建一个ServerThread线程，PackageManagerService服务就是在这个线程中启动的。
 - ③ 在ServerThread的run方法中会去调用PackageManagerService的main函数，创建了一个PackageManagerService实例，并把它添加到ServiceManager中去，ServiceManager是Android系统Binder进程间通信机制的守护进程，负责管理系统中的Binder对象。
 - ④ PackageManagerService会去扫描移动设备上的五个目录（/system/framework、/system/app、/vendor/app、/data/app、/data/app-private）中的Apk文件，对每一个Apk文件进行解析和安装，并且把解析后的应用程序信息保存在PackageManagerService中。
 - ⑤ 这样就把前面解析应用程序得到的package、provider、service、receiver和activity等信息保存在了PackageManagerService服务中了。但是，这些应用程序只是相当于在PackageManagerService服务注册好了，如果我们想要在Android桌面上看到这些应用程序，还需要有一个Home应用程序，负责从PackageManagerService服务中把这些安装好的应用程序取出来，并以友好的方式在桌面上展现出来，例如以快捷图标的形式。

 
 

 


