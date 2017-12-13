#### 一、java集合类简介
先看看图，了解一下层次关系<br>
如图所示：<br>
实线边框的是实现类；
折线边框的是抽象类；
而点线边框的是接口；
虚线三角形箭头是实现；
实线三角形箭头是泛华<br>
![](https://github.com/tongsiw/Interview/blob/master/picture/javacollections.png)

集合大致可以分为Set、List、Queue和Map四种体系。其中Set代表无序、不可重复的集合；List代表有序、重复的集合；而Map则代表具有映射关系的集合；Queue代表一种队列集合。<br><br>
由上图可以看出，Java的集合类主要由两个接口派生而出：Collection和Map。而ArrayList,HashSet,LinkedList,TreeSet,HashMap，TreeMap是我们经常会有用到的已实现的集合类。<br><br>

#### 二、Collection接口
Collection是Set,Queue,List的父接口，接口中定义的方法如下：<br>
```
int size();                                   返回此collection中元素个数
boolean isEmpty();                            如果此collection不包含元素，则返回true
boolean contains(Object o);                   如果此collection包含指定的元素，则返回true
Iterator<E> iterator();                       返回在此collection的元素上进行迭代的迭代器
Object[] toArray();                           返回包含此collection中所有元素的数组
<T> T[] toArray(T[] a);                       返回包含此collection中所有元素的数组；返回数组的运行时类型与指定数组的运行时类型相同
boolean add(E e);                             确保此collection包含指定的元素（可选操作）
boolean remove(Object o);                     从此collection中移除指定元素的单个实例，如果存在的话
boolean containsAll(Collection<?> c);         如果此collection包含指定collection中的所有元素，则返回true
boolean addAll(Collection<? extends E> c);    将指定collection中的所有元素都添加到此collection中（可选操作）
boolean removeAll(Collection<?> c);           移除此collection中那些也包含在指定collection中的所有元素
boolean retainAll(Collection<?> c);           仅保留此collection中那些也包含在指定collection中的所有元素
void clear();                                 移除此collection种的所有操作（可选操作）
boolean equals(Object o);                     比较此collection与指定对象是否相等
int hashCode();                               返回此collection的哈希码值
```
上面的方法相信大家都非常熟悉，下面介绍它的三个子接口
##### 1、List集合(有序，可重复)
List里存放的对象是有序的，同时也是可以重复的，List关注的是索引，拥有一系列和索引相关的方法，查询速度快。<br>
因为往list集合里插入或删除数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。<br>

##### 2、set结合（无序，不可重复）
Set里存放的对象是无序，不能重复的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。

##### 3、Queue集合（队列）
队列是一种数据结构，通常是指 ***先进先出*** 的容器，如果你试图向一个 已经满了的阻塞队列中添加一个元素或者是从一个空的阻塞队列中移除一个元索，将导致线程阻塞．在多线程进行合作时，阻塞队列是很有用的工具。工作者线程可 以定期地把中间结果存到阻塞队列中而其他工作者线线程把中间结果取出并在将来修改它们。队列会自动平衡负载。如果第一个线程集运行得比第二个慢，则第二个 线程集在等待结果时就会阻塞。如果第一个线程集运行得快，那么它将等待第二个线程集赶上来。<br>
下面试jdk1.5中的阻塞队列的操作：<br>
```
add        增加一个元索                 如果队列已满，则抛出一个IIIegaISlabEepeplian异常
remove     移除并返回队列头部的元素      如果队列为空，则抛出一个NoSuchElementException异常
element    返回队列头部的元素           如果队列为空，则抛出一个NoSuchElementException异常
offer      添加一个元素并返回true       如果队列已满，则返回false
poll       移除并返问队列头部的元素      如果队列为空，则返回null
peek       返回队列头部的元素           如果队列为空，则返回null
put        添加一个元素                 如果队列满，则阻塞
take       移除并返回队列头部的元素      如果队列为空，则阻塞
```

#### 三、Map接口
Map：一组成对的“键值对”对象，允许你使用键来查找值，键唯一、值不唯一

#### 四、大致说下Collection和Map的实现类
##### 1、List集合实现类
###### ArrayList LinkedList Vector Stack 区别
***ArrayList：*** 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低。<br>
***LinkedList：*** 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率低。<br>
***Vector：*** 是矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现。但是ArrayList是非线程安全的，而Vector是线程安全的。<br>
***Stack：*** 是栈，它继承于Vector。它的特性是：先进后出(FILO, First In Last Out)。<br>
基于上：<br>
(1) 对于需要快速插入，删除元素，应该使用LinkedList。<br>
(2) 对于需要快速随机访问元素，应该使用ArrayList。<br>
(3) 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类。多线程使用Vector<br>

##### 2、Set集合实现类
###### HashSet TreeSet LinkedHashSet 区别
***HashSet:*** <br>
(1)HashSet中不能有相同的元素，可以有一个Null元素，存入的元素是无序的 <br>
(2)HashSet底层数据结构是哈希表，哈希表就是存储唯一系列的表，而哈希值是由对象的hashCode()方法生成,hashCode()和equals()方法确保HashSet值是唯一的<br>
(3)非线程安全<br>
(4)添加、删除操作时间复杂度都是O(1)。<br><br>
***TreeSet:*** <br>
(1)TreeSet是中不能有相同元素，不可以有Null元素，根据元素的自然顺序进行排序 <br>
(2)TreeSet底层的数据结构是红黑树(一种自平衡二叉查找树),保证元素的排序和唯一性 <br>
(3)非线程安全 <br>
(4)添加、删除操作时间复杂度都是O(log(n)) <br><br>
***LinkedHashSet:*** <br>
(1)LinkedHashSet中不能有相同元素，可以有一个Null元素，元素严格按照放入的顺序排列。<br>
(2)LinkedHashSet底层数据结构由哈希表和链表组成,链表保证了元素的有序即存储和取出一致，哈希表保证了元素的唯一性<br>
(3)非线程安全<br>
(4)添加、删除操作时间复杂度都是O(1)。<br><br>

###### 小结
通过以上特点可以分析出，三者都保证了元素的唯一性，如果无排序要求可以选用HashSet；如果想取出元素的顺序和放入元素的顺序相同，那么可以选用LinkedHashSet。如果想插入、删除立即排序或者按照一定规则排序可以选用TreeSet



