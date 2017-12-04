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
下面讲解详细的实现步骤
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

