## 简介

**ArrayList 是 Java 集合框架中 List 接口的一个实现类。底层是数组，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长。**

`ArrayList`是`Vector`的翻版，区别在于`ArrayList`是线程不安全的，而`Vector`则是线程安全的。但是`Vector`是一个较老的集合，具有很多缺点，不建议使用，这里我们就不对其进行分析了。

ArrayList 可以说是我们使用最多的 List 集合，它有以下特点：

- 它是基于数组实现的List类
- 可以动态地调整容量
- 有序的（元素输出顺序与输入顺序一致）
- 元素可以为 null
- 不同步，非线程安全，效率高
- 查询快，增删慢
- 占用空间更小，对比 LinkedList，不用占用额外空间维护链表结构

## 继承结构

![ArrayList继承结构](https://gitee.com/yuanjianchen/pic/raw/master/2020/12/image-20201231151455078.png)

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

可以看到，`ArrayList`是`AbstractList`的子类，同时实现了`List`接口。除此之外，它还实现了三个标识型接口，这几个接口都没有任何方法，仅作为标识表示实现类具备某项功能。`RandomAccess`表示实现类支持快速随机访问，`Cloneable`表示实现类支持克隆，具体表现为重写了`clone`方法，`java.io.Serializable`则表示支持序列化，如果需要对此过程自定义，可以重写`writeObject`与`readObject`方法。

## 成员变量

```java
// 序列还
private static final long serialVersionUID = 8683452581122892189L;

/**
     * 数组初始默认容量为 10 
     */
private static final int DEFAULT_CAPACITY = 10;

/**
     * 空集合实例共享的空数组容器
     */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
     * 默认空集合的空数组容器  与 EMPTY_ELEMENTDATA 区分开，便于了解第一次添加元素是容器扩大多少
     */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
     * ArrayList 的储存容器，数组长度就是 ArrayList 的容量。任何一个空集合的 elementData 都为
     * DEFAULTCAPACITY_EMPTY_ELEMENTDATA，当集合添加第一个元素时，容量会扩展为 DEFAULT_CAPACITY
     * transient 表示不可被序列化
     */
transient Object[] elementData; // non-private to simplify nested class access

/**
     *	集合的大小（数组元素的个数）
     */
private int size;

/**
     * 数组最大长度。Integer.MAX_VALUE - 8 是因为一些 VM 会在数组保留一些信息头
     * 尝试申请更大的空间会抛出OutOfMemoryError: Requested array size exceeds VM limit
     */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

## 构造函数

```java
/**
 	* 根据指定容量创建对象数组
 	*/
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
     * 初始化一个空数组。
     * 初始化容量为 0，只有添加第一个元素时，才会扩容为 10 
     */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
     * 造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray 有可能不返回一个 Object 数组
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10。

## 内部类

```java
private class Itr implements Iterator<E> {}
private class ListItr extends Itr implements ListIterator<E> {}
private class SubList extends AbstractList<E> implements RandomAccess {}
static final class ArrayListSpliterator<E> implements Spliterator<E> {}
```

ArrayList有四个内部类，其中的**Itr是实现了Iterator接口**，同时重写了里面的**hasNext()**， **next()**， **remove()** 等方法；其中的**ListItr** 继承 **Itr**，实现了**ListIterator接口**，同时重写了**hasPrevious()**， **nextIndex()**， **previousIndex()**， **previous()**， **set(E e)**， **add(E e)** 等方法，所以这也可以看出了 **Iterator和ListIterator的区别**：ListIterator在Iterator的基础上增加了添加对象，修改对象，逆向遍历等方法，这些是Iterator不能实现的。

## 核心方法

### add

```java
/**
     * 添加指定元素到集合末尾
     */
public boolean add(E e) {
  	// 先确保elementData数组的长度足够，size是数组中数据的个数，因为要添加一个元素，所以size+1，
    // 先判断size+1的这个个数数组能否放得下，在这个方法中去判断数组长度是否够用
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 在数据中正确的位置上放上元素e，并且size++
    elementData[size++] = e;
    return true;
}

/**
     * 在集合指定位置插入元素，同时将当前位置上的元素以及右面的元素右移
     */
public void add(int index, E element) {
  	// 索引检查，抛出 IndexOutOfBoundsException
    rangeCheckForAdd(index);
		// 先确保elementData数组的长度足够
    ensureCapacityInternal(size + 1);  // Increments modCount!!
  	// 拷贝数组，指定位置以及之后的元素右移一位, 有效率问题
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

/**
     * 校验插入位置是否合理
     */
private void rangeCheckForAdd(int index) {
    // 插入的位置肯定不能大于size 和小于0
    if (index > size || index < 0)
      	// 如果是，就报数组越界异常
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
     * 将一个集合的元素按照其 Iterator（迭代器）遍历顺序添加到当前集合后面
     */
public boolean addAll(Collection<? extends E> c) {
  	// 把该集合转为对象数组
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
  	// 将集合
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

/**
     * 将指定集合插入到指定位置
     */
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
		// 计算需要移动的元素个数
    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//计算容量。判断初始化的elementData是不是空的数组,如果是空的话，返回默认容量10与minCapacity=size+1的较大值者。
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 此处考虑溢出 minCapacity 为 size + 1, 当添加新元素时，如果集合的 size + 1 大于 集合内部数组 elementData 的长度，此时就需要进行扩容。
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
     * 集合扩容
     */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
  	// 右移一位 扩容 1.5 倍 oldCapacity >> 1  相当于  oldCapacity/2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
		// 扩容 1.5 倍之后容量还不够，则是使用申请容量
    if (newCapacity - minCapacity < 0)

        newCapacity = minCapacity;
		// 如果扩容后容量大于数组最大分配容量(Integer.MAX_VALUE - 8) 则执行 hugeCapacity 方法
    if (newCapacity - MAX_ARRAY_SIZE > 0)

        newCapacity = hugeCapacity(minCapacity);

    // minCapacity is usually close to size, so this is a win:

    elementData = Arrays.copyOf(elementData, newCapacity);

}

/**
     * 如果扩容后容量大于数组最大分配容量(Integer.MAX_VALUE - 8)
     */
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow 溢出
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

至此，就分析完了数组的添加方法，以及添加时的扩容机制。可以看出每次扩容以及在指定位置插入元素或集合是都要调用`Arrays.copyOf()`和`System.arraycopy()`进行数组间的复制操作。虽然是底层方法，但每次复制也是影响效率的。`为了避免频繁扩容，根据空间和时间效率考量，数组扩容都会为 1.5 倍`。

了解了扩容机制，当处理大量数据时，为了避免频繁扩容,ArrayList 为我们提供了两种可行方案：

- 使用`ArrayList(int initialCapacity)`这个有参构造，在创建时就声明一个较大的大小，这样解决了频繁拷贝问题，但是需要我们提前预知数据的数量级，也会一直占有较大的内存。
- 除了添加数据时可以自动扩容外，我们还可以在插入前先进行一次扩容。只要提前预知数据的数量级，就可以在需要时直接一次扩充到位，与`ArrayList(int initialCapacity)`相比的好处在于不必一直占有较大内存，同时数据拷贝的次数也大大减少了。这个方法就是**ensureCapacity(int minCapacity)**，其内部就是调用了`ensureCapacityInternal(int minCapacity)`。

使用 `ensureCapacity()`做一下测试

```java
public class EnsureCapacityTest {

    public static void main(String[] args) {
        ArrayList<Object> list = new ArrayList<Object>();
        final int n = 10000000;
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < n; i++) {
            list.add(i);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("使用ensureCapacity方法前：" + (endTime - startTime));

        list = new ArrayList<Object>();
        long startTime1 = System.currentTimeMillis();
        list.ensureCapacity(n);
        for (int i = 0; i < n; i++) {
            list.add(i);
        }
        long endTime1 = System.currentTimeMillis();
        System.out.println("使用ensureCapacity方法后：" + (endTime1 - startTime1));
    }
}
```

输出结果为：

```java
使用ensureCapacity方法前：2645
使用ensureCapacity方法后：367
```

通过运行结果，我们可以很明显的看出向 ArrayList 添加大量元素之前最好先使用`ensureCapacity` 方法，以减少增量重新分配的次数。

### remove 方法

```java
/**
     * 删除指定位置的元素
     */
public E remove(int index) {
  	// 数组范围检查
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);
		// 要左移的元素的个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
  	// 将数组最后一个元素置为 null 以便 GC 收集，同时将集合 size 减一
    elementData[--size] = null; // clear to let GC do its work
		// 将删除的元素返回
    return oldValue;
}

