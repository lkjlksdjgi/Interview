# 一、Activity生命周期
先看看activity生命周期图<br>
![](https://github.com/tongsiw/Interview/blob/master/picture/ActivityLife%20Cycle.png) <br>
图中详细给出了Activity整个生命周期的过程，以及在不同的状态期间相应的回调方法。一个最简单的完整的Activity生命周期会按照如下顺序回调:<br>
onCreate(创建) ---> onStart(可见，但不可交互) ---> onResume(可见可交互) --->onPause(不可交互) --->onStop(不可见) --->onDestory(销毁)
<br><br>
    在实际应用场景中，假设A Activity位于栈顶，此时用户操作，从A Activity跳转到B Activity。那么对AB来说，具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？<br>
    当用户点击A中按钮来到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop。



# 二、Activit启动模式
# 三、Activit切换动画
