#### 1、Serializable和Parcalable的区别。
Parcelable的设计初衷是因为Serializable效率过慢（使用反射），为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计，这些数据仅在内存中存在。Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。
- Parcelable的性能比Serializable好，因为后者在反射过程频繁GC，所以在内存间数据传输时推荐使用Parcelable，如activity间传输数据。

  而Serializable可将数据持久化方便保存，所以在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。 

-  Parcelable不能使用在要将数据存储在磁盘上的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点，但此时还是建议使用Serializable 。

#### 2、dpi、ppi、px、pt、dp、sp的区别。
- px(pixel)：像素，电子屏幕上组成一幅图画或照片的最基本单元
- pt(point)：点，印刷行业常用单位，等于1/72英寸
- ppi(pixel per inch)：每英寸像素数，ppi是指屏幕上的像素密度，该值越高，则屏幕越细腻。   

  dpi最初用于衡量打印物上每英寸的点数密度。dpi值越小图片越不精细。当dpi的概念用在计算机屏幕上时，就应称之为ppi。同理： PPI就是计算机屏幕上每英寸可以显示的像素点的数量。因此，在电子屏幕显示中提到的ppi和dpi是一样的
  
  `ppi= 屏幕对角线上的像素点数/对角线长度 = √ （屏幕横向像素点^2 + 屏幕纵向像素点^2）/对角线长度`

- dpi(dot per inch)：每英寸多少点，该值越高，则图片越细腻
- density：屏幕密度，每平方英寸上所包含的像素点。
- dp(dip)：Density-independent pixel, 是安卓开发用的长度单位，1dp表示在屏幕像素点密度为160ppi时1px长度
- sp:(scale-independent pixel)：安卓开发用的字体大小单位。
```
例如：4.95英寸 1920*1080的手机屏幕
计算公式：
dpi = √(1920^2 + 1080^2) / 4.95 = 445
density = dpi^2 = 445^2 = 198025
1dp = (dp * dpi) / 160 = (1 * 445) / 160 = 2.78px
```
**Android系统中定义的dpi：**      
![dpi](https://github.com/chen-eugene/Interview/blob/master/image/1539659587(1).png)       

意思就是：在 1920*1080 分辨率的手机上默认就使用 480 的 dpi ，不管的你的尺寸是多大都是这样，除非厂家手动修改了配置文件   

**在系统中使用的全部都是系统 dpi，没有使用物理 dpi，也获取不到物理 dpi。物理 dpi 主要用于厂家对于手机的参数描述，所以dp能够适配大多数手机，但不能适配所有手机。**



#### 3、应用最多占多少内存。
```
获取app的分配的最大内存
Runtime rt = Runtime.getRuntime();
long maxMemory = rt.getMaxMemory();

ActivityManager activityManager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
activityManager.getMemoryClass()；
activityManager.getLargeMemoryClass()；
```
在设置android:largeheap = "false"时和android:largeheap = "true"是获取到的最大内存存在明显的差异。

原则上Google原始OS的默认值是16m，但是各个厂商的OS会对这个值进行修改。

可以在/system/build.prop文件中查看内存的配置参数，
```
shell@NX510J:/ $ cat /system/build.prop | grep heap
dalvik.vm.heapsize=36m
   
dalvik.vm.heapstartsize=8m    ----起始分配内存
dalvik.vm.heapgrowthlimit=192m ---- 一般情况app申请的最大内存 dalvik.vm.heapsize=512m   ---- 设置largeheap时，App可用的最大内存dalvik.vm.heaptargetutilization=0.75  ---- GC相关
dalvik.vm.heapminfree=512k
dalvik.vm.heapmaxfree=8m     ----- GC机制相关
```
getMemoryClass()和getLargeMemoryClass()方法最终读取的仍然是dalvik.vm.heapgrowthlimit和dalvik.vm.heapsize的值。

**设置largeHeap的确可以申请到更多的内存，但是收到dalvik.vm.heapsize的限制，当内存很大时，每次gc将花费更多的时间，性能也会下降。**

#### [4、了解sp的apply和commit的区别吗，sp具体是怎么实现的，sp什么时候会将内存中的数据写入到文件中。](http://weishu.me/2016/10/13/sharedpreference-advices/)
Android 中的 SharedPreference 是轻量级的数据存储方式，能够保存简单的数据类型，比如 String、int、boolean 值等。其内部是以 XML 结构保存在 /data/data/包名/shared_prefs 文件夹下，数据以键值对的形式保存。

- 初始化：首次使用SP的时候会通过IO操作把xml文件读取并存入一个map对象中，所以get操作实际上是直接访问的map集合，提高了效率，如果xml不存在就重新创建一个。

  SP 在第一次创建后会维持一个SingleTon，以后都会返回这个实例，更新app时，xml文件并不会被删除。

- 读数据：首先取得 SharedPreferencesImpl 对象锁，然后同步等待从磁盘加载数据完成，最后返回数据。如果单个 SP 存储的内容过多，将会导致 get 方法的时候阻塞过长，特别是在主线程调用的时候，所以建议在单个SP中尽量少地保存数据。
- 写数据：SP 写入数据的操作是通过 Editor 完成的，一是将数据先写入内存中，即map集合；二是把数据写入到磁盘中。
  - commit：同步执行，会阻塞当前线程，有返回值。
  - apply：异步执行，没有返回值，推荐使用apply。 


