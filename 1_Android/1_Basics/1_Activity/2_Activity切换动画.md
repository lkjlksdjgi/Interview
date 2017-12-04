动画，用的最多的应该就是属性动画和补间动画了，通常我们要实现Activity之间的切换动画，都会这么写<br>
```
Intent intent=new Intent(this,OtherActivity.class);
startActivity(intent);
//这是官方的动画效果，当然你也可以在res/anim 下  声明两个动画效果
overridePendingTransition(android.R.anim.fade_in,android.R.anim.fade_out);
```
动画就是通过overridePendingTransition这个函数来实现的，兼容Android的2.0版本之后，关于这个函数的用法必须注意：<br>
***它必需紧挨着startActivity()或者finish()函数之后调用***
<br>
在Api21（5.0）以后，我们也可以使用内置的Activity切换动画，效果很炫，当然只能兼容到API21以后<br>
Activity切换动画期间，就进入或退出两种情况，而进入分为***第一次进入Activity***和***返回当前Activity***；另外还有一种就是Activity在切换的时候，两个Activity中的View具有过度切换的效果，即共享元素<br>
```
enter：用于决定第一次打开当前Activity时的动画 
exit : 用于决定退出当前Activity时的动画 
reenter: 用于决定如果当前Activity已经打开过，并且再次打开该Activity时的动画 
shared elements:用于决定在两个Activity之间切换时，指定两个Activity中对应的View的过渡效果
```
#### 一、详细的实现步骤
##### 1、在setContentView之前，告诉Window页面切换需要使用动画。代码如下
```
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
```
##### 2、加载切换动画
```
Transition explode = TransitionInflater.from(this).inflateTransition(R.transition.explode);
```
R.transition.explode就是要执行的动画,是在res/transition目录下的xml文件explode文件代码如下：
```
<explode xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"/>
```
上面的动画是Android系统自带的，可以用下面这种写法,就不用写xml文件了：<br>
```
Transition explode = TransitionInflater.from(this).inflateTransition(android.R.transition.explode);
```
##### 3、设置动画相关属性,两种方式，一种是代码，另一种是主题
###### 代码设置
```
//退出时使用
getWindow().setExitTransition(explode);
//第一次进入时使用
getWindow().setEnterTransition(explode);
//再次进入时使用
getWindow().setReenterTransition(explode);
```
###### 主题设置
```
<item name="android:windowExitTransition">@transition/explode</item>
<item name="android:windowEnterAnimation">@transition/explode</item>
<item name="android:windowReenterTransition">@transition/explode</item>
```
##### 4、调用startActivity
```
startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
```
***注意：***这里必须是两个参数的

#### 二、几种动画效果
##### 1、explode
Explode即爆炸效果,在res/transition目录下新建一个xml文件，如explode.xml文件，代码如下：<br>
```
<explode xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"/>
```
##### 2、slide
slide即滑动效果，在res/transition目录下新建一个xml文件，如slide.xml文件，代码如下：<br>
```
<slide xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:interpolator/decelerate_cubic"
    android:slideEdge="end" />
```
##### 3、fade
fade即淡化效果，在res/transition目录下新建一个xml文件，如fade.xml文件，代码如下：<br>
```
<fade xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300" />
```
##### 4、shared element
shared element即共享元素效果，共享元素效果是将前面一个Activity的某个子View与后面一个Activity的某个子View之间有过渡效果<br>
实现步骤：<br>
1、在两个Activity中的需要实现共享元素效果的View，添加一个属性：
```
android:transitionName="shareelement"
```
属性值必须一样。<br>
2、调用startActivity
```
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, firstShareelementView, "shareelement").toBundle());
```
相关参数解释一下：<br>
firstShareelementView：这个参数是第一个Activity中需要实现共享元素效果View的id；<br>
"shareelement" ： 这个是Activity布局文件中需要实现共享元素效果View的transitionName的值，必须一样。<br>
<br>
上面是两个Activity中只有一个View需要实现共享元素，如果是两个以上该如何实现呢？直接贴源码<br>
***第一个Activit的xml布局文件***<br>
```
 <?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.siw.activityanimatedemo.OneActivity">

    <Button
        android:id="@+id/firstShareelementView"
        android:gravity="center"
        android:text="One"
        android:transitionName="firstShareelementView"
        android:background="@color/colorPrimaryDark"
        android:layout_width="match_parent"
        android:layout_height="120dp" />
    <Button
        android:id="@+id/twoShareelementView"
        android:gravity="center"
        android:text="One"
        android:background="@color/colorAccent"
        android:layout_alignParentBottom="true"
        android:transitionName="twoShareelementView"
        android:layout_width="match_parent"
        android:layout_height="120dp" />
</RelativeLayout>

```
***第二个Activit的xml布局文件***<br>
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.siw.activityanimatedemo.TwoActivity">

    <Button
        android:gravity="center"
        android:text="two"
        android:background="@color/colorAccent"
        android:layout_width="match_parent"
        android:transitionName="twoShareelementView"
        android:layout_height="120dp" />

    <Button
        android:text="two"
        android:transitionName="firstShareelementView"
        android:background="@color/colorPrimaryDark"
        android:layout_alignParentBottom="true"
        android:layout_width="match_parent"
        android:layout_height="120dp" />
</RelativeLayout>

```
***第一个Activit的java代码***<br>
```
        Button firstShareelementView = (Button) findViewById(R.id.firstShareelementView);
        Button twoShareelementView = (Button) findViewById(R.id.twoShareelementView);
        Intent intent = new Intent(this, TwoActivity.class);
        //第一个参数是view的id，第二个参数是transitionName的值
        Pair first = new Pair<>(firstShareelementView, "firstShareelementView");
        Pair second = new Pair<>(twoShareelementView, "twoShareelementView");
        ActivityOptionsCompat transitionActivityOptions = ActivityOptionsCompat.makeSceneTransitionAnimation(this, first, second);
        ActivityCompat.startActivity(this,intent, transitionActivityOptions.toBundle());
```
##### 5、自定义shared element动画效果
***具体步骤如下：***<br>
1、定义好transitionName的值：android:transitionName="customshareelement"<br>
2、创建一个移动轨迹路径类PathMotion类,这个是在第二个Activity中完成的<br>
我们可以创建ArcMotion对象，ArcMotion是PathMotion子类，是个曲线路径。<br>
[点击进入ArcMotion文档](https://developer.android.com/reference/android/transition/ArcMotion.html) <br>
```
        //定义ArcMotion
        ArcMotion arcMotion = new ArcMotion();
        arcMotion.setMinimumHorizontalAngle(50f);
        arcMotion.setMinimumVerticalAngle(50f);
```
3、创建一个插值器控制速度<br>
```
Interpolator interpolator = AnimationUtils.loadInterpolator(this, android.R.interpolator.anticipate_overshoot);
```
4、实例化ChangeBounds，并设定相关属性
```
        ChangeBounds changeBounds = new ChangeBounds();

        changeBounds.setPathMotion(arcMotion);
        changeBounds.setInterpolator(interpolator);
        changeBounds.setDuration(300);
        changeBounds.addTarget(custom);
```
5、将切换动画应用到当前的Activity的进入和返回
```
        getWindow().setSharedElementEnterTransition(changeBounds);
        getWindow().setSharedElementReturnTransition(changeBounds);
```
6、从第二个页面返回第一个页面，可以用这个方法
```
 finishAfterTransition();
```
[需要源码，狂点这里](http://download.csdn.net/download/tongsiw/10144064)
