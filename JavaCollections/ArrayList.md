## ArrayList源码分析

### 简介
----

ArrayList是一种以数组实现的List，与数组相比，它具有动态扩展的能力，因此也可称之为动态数组。
### 继承体系
----

![ArrayList继承体系](http://www.topjava.cn/images/group/sike-java/sike-java-jihe/202105091520338861.png)
- ArrayList实现了List, RandomAccess, Cloneable, java.io.Serializable等接口。

- ArrayList实现了List，提供了基础的添加、删除、遍历等操作。

- ArrayList实现了RandomAccess，提供了随机访问的能力。

- ArrayList实现了Cloneable，可以被克隆。

- ArrayList实现了Serializable，可以被序列化。
### 源码解析
----
#### 变量
```java
     /**
      * 默认容量10
      */
     private static final int DEFAULT_CAPACITY = 10;
 
     /**
      * new ArrayList(0);时使用
      */
     private static final Object[] EMPTY_ELEMENTDATA = {};
 
     /**
      * 用于空构造时使用，并在第一次添加元素后初始化容量到默认值
      */
     private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
 
     /**
      * 存放元素的数组
      */
     transient Object[] elementData; 
 
     /**
      * 集合的大小
      */
     private int size;
```
#### 构造方法
---
ArrayList(int initialCapacity)构造方法
```java
   public ArrayList(int initialCapacity) {
       if (initialCapacity > 0) {
           // 如果传入的初始容量大于0，就新建一个数组存储元素
           this.elementData = new Object[initialCapacity];
       } else if (initialCapacity == 0) {
           // 如果传入的初始容量等于0，使用空数组EMPTY_ELEMENTDATA
           this.elementData = EMPTY_ELEMENTDATA;
       } else {
           // 如果传入的初始容量小于0，抛出异常
           throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
       }
   }
```

ArrayList()构造方法
```java
   public ArrayList() {
       // 如果没有传入初始容量，则使用空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA
       // 使用这个数组是在添加第一个元素的时候会扩容到默认大小10
       this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
   }
```
ArrayList(Collection<? extends E> c)构造方法
```java
    /**
    * 把传入集合的元素初始化到ArrayList中
    */
    public ArrayList(Collection<? extends E> c) {
        // 集合转数组
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 检查c.toArray()返回的是不是Object[]类型，如果不是，重新拷贝成Object[].class类型
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 如果c的空集合，则初始化为空数组EMPTY_ELEMENTDATA
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
add(E e) 方法 时间复杂度O(1)
```java
    public boolean add(E e) {
        // 检查是否需要扩容
        ensureCapacityInternal(size + 1);
        // 把元素插入到最后一位
        elementData[size++] = e;
        return true;
    }
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 如果是空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，就初始化为默认大小10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            // 扩容
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 新容量为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 如果新容量发现比需要的容量还小，则以需要的容量为准
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果新容量已经超过最大容量了，则使用最大容量
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 以新容量拷贝出来一个新数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
add(int index, E element)方法

添加元素到指定位置，时间复杂度O(n)
```java
    public void add(int index, E element) {
        // 检查是否越界
        rangeCheckForAdd(index);
        // 检查是否需要扩容
        ensureCapacityInternal(size + 1);
        // 将inex及其之后的元素往后挪一位，则index位置处就空出来了
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 将元素插入到index的位置
        elementData[index] = element;
        // 大小增1
        size++;
    }
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```
addAll(Collection<? extends E> c) 方法

```java
    /**
    * 将集合c中所有元素添加到当前ArrayList中
    */
    public boolean addAll(Collection<? extends E> c) {
        // 将集合c转为数组
        Object[] a = c.toArray();
        int numNew = a.length;
        // 检查是否需要扩容
        ensureCapacityInternal(size + numNew);
        // 将c中元素全部拷贝到数组的最后
        System.arraycopy(a, 0, elementData, size, numNew);
        // 大小增加c的大小
        size += numNew;
        // 如果c不为空就返回true，否则返回false
        return numNew != 0;
    }
```

get(int index)方法
```java
    public E get(int index) {
        // 检查是否越界
        rangeCheck(index);
        // 返回数组index位置的元素
        return elementData(index);
    }
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    E elementData(int index) {
        return (E) elementData[index];
    }
```

remove(int index)方法
```java
public E remove(int index) {
    // 检查是否越界
    rangeCheck(index);
    modCount++;
    // 获取index位置的元素
    E oldValue = elementData(index);
    // 如果index不是最后一位，则将index之后的元素往前挪一位
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    // 将最后一个元素删除，帮助GC
    elementData[--size] = null; // clear to let GC do its work
    // 返回旧值
    return oldValue;
}
```
remove(Object o)方法
```java
    public boolean remove(Object o) {
        if (o == null) {
            // 遍历整个数组，找到元素第一次出现的位置，并将其快速删除
            for (int index = 0; index < size; index++)
                // 如果要删除的元素为null，则以null进行比较，使用==
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            // 遍历整个数组，找到元素第一次出现的位置，并将其快速删除
            for (int index = 0; index < size; index++)
                // 如果要删除的元素不为null，则进行比较，使用equals()方法
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    private void fastRemove(int index) {
        // 少了一个越界的检查
        modCount++;
        // 如果index不是最后一位，则将index之后的元素往前挪一位
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        // 将最后一个元素删除，帮助GC
        elementData[--size] = null; // clear to let GC do its work
    }
```
retainAll(Collection<?> c)方法

求两个集合的交集
```java
    public boolean retainAll(Collection<?> c) {
        // 集合c不能为null
        Objects.requireNonNull(c);
        // 调用批量删除方法，这时complement传入true，表示删除不包含在c中的元素
        return batchRemove(c, true);
    }
    /**
    * 批量删除元素
    * complement为true表示删除c中不包含的元素
    * complement为false表示删除c中包含的元素
    */
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        // 使用读写两个指针同时遍历数组
        // 读指针每次自增1，写指针放入元素的时候才加1
        // 这样不需要额外的空间，只需要在原有的数组上操作就可以了
        int r = 0, w = 0;
        boolean modified = false;
        try {
            // 遍历整个数组，如果c中包含该元素，则把该元素放到写指针的位置（以complement为准）
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // 正常来说r最后是等于size的，除非c.contains()抛出了异常
            if (r != size) {
                // 如果c.contains()抛出了异常，则把未读的元素都拷贝到写指针之后
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // 将写指针之后的元素置为空，帮助GC
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                // 新大小等于写指针的位置（因为每写一次写指针就加1，所以新大小正好等于写指针的位置）
                size = w;
                modified = true;
            }
        }
        // 有修改返回true
        return modified;
    }
```
removeAll(Collection<?> c)

求差集
```java
    public boolean removeAll(Collection<?> c) {
        // 集合c不能为空
        Objects.requireNonNull(c);
        // 同样调用批量删除方法，这时complement传入false，表示删除包含在c中的元素
        return batchRemove(c, false);
    }
```
#### 序列化问题
----
  transient Object[] elementData;  如何序列化？
  
```java
    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException{
        // 防止序列化期间有修改
        int expectedModCount = modCount;
        // 写出非transient非static属性（会写出size属性）
        s.defaultWriteObject();
        // 写出元素个数
        s.writeInt(size);
        // 依次写出元素
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        // 如果有修改，抛出异常
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        // 声明为空数组
        elementData = EMPTY_ELEMENTDATA;
        // 读入非transient非static属性（会读取size属性）
        s.defaultReadObject();
        // 读入元素个数，没什么用，只是因为写出的时候写了size属性，读的时候也要按顺序来读
        s.readInt();
        if (size > 0) {
            // 计算容量
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            // 检查是否需要扩容
            ensureCapacityInternal(size);
            Object[] a = elementData;
            // 依次读取元素到数组中
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```
elementData定义为transient的优势，自己根据size序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用。
