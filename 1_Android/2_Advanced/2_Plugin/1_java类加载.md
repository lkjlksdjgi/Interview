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

#### 一、常见的ClassLoader介绍
classLoader 其实就是类加载器，具体作用就是将类加载到java虚拟机中去，之后类中的程序就可以运行了。java中常见的类加载器主要有以下三个：<br>
***1、BootstrapClassLoader(根类加载器) ：*** BootstrapClassLoader是纯C++实现的类加载器，没有对应的java类，主要加载的是jre/lib/目录下的核心库<br>
***2、ExtClassLoader(扩展类加载器) :*** 类的全名sun.misc.Launcher$ExtClassLoader，主要加载的是jre/lib/ext目录下的扩展包<br>
***3、AppClassLoader(应用类加载器) :*** 类的全名sun.misc.Launcher$AppClassLoader，主要加载CLASSPATH路径下的包<br>

#### 二、下面通过程序，打印出上面3个类加载器的加载路径
##### 1、AppClassLoader(应用类加载器)
```
public class Main {
	
	public static void main(String[] args) {
		Class<Main> mainClass = Main.class;
	        ClassLoader classLoader = mainClass.getClassLoader();
		//打印类加载器名称
	        System.out.print(classLoader.toString());
		//打印AppClassLoader加载路径
	        URL[] urLs = ((URLClassLoader) classLoader).getURLs();
	        for (URL url: urLs) {
	            System.out.print(url);
	            System.out.print("\n");
	        }
	}
}

打印结果
sun.misc.Launcher$AppClassLoader@4e0e2f2a
file:/D:/Study_Space/ClassLoaderDemo/ClassLoaderDemo/bin/

```
从打印结果可看到，AppClassLoader主要加载CLASSPATH路径下类，还有一些第三方包等，也就是加载我们应用程序的类和第三方库<br><br>

##### 2、ExtClassLoader(扩展类加载器)
提到ExtClassLoader，这里必须注意一下两个概念的区别：父类加载器和类中继承，父类加载器不像继承，它是没有父子关系的
```
public class Main {
	

	public static void main(String[] args) {
		
		Class<Main> mainClass = Main.class;
	        ClassLoader classLoader = mainClass.getClassLoader();
		//获取AppClassLoader的父类加载器
	        ClassLoader extClassLoader = classLoader.getParent();
		//打印类加载器名称
	        System.out.print( extClassLoader.toString());
	        System.out.print("\n");
	        URL[] extUrLs = ((URLClassLoader) extClassLoader).getURLs();
		//打印AppClassLoader加载路径
	        for (URL url: extUrLs) {
	            System.out.print(url);
	            System.out.print("\n");
	        }
	  
	}
}

打印结果
sun.misc.Launcher$ExtClassLoader@2a139a55

file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/access-bridge-64.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/cldrdata.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/dnsns.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/jaccess.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/jfxrt.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/localedata.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/nashorn.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/sunec.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/sunjce_provider.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/sunmscapi.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/sunpkcs11.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/ext/zipfs.jar
```









