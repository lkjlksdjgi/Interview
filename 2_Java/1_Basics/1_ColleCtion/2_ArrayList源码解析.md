#### 一、简介
ArrayList内部是用***数组***存储数据的，是一个可以动态增长的顺序表。因为其底层是用***数组***实现的，所以他拥有数组的缺点，空间效率不高，但是由于数组的内存连续，可以根据下标以O1的时间读写(改查)元素，所以他的时间效率很高 <br><br>

#### 二、源码解析
下面主要从ArrayList的 构造方法、增删改查、迭代器这几个方面进行讲解

##### 1、构造方法
```

    //空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    
    //无参构造函数里面默认的空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    //这个数组是ArrayList真正存放元素的地方
    transient Object[] elementData; // non-private to simplify nested class access
    
    //elementData数组存放数据的个数
    private int size;

   //默认的构造方法
    public ArrayList() {
        //这里将空数组赋值给elementData数组
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
     
    //带初始容量的构造方法 
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {//如果初始容量大于0，就新建一个长度为initialCapacity的Object数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//如果容量为0，直接将EMPTY_ELEMENTDATA赋值给elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//如果容量小于0，直接抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }
    
    //通过别的集合来创建ArrayList的构造方法
    public ArrayList(Collection<? extends E> c) {
        //利用Collection.toArray()方法得到一个对象数组，然后赋值给elementData 
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {//重新设置size的值，size的值为当前elementData数组的长度
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            //c.toArray可能没有返回Object[]数组，这个时候利用Arrays.copyOf来复制集合c中的元素到elementData数组中
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            //如果数组elementData长度为0，将elementData数组替换成 EMPTY_ELEMENTDATA 数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
    
```
当我们初始化完一个ArrayList的时候，就会创建一个elementData数组以及初始化elementData数组的长度size <br><br>

想一想，elementData数组为什么是Object类型的，而不是 ***E*** 类型的？因为如果是泛型的话，那么新建一个数组的时候，就必须用反射相关API来初始化它。<br><br>

##### 2、添加
添加主要的方法有
```
boolean add(E e)                                                  在数组尾部添加元素                时间复杂度 O(1)
void add(int index, E element)                                    在数组index位置插入元素           时间复杂度 O(N)
boolean addAll(Collection<? extends E> c)                         在数组尾部添加一个集合            时间复杂度 O(1)
boolean addAll(int index, Collection<? extends E> c)              在数组index位置添加集合           时间复杂度 O(N)  
```
当然添加的方法里面还涉及到两个重要的方法，一个是越界判断，一个是判断是否需要扩容，方法如下
```
void rangeCheckForAdd(int index) //判断是否越界
void ensureCapacityInternal(int minCapacity) //判断是否需要扩容              时间复杂度 O(N)
```
<br>
接下来看看上面几个方法的详细内容

```

    //默认扩容容量
    private static final int DEFAULT_CAPACITY = 10;
    
    public boolean add(E e) {
        //判断是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //在数组末尾追加一个元素，并将size加1
        elementData[size++] = e;
        return true;
    }
    
    public void add(int index, E element) {
        //判断是否越界
        rangeCheckForAdd(index);
        //判断是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //将index开始的数据 向后移动一位
        System.arraycopy(elementData, index, elementData, index + 1,size - index);
        //
        elementData[index] = element;
        size++;
    }
    
    
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //复制数组
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0){
        //移动（复制)数组
          System.arraycopy(elementData, index, elementData, index + numNew, numMoved);
        )
        //复制数组完成批量赋值
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    //越界判断 如果越界抛异常
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    //判断是否需要扩容
    private void ensureCapacityInternal(int minCapacity) {
        //如果elementData数组是通过默认的构造方法进行初始化的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //取DEFAULT_CAPACITY和minCapacity中的最大值并赋值给minCapacity
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
       //modify count的简写，当ArrayList的结构发生修改的时候，modCount自动加 1 ，主要有添加、删除、清空、扩容四种操作
        modCount++;

        // overflow-conscious code
        //判断是否需要扩容，当数组最小容量大于elementData数组长度的时候，则需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    //数组扩容
    private void grow(int minCapacity) {
        // overflow-conscious code
        //当前数组长度
        int oldCapacity = elementData.length;
        //默认扩容一半
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        
        if (newCapacity - minCapacity < 0)//如果扩容一半还不够 ，那么就用数组能容纳的最小的数量。
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //扩充数组到指定长度
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    //这个不解释了
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    
```


