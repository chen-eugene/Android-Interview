#### 1、Touch事件的分发流程。

[ViewGroup事件传递](https://blog.csdn.net/yanbober/article/details/45912661)  
[View事件传递](https://blog.csdn.net/yanbober/article/details/45887547)

Touch事件的分发过程有三个重要的方法来完成：dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent。 

**分发流程：**
- 对于跟ViewGroup来说，Touch事件产生后首先回传递给它的dispatchTouchEvent方法，如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示它要拦截当前事件，它的onTouchEvent会被调用去处理此事件；如果这个ViewGroup的onInterceptTouchEvent方法返回false就表示它不拦截当前事件，此时事件就会传递给它的子视图，子视图的dispatchTouchEvent方法会被调用，如此反复直到事件被最终处理。

-  当一个View需要处理事件时，如果它设置了OnTouchListener，那么OnTouchListener中的onTouch方法会被调用，如果onTouch返回false，则当前View的onTouchEvent方法会被调用；如果返回true，则onTouchEvent将不会被调用，表示当前View消耗掉了此事件。

- 在整个事件传递过程中，如果一个View的onTouchEvent返回false，那么他的父视图的onTouchEvent方法将会被调用，以此类推。如果所有的View都不处理当前事件，那么此事件最终将会被传递给Activity处理，即Activity的onTouchEvent方法会被调用。

**注意点：**
- View没有onInterceptTouchEvent方法，一点有事件传递给它，那么它的onTouchEvent方法将会被调用。

- 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件(onTouchEvent返回false)，那么它将不会接收到同一个事件序列的后续事件，并且事件将会重新交给他的父视图处理，即父视图的onTouchEvent方法会被调用。

- 如果View不消耗除ACTION_DOWN意外的其他事件，此父视图的onTouchEvent方法不会被调用，但是当前的View可以持续接收到后续的事件，最终事件会被传递给Activity处理。

- dispatchTouchEvent回true就是消费事件，这种说法不完全正确。dispatchTouchEvent事件派发是传递的，如果返回值为false将停止下次事件派发，如果返回true将继续下次派发。

#### 2、View的位置参数有哪些，left、x、translationX的含义以及三者的关系。
- view的位置由left、top、right、bottom四个属性决定，这几个坐标可以通过getLeft()、getTop()、getRight()、getBottom()获取。注意这四个坐标是相对坐标，即相对于父容器的坐标。当view发生移动时，这几个坐标是不变的。

- 从Android 3.0开始，增加了几个参数：x、y、translationX、translationY，都是相对于父容器的坐标。

   x指view左上角的横坐标，当view发生移动时，x会变化；

   translationX指view左上角的横坐标相对于父容器的偏移量，当view发生移动时，translationX会变化。    
   `x = left + translationX `

- rawX是绝对坐标，是相对于屏幕左上角的横坐标，view本身没有getRawX的方法，这个方法一般在MotionEvent对象里使用。

- scrollX指的是view在滑动过程中，view的左边缘和view内容的左边缘在水平方向的距离（注意与translationX 的区别，translationX 指的是view本身的移动，scrollX是view的内容移动），也就是说调用了view的scrollTo或scrollBy方法，view本身不会移动，只会移动view的内容。    
![坐标图](https://github.com/chen-eugene/Interview/blob/master/image/20160808154319878.png)

#### 3、什么是MeasureSpec。
MeasureSpec是一个32位的int值，高2位表示SpecMode，指测量模式；低30位表示SpecSize，指在某种测量模式下的规格大小。MeasureSpec一旦确定后，onMeasure方法就可以确定View的测量宽/高。

- 对于DecorView，其MeasureSpec由窗口的尺寸和其资深的LayoutParams来共同决定。
- 对于普通的View，其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams共同决定。  

![MeasureSpec](https://github.com/chen-eugene/Interview/blob/master/image/20170311114110621.jpg)


#### 3、自定义View的流程，自定义View需要注意的问题，例如自定义View是否需要重写onLayout，onMeasure。

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

