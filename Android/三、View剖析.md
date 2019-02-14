#### [1、Touch事件的分发流程。](https://blog.csdn.net/carson_ho/article/details/54136311)

[ViewGroup事件传递](https://blog.csdn.net/yanbober/article/details/45912661)  
[View事件传递](https://blog.csdn.net/yanbober/article/details/45887547)

Touch事件的分发过程有三个重要的方法来完成：dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent。 

**分发流程：**
- Android事件派发是先传递到最顶级的ViewGroup，再由ViewGroup递归传递到View的。
- 在ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouchEvent方法返回true不在向下继续传递它的onTouchEvent会被调用去处理此事件；如果这个ViewGroup的onInterceptTouchEvent方法返回false就表示它不拦截当前事件，此将会递归传递给子视图。

-  当一个View需要处理事件时，如果它设置了OnTouchListener，那么OnTouchListener中的onTouch方法会被调用，如果onTouch返回false，则当前View的onTouchEvent方法会被调用；如果返回true，则onTouchEvent将不会被调用，表示当前View消耗掉了此事件。

- 在整个事件传递过程中，如果一个View的onTouchEvent返回false，那么他的父视图的onTouchEvent方法将会被调用，以此类推。如果所有的View都不处理当前事件，那么此事件最终将会被传递给Activity处理，即Activity的onTouchEvent方法会被调用。

**注意点：**
- View没有onInterceptTouchEvent方法，一点有事件传递给它，那么它的onTouchEvent方法将会被调用。

- 如果DOWN事件没有被消耗，那么后续的事件将不会继续下发。

- 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件(onTouchEvent返回false)，那么它将不会接收到同一个事件序列的后续事件，并且事件将会重新交给他的父视图处理，即父视图的onTouchEvent方法会被调用。

- 如果View不消耗除ACTION_DOWN意外的其他事件，此父视图的onTouchEvent方法不会被调用，但是当前的View可以持续接收到后续的事件，最终事件会被传递给Activity处理。

- dispatchTouchEvent回true就是消费事件，这种说法不完全正确。dispatchTouchEvent事件派发是传递的，如果返回值为false将停止下次事件派发，如果返回true将继续下次派发。
 
 
#### [2、点击事件被拦截，但是想传到下面的view，如何操作。](https://blog.csdn.net/xw13782513621/article/details/77528518)
 
  如果`onInterceptTouchEvent`返回true，那么本次touch事件之后的所有action都不会在向下传递，将会交给此View的`onTouchEvent`方法处理。
  子视图调用`getParent().requestDisallowInterceptTouchEvent(true)`方法，将会阻止父视图拦截touch事件，此touch事件的后续action也不会被拦截。
 
  
#### [3、具体问题如下图所示](https://www.jianshu.com/p/94307344eed4)

 ![第三题](https://github.com/chen-eugene/Android-Interview/blob/master/image/wqiruy.png)
 
 不只是重写onTouchEvent方法，重写onDispatchTouchEvent方法是一样的道理。返回true表示消耗事件，剩下所有的事件会传递到这里，由它处理。返回false表示不消耗事件，正常传递，并最终向上交还给父视图。
 
 问题分析：
 - C的onTouchEvent方法返回true，那么它消耗了DOWN事件，其父视图将会继续分发后续的事件。
 - B拦截MOVE事件，那么MOVE和后续事件将会最终传递到B，不会再继续向下分发。
 - 所以C只能接收到DOWN事件，B能够接收到整个事件序列。
 
 
  
#### 4、View的位置参数有哪些，left、x、translationX的含义以及三者的关系。
- view的位置由left、top、right、bottom四个属性决定，这几个坐标可以通过getLeft()、getTop()、getRight()、getBottom()获取。注意这四个坐标是相对坐标，即相对于父容器的坐标。当view发生移动时，这几个坐标是不变的。

- 从Android 3.0开始，增加了几个参数：x、y、translationX、translationY，都是相对于父容器的坐标。

   x指view左上角的横坐标，当view发生移动时，x会变化；

   translationX指view左上角的横坐标相对于父容器的偏移量，当view发生移动时，translationX会变化。    
   `x = left + translationX `

- rawX是绝对坐标，是相对于屏幕左上角的横坐标，view本身没有getRawX的方法，这个方法一般在MotionEvent对象里使用。

- scrollX指的是view在滑动过程中，view的左边缘和view内容的左边缘在水平方向的距离（注意与translationX 的区别，translationX 指的是view本身的移动，scrollX是view的内容移动），也就是说调用了view的scrollTo或scrollBy方法，view本身不会移动，只会移动view的内容。    
![坐标图](https://github.com/chen-eugene/Interview/blob/master/image/20160808154319878.png)
  
  
#### 5、什么是MeasureSpec。
MeasureSpec是一个32位的int值，高2位表示SpecMode，指测量模式；低30位表示SpecSize，指在某种测量模式下的规格大小。MeasureSpec一旦确定后，onMeasure方法就可以确定View的测量宽/高。

- 对于DecorView，其MeasureSpec由窗口的尺寸和其资深的LayoutParams来共同决定。
- 对于普通的View，其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams共同决定。  

![MeasureSpec](https://github.com/chen-eugene/Interview/blob/master/image/20170311114110621.jpg)
  
  
#### 6、View绘制过程。
ViewRoot对应于ViewRootImpl类，是链接WindowManager和DocorView的纽带，View的三大流程均是通过ViewRoot来完成的。在ActivityThread中，当Activity对象被创建完毕后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联。

View的绘制流程就是从ViewRoot的performTraversals方法开始的，经过measure、layout和draw三个过程之后最终将View绘制出来。

**measure过程：**
 - View的measure过程：根据父类的measureSpec和自身的layoutParams来确定自身的measureSpec。

    直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时的自身大小，否则在布局中使用wrap_content就相当于match_content。
  
 - ViewGroup的measure过程：ViewGroup没有重写onMeasure方法，但是提供了measureChild方法来循环遍历，ViewGroup会根据子元素的大小来测量自己的大小，但不能超过父容器剩余的大小。

**layout过程：**
- 当ViewGroup的位置被确定后，它会在onLayout方法中遍历所有的子元素并调用其layout方法，在layout方法中onLayout方法又会被调用。
- layout方法确定View本身的位置，而onLayout方法则会确定所有子元素的位置。

**draw过程：**
- 绘制背景background.draw(canvas)。
- 绘制自身(onDraw)。
- 绘制children(dispatchDraw)。
- 绘制装饰（如前景，scrollbar等）。
  
  
#### 7、怎么获取View的宽高。
**getMeasuredWidth和getWidth的区别：**      

 - getMeasuredWidth返回的是View的测量宽/高，但某些情况下（如ListView）系统可能需要多次测量才能确定最终的测量宽/高，此时在onMeasure方法中拿不到准确的测量宽度。
 - getWidth返回是最终宽度，通常情况下，View的测量宽/高就等于最终宽/高，但也存在极端情况导致两者不一致（如重写layout方法，手动修改最终宽/高）。
 - getMeasureWidth必须在onMeasure之后才有效，getWidth必须在onLayout之后才有效。

**获取View的宽/高：**    

View的measure过程和Activity的生命周期不是同步执行的，所以不能通过Activity的生命周期来直接获取宽/高。

- Activity/View#onWindowFocusChanged：View初始化已经完成时会调用。

   需要注意的是：当Activity得到焦点和失去焦点时均会被调用一次，如果频繁的执行onResume和onPause，那么onWindowFocusChanged将会被频繁调用。

- view.post：通过post可以将runnable投递到消息队列的尾部，当Looper调用此runnable的时候，View已经初始化完成。
- ViewTreeObserver：随着View树状态的改变，onGlobalLayout会被多次调用。
- view.measure(int widthMeasureSpec,int heightMeasureSpec)通过手动对View进行测量来得到宽/高，但也分情况处理：
  
   match_parent：无法测量具体的宽/高，因为构造此种MeasureSpec需要知道parentSize，这个时候无法知道parentSize大小，所以测不出View的大小。

   具体的数值：
```
比如宽/高都是100px
int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
view.measure(widthMeasureSpec, heightMeasureSpec);
```
   wrap_content：
```
int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec((1 << 30) - 1, View.MeasureSpec.AT_MOST);
int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec((1 << 30) - 1, View.MeasureSpec.AT_MOST);
view.measure(widthMeasureSpec, heightMeasureSpec);
```
  
  
#### 8、在Activity的onCreate方法中能不能获取宽高，在onResume方法呢？ui绘制流程和Activity的生命周期有什么关系，ui开始绘制的时机到底是什么时候？

 在onCreate中获取View的宽高都是0，因为View还没有进行测量和绘制，在onResume方法中同样也无法获取到View的宽高。
 
 在Activity的attach方法中，会为Activity创建一个Window对象，在`Activity.handleResumeActivity`方法中才会去创建ViewRootImpl并添加视图，这个过程在`performResumeActivity`方法调用完成之后，所以在onResume中也不能获取到View的宽高，View还没有开始绘制。所以UI绘制的起点是在onResume方法调用完成之后。
 
 
#### 9、Android能够在子线程中更新UI吗？为什么要设计成不能在子线程中更新UI？
 
 ```
 void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
 }
 ```
 
 更新UI是监测是否为UI线程是由ViewRootImpl的checkThread方法来完成的，而ViewRootImpl的创建是在`ActivityThread.handleResumeActivity`方法中。所以在第一次onResume方法调用的时候或者之前的回调方法中，在子线程中更新UI是不会抛出异常的。
 
 
 这样设计的考虑：Android的UI操作并不是线程安全的，如果允许在子线程中更新UI的话，必然会导致界面混乱。想要避免这个问题就必须采用加锁来保证UI操作的线程安全，这势必会导致效率的下降。所以为了提高UI操作的效率和UI操作的线程安全，就禁止在子线程中更新UI。 
 
  
#### 10、invalidate、postInvalidate、requestLayout的区别。

- invalidate：请求重绘View树（也就是draw方法），如果View大小没有发生变化就不会调用layout过程，并且指挥绘制哪些需要重绘的View，也就是哪个View（View只绘制该View，ViewGroup绘制整个ViewGroup）请求invalidate方法就绘制该View。
- postInvalidate：invalidate方法只能在UI线程中请求，postInvalidate可以在非UI线程中调用。
- requestLayout：requestLayout()方法会调用measure过程和layout过程，不会调用draw过程，也不会重新绘制任何View包括该调用者本身。
  
  
#### 11、自定义View的流程，自定义View需要注意的问题，例如自定义View是否需要重写onLayout，onMeasure。

**自定义View的流程：**

- 集成特定的View（如TextView）。 

  主要用于扩展已有的View的功能，这种方法不需要自己支持wrap_content和padding。

- 继承特定的ViewGroup（如LinearLayout）

  主要用于将几种View进行组合，这种方法不需要自己处理ViewGroup的测量和布局。

- 继承View重写onDraw方法。

   主要用于不规则的效果，这种方式需要对View进行绘制，重写onDraw方法，并且采用这种方法需要自己支持wrap_content和padding。

- 继承ViewGroup。

  相当于自定义新的布局，这种方式需要自己处理ViewGroup的测量和布局两个过程，并且同时处理子视图的测量和布局过程。
  
**注意点：**
- 集成子View或ViewGroup的控件，如果不在onMeasure方法中对warp_content进行处理，那么设中为warp_content的时候，实际的大小为parentSize。
- 直接继承自ViewGroup的控件需要在onMeasure和onLayout中考虑padding和子视图margin对其造成的影响，不然将导致padding和子视图的margin失效。
- View带有滑动嵌套情形时，需要处理好滑动冲突。
- View中如果有线程或者动画，需要及时停止。

   当包含View的Activity退出或者当前View被remove时，View的onDetachedFromWindow方法会被调用，和此方法对应的是onAttachedToWindow，当View变得不可见时同样需要停止线程和动画，否则可能会造成内存泄漏。
   
   
#### 12、动画的原理，底层如何给上层信号。

 - 补间动画：通过确定开始的视图样式和结束的视图样式，中间变化过程由系统补全。包括平移、缩放、旋转和透明度四种变换。**只能作用于View。**

 - 逐帧动画：将动画拆分成帧的形式，且定义一帧为一张图片，按顺序播放一组预先定义好的图片。
 
 - 属性动画：在一定时间间隔内，通过不断对值进行改变和不断查找对象的set方法将值赋给该对象，从而实现该对象在该属性上的动画效果。**可以作用任意对象。**
 

#### [13、插值器和估值器。](https://blog.csdn.net/carson_ho/article/details/72863901)

 - 插值器：设置属性值从初始值过渡到结束值的变化规律，即确定了动画变化的趋势，如匀加速，匀减速等。
 
 - 估值器：设置属性值从初始值过渡到结束值的变化具体数值，即确定了动画变化的具体效果。如修改颜色，大小等。
 
 
#### 14、[RecyclerView和ListView复用机制。](https://www.jianshu.com/p/9306b365da57)

 Recycler是RecyclerView的内部类，是整个复用机制的核心，专门向外提供复用机制的api。Recycler的几个属性变量十分重要。
 - mCachedViews：默认大小为2，可以理解为一级缓存，只有跟中条件都匹配成功(postion是否一致，type是否相同，ViewHolder是否被remove掉)，就可以直接进行复用，无需绑定数据。
 
 - RecyclerViewPool：是RecyclerView的内部类，二级缓存，会根据不同的item type来创建不同的List，每个List的默认大小为5。只要type值匹配成功，那么就可以复用，需要调用onBindViewHolder方法，绑定数据。
 
 复用机制：
 
 - 滑动过程中的回收和复用都会优先从mCacheViews集合中查找，只有原来的卡位可以重新复用这个 ViewHolder，新位置的卡位无法从 mCachedViews 里拿 ViewHolder出来用。
 
   怎么理解呢：当滑动RecyclerView使一二个位置上的item移出屏幕，那么会被优先缓存到这个集合中，当再次滑动使得一二位置上的item移入屏幕，那么见直接从mCacheViews中查找，如找到直接复用，不需要重新填充数据。
 
 - 如果mCachedViews中没有找到可以复用的ViewHolder，那么将在ViewPool中查找。不同于mCacheViews的查找规则那么严格(postion是否一致，type是否相同，ViewHolder是否被remove掉)，ViewPool中只要存在匹配上type，那么就可以复用，并且会重新调用onBindViewHolder绑定数据。
   
 - 如果以上都没有找到可以复用的ViewHodler，那么就会调用onCreateViewHolder创建新的ViewHolder。
 
 回收机制：
 
 - 首先会将回收的ViewHolder放入mCacheViews中，如果mCacheView已经满了，那么就会将mCacheViews中第0个位置上的ViewHolder放到ViewPool中去，第1的位置向前移动，然后在放入最新的ViewHolder。
 
 - 回收完成之后会回调onRecycled方法。


#### 15、处理滑动的几种方式，Scroller滑动的原理。

 ![滑动冲突场景](https://github.com/chen-eugene/Interview/blob/master/image/1543826555.png)
 
 - 场景1：外部滑动方向和内部滑动方向不一致。如ScrollView嵌套RecyclerView。
 
  处理规则：当用户左右滑动时，需要让外部的View拦截事件，当用户上下滑动时，需要让内部的View拦截事件。
 
 
 - 场景2：外部滑动方向和内部滑动方向一致。
 
  处理规则：根据业务状态，来决定让那一层的View滑动。
 
 - 场景3：场景1和场景2的嵌套。


#### 16、简述工作线程更新UI的方法。
 
 - 通过handler.sendMessage，更新UI主线程。
 
 - 通过handler.post，更新UI主线程。
 
 - 使用runOnUiThread，更新UI主线程。
 
 - 通过view.post，更新UI主线程。
 
 
 
#### 17、Selector是怎么实现的。


#### [18、View动画是怎么实现的，为什么移动后点击事件还在原来的位置，属性动画的原理机制。](https://www.jianshu.com/p/3530ba1e9885)

  **补间动画：**
  ```
  View通过调用startAnimation()方法设置补间动画。
  public void startAnimation(Animation animation) {
        animation.setStartTime(Animation.START_ON_FIRST_FRAME);
        setAnimation(animation);
        invalidateParentCaches();
        invalidate(true);
  }
  ```
  由此可见补间动画是通过调用invalidate()方法来进行重绘，只会触发onDraw方法，不会调用onMeasure和onLayout方法，也就是说虽然View的原本位置是没有改变的。
  
  **属性动画：**
  属性动画设置完成之后调用start开始执行动画，最终会执行到ValueAnimator的shceduleAnimation方法
  ```
        private void scheduleAnimation() {
            if (!mAnimationScheduled) {
                mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, mAnimate, null);
                mAnimationScheduled = true;
            }
        }
  ```
  由此可见属性动画通过Choregrapher分发VSnyc信号来更新的，这个过程如果检测到View的属性值发生变化，那么将会重新执行View的onMeasure、onLayout和onDraw这个过程。
  ```
  final Runnable mAnimate = new Runnable() {
            @Override
            public void run() {
                mAnimationScheduled = false;
                doAnimationFrame(mChoreographer.getFrameTime());
            }
        };
  
  void initAnimation() {
        if (!mInitialized) {
            // mValueType may change due to setter/getter setup; do this before calling super.init(),
            // which uses mValueType to set up the default type evaluator.
            final Object target = getTarget();
            if (target != null) {
                final int numValues = mValues.length;
                for (int i = 0; i < numValues; ++i) {
                    mValues[i].setupSetterAndGetter(target);//会通过反射去查找target的Setter和Getter方法，并存储在HashMap中
                }
            }
            super.initAnimation();
        }
  }
  ```
 在这个过程中会执行到initAnimation()方法，ObjectAnimator重写了此方法，并调用setupSetterAndGetter查找Setter和Getter方法并存储在HashMap中，然后会在ValueAnimator的animationFrame方法中对target属性进行赋值。
 
 所以说补间动画不会修改View的实际位置，只会修改View的显示位置，而属性动画将会对View进行重新测量、布局和绘制，会修改View的实际位置。

#### 19、Android多点触控。


#### 20、ViewPager、RecyclerView和SrcollView嵌套滑动冲突，显示不完整，滑动卡顿问题。


#### 21、滑动卡顿如何解决（不同原因及对应处理方式）。


#### [22、Native和js交互流程。](https://blog.csdn.net/carson_ho/article/details/64904691)

 - Native调用js：
   - 通过WebView的loadUrl()
     - JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。
     - 效率低，会使页面重新刷新，不能获取返回值
   - 通过WebView的evaluateJavascript()
     - 效率高，但只支持4.4以上版本
   
 - js调用Native：
   - 通过WebView的addJavascriptInterface()进行对象映射
     - [在Android4.2以下，存在安全漏洞](https://www.jianshu.com/p/3a345d27cd42)
   - 通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url
     - 操作麻烦，不方便获取返回值。
   - 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt()方法回调拦截JS对话框alert()、confirm()、prompt()消息
     - 使用复杂
  
























   