/**
     * 检查数组是否越界
     */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
     * 从集合中删除第一个要删除的元素
     */
public boolean remove(Object o) {
  	// 循环遍历删除
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
              	// 与 o 相同，进行快速删除
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/*
     * 快速删除
     */
private void fastRemove(int index) {
    modCount++;
  	// 需要往右移动的元素的个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

/**
     * 清空集合， 此方法是将数组中的每一个元素值置为 null 并不会使数组长度发生变化
     */
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}

public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}

public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

/**
     * removeAll 和 retainAll 都调用此批量删除方法
     * removeAll 删除当前集合与指定集合的交集
     * retainAll 保留当前集合与指定集合的交集
     */
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
  	// r用来控制循环，w是记录有多少个交集
    int r = 0, w = 0;
    boolean modified = false;
    try {
      	// 遍历当前集合
        for (; r < size; r++)
            // 如果指定集合包含当前元素 并且 complement， complement：true 保留当前元素  false 删除当前元素
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r]; // 如果保留当前元素 则将当前元素一次填充到 当前数组中 同时计算元素的个数
    } finally {
      	// 即使 c.contains()发生异常，仍保持和 AbstractCollection 的兼容，
      	// 将 elementData 后续为处理的元素拷贝到 elementData w 索引处后面
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r; //记录处理过的元素的个数
        }
      	// 如果处理后的元素个数和之前原始数组长度不同，说明 w 索引后的数据都要置为 null
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