看完上面源码，我们发现每个add方法都会进行一个是否需要扩容的判断，而越界判断主要就是判断那个index是否大于数组的长度<br><br>


##### 3、删除 和 清空
```
    public E remove(int index) {
        //判断是否越界
        rangeCheck(index);

       //modify count的简写，当ArrayList的结构发生修改的时候，modCount自动加 1 ，主要有添加、删除、清空、扩容四种操
        modCount++;
        //读出要删除的值
        E oldValue = elementData(index);
        
        //需要移动的元素个数
        int numMoved = size - index - 1;
       
        if (numMoved > 0)
        //从指定的源数组中，从指定位置复制数组到目标数组的指定位置。
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //置空数组尾部数据
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    
    //删除数组中的某一个元素
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    //根据index删除元素
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                 //根据index删除元素
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    
    //根据index删除元素
    private void fastRemove(int index) {
        modCount++;
        //需要移动的元素个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
        //从指定的源数组中，从指定位置复制数组到目标数组的指定位置。
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }


  public boolean removeAll(Collection<?> c) {
        //判断是否为空
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }

//批量移动
 private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;//w 代表批量删除后 数组还剩多少元素
        boolean modified = false;
        try {
          //这里主要就是将elementData数组中包含的C数组元素全部删除，非常高效
           for (; r < size; r++)
            // 如果 c里不包含当前下标元素，则保留
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            //TODO 
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
    
    //清空
    public void clear() {
        //modify count的简写，当ArrayList的结构发生修改的时候，modCount自动加 1 ，主要有添加、删除、清空、扩容四种操作
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
           //将数组所有元素置空
            elementData[i] = null;
       //将数组长度置为0
        size = 0;
    }
    
```
##### 4、改
```
   //时间复杂度为O(1)
   public E set(int index, E element) {
        //越界检查
        rangeCheck(index);
        //取出元素 
        E oldValue = elementData(index);
        //覆盖元素
        elementData[index] = element;
        return oldValue;
    }
```
##### 5、查
```
     //获取指定角标的值，时间复杂度为O(1)
     public E get(int index) {
        //越界检查
        rangeCheck(index);
        //取出元素 
        return elementData(index);
    }
    
    //获取数组对象的位置  时间复杂度为O(N)
    
    public int indexOf(Object o) {
      //这个没啥说的
       if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

##### 6、迭代器
```
   public Iterator<E> iterator() {
        return new Itr();
    }
    
    private class Itr implements Iterator<E> {
        
        //游标，默认是0
        int cursor;       // index of next element to return
        
        //上一次返回的元素 (删除的标志位)
        int lastRet = -1; // index of last element returned; -1 if no such
        
        //用于判断集合是否修改过结构的 标志
        int expectedModCount = modCount;

        //如果游标指向数组最后一位，返回false，反之返回true
        public boolean hasNext() {
            return cursor != size;
        }

      public E next() {
            //判断是否修改过   
            checkForComodification();
            int i = cursor;
            //当游标大于等于数组长度时，抛出异常
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            //这个异常抛出的原因就是，在迭代的时候，对数组进行操作了
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];//返回元素 ，并设置上一次返回的元素的下标
        }
        

      //remove 掉 上一次next的元素
      public void remove() {
            //先判断是否next过
            if (lastRet < 0)
                throw new IllegalStateException();
             //判断是否修改过   
            checkForComodification();

            try {
                //删除元素 remove方法内会修改 modCount 所以后面要更新Iterator里的这个标志值
                ArrayList.this.remove(lastRet);
                //要删除的游标
                cursor = lastRet;
                //不能重复删除 所以修改删除的标志位
                lastRet = -1;
                
                //更新 判断集合是否修改的标志，
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        //判断是否修改过了List的结构，如果有修改，抛出异常
         final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

#### 总结
在面试中还有可能会问到和Vector的区别，看Vector的源码，发现内部也是数组做的，区别在于Vector在API上都加了synchronized所以它是线程安全的，以及Vector扩容时，是翻倍size，而ArrayList是扩容50%。





