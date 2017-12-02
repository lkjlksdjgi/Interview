# 一、Activity生命周期
先看看activity生命周期图<br>
![](https://github.com/tongsiw/Interview/blob/master/picture/ActivityLife%20Cycle.png) <br>
图中详细给出了Activity整个生命周期的过程，以及在不同的状态期间相应的回调方法。一个最简单的完整的Activity生命周期会按照如下顺序回调:<br>
onCreate(创建) ---> onStart(可见，但不可交互) ---> onResume(可见可交互) --->onPause(不可交互) --->onStop(不可见) --->onDestory(销毁)
<br><br>
1. 在实际应用场景中，假设A Activity位于栈顶，此时用户操作，从A Activity跳转到B Activity。那么对AB来说，具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？<br>
   当用户点击A中按钮来到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop。<br>
 
2. 其实在实际应用场景中，有一种非常常见的，就是横竖屏切换，此时的Activity生命周期是什么样的呢？<br>
  当横竖屏切换的时候，我们会发现Activity生命周期方法依次是：onPause --> ***onSaveInstanceState*** --> onStop --> onDestory -->onCreate --> onStart -->***onRestoreInstanceState*** -->onResume。可以发现，当横竖屏的时候Activity是先销毁在创建的。Activity因横竖屏而异常销毁，Android系统会自动调用onSaveInstanceState来保存Activity当前状态信息，所以我们可以在onSaveInstanceState方法里面保存一些信息以便Activity重建之后恢复数据。Activity被重新创建之后，系统还会去调用onRestoreInstanceState方法，当然通常情况下我们会根据判断onCreate方法里面bundle有没有数据来做相应的初始化。<br>
onSaveInstanceState和onRestoreInstanceState只有在Activity异常终止时才会被调用的，正常情况是不会调用这两个方法的<br>

3. 上面说到因横竖屏而导致Activity异常销毁然后又重建，那有没有办法使其不重建呢？答案是有的。<br>
      我们可以给Activity指定configChange属性，当我们不想Activity在屏幕旋转后导致销毁重建时，可以设置configChange=“orientation”；当SDK版本大于13时，我们还需额外添加一个“screenSize”的值.<br>
    对于这两个值含义如下：<br>
   (1) orientation :屏幕方向发生变化，配置该参数可以解决横竖屏切换时，Activity重建问题（API<13） <br>
    (2) screenSize :当设备旋转时，屏幕尺寸发生变化，API>13后必须配置该参数才可以保证横竖切换不会导致Activity重建。 <br>
 
4. 内存不足，导致优先级低的Activity被销毁<br>
Activity优先级是与Activity运行状态一一对应的<br>
(1). 前台Activity---正在和用户交互的，优先级最高<br>
(2). 可见的非前台Activity --- 比如说 弹出一个对话框，Activity可见但不可交互<br>
(3). 后台Activity --- 执行了onStop方法之后的Activity<br>
当系统内存不足时，会先销毁优先级低的Activity。这个时候的Activity销毁和创建的生命周期与上面横竖屏一样。
<br>
<br>

# 二、Activity启动模式
#### 1、Activity4种启动模式
标准模式（standard）<br>
栈顶复用模式（singleTop）<br>
栈内复用模式（singleTask）<br>
单例模式（singleInstance）<br>
Activity的管理是采用任务栈的形式，任务栈采用 ***“后进先出”*** 的栈结构。
<br>
#### 2、Activity4种启动模式详解
##### (1)标准模式（standard）
标准模式是默认的启动模式，也就是在清单文件中不申明launchMode时，默认这个模式。每次启动一个Activity都会重写创建一个新的实例，不管这个实例存不存在。这种模式下，谁启动了该模式的Activity，该Activity就属于启动它的Activity的任务栈中<br>
在清单文件中配置，也可以不配置，默认标准模式<br>
```<activity android:name=".StandardActivity" android:launchMode="standard" > ```
<br>
如果在Service等启动一个Activity，其并没有所谓的任务栈，所以需要添加一个Flag标记：FLAG_ACTIVITY_NEW_TASK <br>
Activity的Flag很多，主要用于设定Activity的启动模式，可以在启动Activity时，通过Intent的addFlags()方法设置。<br>
Activity常用的Flag：<br>
a). FLAG_ACTIVITY_NEW_TASK:其效果与指定Activity为singleTask模式一致。<br>
b). FLAG_ACTIVITY_SINGLE_TOP :其效果与指定Activity为singleTop模式一致。<br>
c). FLAG_ACTIVITY_CLEAR_TOP: 具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。如果和singleTask模式一起出现，若被启动的Activity已经存在栈中，则清除其之上的Activity，并调用该Activity的onNewIntent方法。如果被启动的Activity采用standard模式，那么该Activity连同之上的所有Activity出栈，然后创建新的Activity实例并压入栈中。<br>

##### (2)栈顶复用模式（singleTop）
这个模式下，如果新的activity已经位于栈顶，那么这个Activity不会被重写创建，同时它的***onNewIntent***方法会被调用，其他不会被调用。如果栈顶不存在该Activity的实例，则情况与standard模式相同。<br>
在清单文件中配置，也可以不配置，默认标准模式<br>
```activity android:name=".singletop.SingleTopActivity" android:launchMode="singleTop">``` <br>
应用场景：在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop <br>

##### (3)栈内复用模式（singleTask）
这个模式下，如果栈中存在这个Activity的实例就会复用这个Activity，不管它是否位于栈顶，复用时，会将它上面的Activity全部出栈，并且会回调该实例的onNewIntent方法。<br>
其实这个过程还存在一个任务栈的匹配，如下：<br>
当singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈<br>
(1)如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去.
(2)如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例。 如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法。如果不存在该实例，则新建Activity，并入栈。<br>
此外，我们可以将两个不同App中的Activity设置为相同的taskAffinity，这样虽然在不同的应用中，但是Activity会被分配到同一个Task中去。<br>
配置如下：
```<activity android:name=".MainActivity" android:launchMode="singleTask" android:taskAffinity="com.demo.singletask"/>```

##### (4)单例模式（singleInstance）
这种模式下的Activity会单独占用一个Task栈，具有全局唯一性，其实就是SingleTask的加强版，即整个系统中就这么一个实例。因为栈内复用的特性，所以会直接复用不会创建新的Activity实例，除非这个特殊的任务栈被销毁了。<br>
应用场景：呼叫来电界面

   









