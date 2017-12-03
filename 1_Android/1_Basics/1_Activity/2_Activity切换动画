动画，用的最多的应该就是属性动画和补间动画了，通常我们要实现Activity之间的切换动画，都会这么写<br>
```
Intent intent=new Intent(this,SecondActivity.class);
startActivity(intent);
//这是官方的动画效果，当然你也可以在res/anim 下  声明两个动画效果
overridePendingTransition(android.R.anim.fade_in,android.R.anim.fade_out);
```
动画就是通过overridePendingTransition这个函数来实现的，兼容Android的2.0版本之后，关于这个函数的用法必须注意：<br>
***它必需紧挨着startActivity()或者finish()函数之后调用***