/**
     * 删除 fromIndex 到 toIndex 索引范围的元素
     */
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}

/**
     * 1.8 新增方法
     * 根据给定的条件删除元素 
     */
public boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
  	// 计算需要删除的元素，在过滤方法中出现任何异常都不会改变几个
    int removeCount = 0; // 需要删除的个属于
  	// 记录需要删除元素的位置
    final BitSet removeSet = new BitSet(size); 
  	// 将 modCount 存为final 临时变量 expectedModCount 防止并发修改
    final int expectedModCount = modCount;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        @SuppressWarnings("unchecked")
        final E element = (E) elementData[i];
        if (filter.test(element)) {
          	// 将 removeSet 第 i 为设置为 true/1
            removeSet.set(i);
            removeCount++;
        }
    }
  	// 校验是否存在并发修改
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }

  	// 将剩余的生存元素移到已删除元素剩余的空间上
    final boolean anyToRemove = removeCount > 0;
    if (anyToRemove) {
        final int newSize = size - removeCount;
        for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
          	// 跳过需要删除的 索引
            i = removeSet.nextClearBit(i);
            elementData[j] = elementData[i];
        }
        for (int k=newSize; k < size; k++) {
            elementData[k] = null;  // Let gc do its work
        }
        this.size = newSize;
      	// 校验并发
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    return anyToRemove;
}
```

`remove`几个方法都是类似的。删除过程中会出现以下问题：

* 删除元素时，会将集合内存数组容器中的当前元素置为 null，以便 GC 回收。**但是不会改变集合内部数组容器的大小**。
* 根据索引删除指定位置的元素时，会造成数组的复制，频繁删除影响效率。
* 批量删除/保留指定集合元素时的时间复杂度为 `c1.size * c2.size`，即为`n2`。效率很低。
* jdk8 新增方法`removeIf` 会抛出并发异常。

当遇到集合频繁增加删除时需要考虑 `ArrayList` 是否合适。

## get

```java
public E get(int index) {
  	// 检查索引是否正确
    rangeCheck(index);

    return elementData(index);
}

 public E get(int index) {
     rangeCheck(index);

     return elementData(index);
 }
