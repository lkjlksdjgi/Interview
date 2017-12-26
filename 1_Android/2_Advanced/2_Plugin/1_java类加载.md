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
2、
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
提到ExtClassLoader，这里必须注意一下两个概念的区别：父类加载器和类中继承，父类加载器不像继承，它是没有父子关系的,看一张ClassLoader类的继承关系图<br>
![](https://github.com/tongsiw/Interview/blob/master/picture/java_classloader.png) <br>
可以看到，AppClassLoader的父类是URLClassLoader，但是AppClassLoader的父类加载器是ExtClassLoader，看下面代码输出结果

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

##### 3、BootstrapClassLoader(根类加载器)
看到这里，是不是已经想到BootstrapClassLoader是ExtClassLoader的父类加载器呢。答案是错误的，因为BootstrapClassLoader是用C++实现的，他没有对应的java类，所以BootstrapClassLoader不是ExtClassLoader的父类加载器。而且因为使用C++实现的，所以想要获取它的加载路径，就必须用特殊的手段去获取：通过看AppClassLoader和ExtClassLoader源码(后面会有源码解析)我们知道，他们都是Launcher的内部类，而Launcher提供了一个方法(getBootstrapClassPath)，来获取BootstrapClassLoader的加载路径，所以用什么手段来获取BootstrapClassLoader的加载路径，就呼之欲出了。

```
public class Main {
	

	public static void main(String[] args) {
	        
	        try {
				Class launcherClass = Class.forName("sun.misc.Launcher");
				Method methodGetBootstrapClassPath= launcherClass.getDeclaredMethod("getBootstrapClassPath", null);
				if(methodGetBootstrapClassPath!=null){
					methodGetBootstrapClassPath.setAccessible(true);
					Object obj = methodGetBootstrapClassPath.invoke(null, null);
					if(obj!=null){
						Method methodGetUrls =  obj.getClass().getDeclaredMethod("getURLs", null);
						if(methodGetUrls != null){
							methodGetUrls.setAccessible(true);
							 URL[] bootstrapClassPath = (URL[]) methodGetUrls.invoke(obj, null);
							 for (URL url: bootstrapClassPath) {
						            System.out.print(url);
						            System.out.print("\n");
						        }
						}
					}
				}
				
			} catch (ClassNotFoundException | NoSuchMethodException | SecurityException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
				e.printStackTrace();
			}
	        
	}
}

打印结果
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/resources.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/rt.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/sunrsasign.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/jsse.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/jce.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/charsets.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/lib/jfr.jar
file:/C:/MySoft/Develop_File/jdk1.8/jre/classes

```
上面就是java中三个类加载器的加载路径，下面将会讲解一下类加载器是怎么加载一个类的

#### 二、类加载器是怎么加载一个类的（父委托加载机制）
类加载器主要通过loadClass这个方法去加载一个类的，下面看源码
```
public Class<?> loadClass(String name) throws ClassNotFoundException {
		return loadClass(name, false);
	}
	
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded 判断这个类是否被加载过，如果被加载过，直接返回
            Class<?> c = findLoadedClass(name);
            if (c == null) {//没有被加载过
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//判断类加载器的父类加载器是否为空
                        c = parent.loadClass(name, false);//调用父类加载器的loadClass
                    } else {//父类加载器为空，就调用findBootstrapClass方法
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    
    private Class<?> findBootstrapClassOrNull(String name) {
        if (!checkName(name)) return null;
        return findBootstrapClass(name);
    }

    // return null if not found
    private native Class<?> findBootstrapClass(String name);
	
```
通过上面源码我们可以知道，如果当类加载器是AppClassLoader时，会判断它的父加载器parent是否为空，如果不为空，则调用父类也就是ExtClassLoader的loadClass(name, false)方法，这个时候又会判断一次ExtClassLoader的父类加载器是否为空，如果不为空，则调用ExtClassLoader的父类加载器loadClass方法，当然ExtClassLoader父类加载器是为空的，所以，最终会调用BootstrapClassLoader类加载器去加载一个类，这一套加载机制，叫做 ***父委托加载机制*** <br>
来一张图，方便理解 <br>
![](https://github.com/tongsiw/Interview/blob/master/picture/classLoader_ParentLoading.png) <br><br>

#### 三、父委托加载机制的好处
现在咱们回头看看最开始那个问题，你会发现回报一个错误
```
错误: 在类 java.lang.Long 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application
```
现在应该明白，为什么会报一个这样的错误了把。就是因为父委托类加载机制。那父委托加载机制有什么好处呢？<br>
这样可以提高软件系统的安全性。如果，别人自己写了一个Long类，在Long类里面添加一些恶意代码，所以很有可能会有人利用这个漏洞去攻击你的程序。






