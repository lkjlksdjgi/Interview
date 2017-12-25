在介绍ClassLoader之前，先提几个问题：
1、新建一个java工程，创建一个Long类，在里面写上如下代码
```
package java.lang;
public class Long {
	public static void main(String[] args) {
		System.out.print("long");
	}
}

```
运行，会发生什么呢？

#### 常见的ClassLoader介绍
classLoader 其实就是类加载器，具体作用就是将类加载到java虚拟机中去，之后类中的程序就可以运行了。java中常见的类加载器主要有以下三个：<br>
***1、BootstrapClassLoader ：*** BootstrapClassLoader是纯C++实现的类加载器，没有对应的java类，主要加载的是jre/lib/目录下的核心库<br>
***2、ExtClassLoader :*** 类的全名sun.misc.Launcher$ExtClassLoader，主要加载的是jre/lib/ext目录下的扩展包<br>
***3、AppClassLoader :*** 类的全名sun.misc.Launcher$AppClassLoader，主要加载CLASSPATH路径下的包<br>