```

ArrayList 由于内部是数组，可以根据索引快速访问，所以 ArrayList 通过索引对对象的访问特别快。同时返回的值都经过了向下转型（Object -> E），这些是对我们应用程序屏蔽的小细节。

## set

```java
public E set(int index, E element) {
  	// 检查索引是否正确
    rangeCheck(index);
		// 替换当前索引位置的元素
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

## indexOf & lastIndexOf

```java
// 从首开始查找数组里面是否存在指定元素
public int indexOf(Object o) {
    // 查找的元素为空
    if (o == null) { 
        // 遍历数组，找到第一个为空的元素，返回下标
        for (int i = 0; i < size; i++) 
            if (elementData[i]==null)
                return i;
    } else { 
        // 查找的元素不为空
        // 遍历数组，找到第一个和指定元素相等的元素，返回下标
        for (int i = 0; i < size; i++) 
            if (o.equals(elementData[i]))
                return i;
    } 
    // 没有找到，返回空
    return -1;
}

//返回列表中指定元素最后一次出现的索引，倒着遍历
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

说明：从头开始查找与指定元素相等的元素，需要注意的是可以查找null元素，意味着ArrayList中可以存放null元素的。与此函数对应的lastIndexOf，表示从尾部开始查找。

## contains

```java
//判断是否含有某个元素
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
```

## toArray

```java
/**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 返回的数组的运行时类型是指定数组的运行时类型。 
     */
public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
    return Arrays.copyOf(elementData, size);
}

public <T> T[] toArray(T[] a) {
    //如果只是要把一部分转换成数组
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    //全部元素拷贝到 数组 a
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

## 扩展

关于`System.arraycopy()`和 `Arrays.copyOf()`方法阅读源码的话，我们就会发现 ArrayList 中大量调用了这两个方法。比如：我们上面讲的扩容操作以及`add(int index, E element)`、`E remove(int index)`、`toArray()` 等方法中都用到了该方法！

### System.arraycopy

```java
// src：源对象
// srcPos：源对象对象的起始位置
// dest：目标对象
// destPost：目标对象的起始位置
// length：从起始位置往后复制的长度。
// 这段的大概意思就是解释这个方法的用法，复制src到dest，复制的位置是从src的srcPost开始，到srcPost+length-1的位置结束，复制到destPost上，从destPost开始到destPost+length-1的位置上
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

该方法是个 native 方法，由其他语言（c++）实现，将指定源数组中的数组从指定位置开始复制到目标数组的指定位置。

### Arrays.copyOf

```java
//Arrays的copyOf()方法传回的数组是新的数组对象，改变传回数组中的元素值，不会影响原来的数组。
//copyOf()的第二个自变量指定要建立的新数组长度，如果新数组的长度超过原数组的长度，则保留数组默认值
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

/**
     * @Description 复制指定的数组, 如有必要用 null 截取或填充，以使副本具有指定的长度
     * 对于所有在原数组和副本中都有效的索引，这两个数组相同索引处将包含相同的值
     * 对于在副本中有效而在原数组无效的所有索引，副本将填充 null，当且仅当指定长度大于原数组的长度时，这些索引存在
     * 返回的数组属于 newType 类
     *
     * @param original 要复制的数组
     * @param newLength 副本的长度
     * @param newType 副本的类
     * 
     * @return T 原数组的副本，截取或用 null 填充以获得指定的长度
     * @throws NegativeArraySizeException 如果 newLength 为负
     * @throws NullPointerException 如果 original 为 null
     * @throws ArrayStoreException 如果从 original 中复制的元素不属于存储在 newType 类数组中的运行时类型
     */
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

### 两者联系与区别

**联系：**看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`方法。
**区别：**

1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置。
2. copyOf()是系统自动在内部新建一个数组，并返回该数组。