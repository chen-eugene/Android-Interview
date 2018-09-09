#### 1、Serializable和Parcalable的区别。
Parcelable的设计初衷是因为Serializable效率过慢（使用反射），为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计，这些数据仅在内存中存在。Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。
- Parcelable的性能比Serializable好，因为后者在反射过程频繁GC，所以在内存间数据传输时推荐使用Parcelable，如activity间传输数据。

  而Serializable可将数据持久化方便保存，所以在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。 

-  Parcelable不能使用在要将数据存储在磁盘上的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点，但此时还是建议使用Serializable 。

#### 3、dpi、ppi、px、pt、dp、sp的区别。
- px(pixel)：像素，电子屏幕上组成一幅图画或照片的最基本单元
- pt(point)：点，印刷行业常用单位，等于1/72英寸
- ppi(pixel per inch)：每英寸像素数，ppi是指屏幕上的像素密度，该值越高，则屏幕越细腻。
 dpi最初用于衡量打印物上每英寸的点数密度。dpi值越小图片越不精细。当dpi的概念用在计算机屏幕上时，就应称之为ppi。同理： PPI就是计算机屏幕上每英寸可以显示的像素点的数量。因此，在电子屏幕显示中提到的ppi和dpi是一样的
$ppi= 屏幕对角线上的像素点数/对角线长度 = √ （屏幕横向像素点^2 + 屏幕纵向像素点^2）/对角线长度$
- dpi(dot per inch)：每英寸多少点，该值越高，则图片越细腻
- dp(dip)：Density-independent pixel, 是安卓开发用的长度单位，1dp表示在屏幕像素点密度为160ppi时1px长度
- sp:(scale-independent pixel)：安卓开发用的字体大小单位。
