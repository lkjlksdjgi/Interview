# 一、Activity生命周期
先看看activity生命周期图<br>
![](https://github.com/tongsiw/Interview/blob/master/picture/ActivityLife%20Cycle.png) <br>
图中详细给出了Activity整个生命周期的过程，以及在不同的状态期间相应的回调方法。一个最简单的完整的Activity生命周期会按照如下顺序回调:<br>
onCreate(创建) ---> onStart(可见，但不可交互) ---> onResume(可见可交互) --->onPause(不可交互) --->onStop(不可见) --->onDestory(销毁)
<br><br>
- 在实际应用场景中，假设A Activity位于栈顶，此时用户操作，从A Activity跳转到B Activity。那么对AB来说，具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？<br>
   当用户点击A中按钮来到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop。<br>
 <br>
 
 - 其实在实际应用场景中，有一种非常常见的，就是横竖屏切换，此时的Activity生命周期是什么样的呢？<br>
  当横竖屏切换的时候，我们会发现Activity生命周期方法依次是：onPause --> ***onSaveInstanceState*** --> onStop --> onDestory -->onCreate --> onStart -->***onRestoreInstanceState*** -->onResume。可以发现，当横竖屏的时候Activity是先销毁在创建的。Activity因横竖屏而异常销毁，Android系统会自动调用onSaveInstanceState来保存Activity当前状态信息，所以我们可以在onSaveInstanceState方法里面保存一些信息以便Activity重建之后恢复数据。Activity被重新创建之后，系统还会去调用onRestoreInstanceState方法，当然通常情况下我们会根据判断onCreate方法里面bundle有没有数据来做相应的初始化。<br>
onSaveInstanceState和onRestoreInstanceState只有在Activity异常终止时才会被调用的，正常情况是不会调用这两个方法的<br>
<br>

- 上面说到因横竖屏而导致Activity异常销毁然后又重建，那有没有办法使其不重建呢？答案是有的。<br>
      我们可以给Activity指定configChange属性，当我们不想Activity在屏幕旋转后导致销毁重建时，可以设置configChange=“orientation”；当SDK版本大于13时，我们还需额外添加一个“screenSize”的值.<br>
    对于这两个值含义如下：<br>
    - orientation :屏幕方向发生变化，配置该参数可以解决横竖屏切换时，Activity重建问题（API<13） <br>
    - screenSize :当设备旋转时，屏幕尺寸发生变化，API>13后必须配置该参数才可以保证横竖切换不会导致Activity重建。 <br>
    
    
－内存不足，导致优先级低的Activity被销毁<br>
Activity优先级是与Activity运行状态一一对应的<br>
1. 前台Activity---正在和用户交互的，优先级最高<br>
2. 可见的非前台Activity --- 比如说 弹出一个对话框，Activity可见但不可交互<br>
3. 后台Activity --- 执行了onStop方法之后的Activity<br>
当系统内存不足时，会先销毁优先级低的Activity。这个时候的Activity销毁和创建的生命周期与上面横竖屏一样。
     




