

## 概述

LinkedList 是 Java 集合框架中一个重要的实现，其底层采用的双向链表结构。和 ArrayList 一样，LinkedList 也支持空值和重复值。由于 LinkedList 基于链表实现，存储元素过程中，无需像 ArrayList 那样进行扩容。但有得必有失，LinkedList 存储元素的节点需要额外的空间存储前驱和后继的引用。另一方面，LinkedList 在链表头部和尾部插入效率比较高，但在指定位置进行插入时，效率一般。原因是，在指定位置插入需要定位到该位置处的节点，此操作的时间复杂度为`O(N)`。最后，LinkedList 是非线程安全的集合类，并发环境下，多个线程同时操作 LinkedList，会引发不可预知的错误。

![LinkedList数据结构](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/interview-9-03.png)

## 继承体系

LinkedList 的继承体系较为复杂，继承自 AbstractSequentialList，同时又实现了 List 和 Deque 接口。继承体系图如下

![LinkedList继承体系](https://gitee.com/yuanjianchen/pic/raw/master/2021/01/image-20210102101609490.png)

LinkedList 继承自 AbstractSequentialList，AbstractSequentialList 提供了一套基于顺序访问的接口。通过继承此类，子类仅需实现部分代码即可拥有完整的一套访问某种序列表（比如链表）的接口。深入源码，AbstractSequentialList 提供的方法基本上都是通过 ListIterator 实现的，比如：

```java
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

public void add(int index, E element) {
    try {
        listIterator(index).add(element);
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

// 留给子类实现
public abstract ListIterator<E> listIterator(int index);
```

所以只要继承类实现了 listIterator 方法，它不需要再额外实现什么即可使用。对于随机访问集合类一般建议继承 AbstractList 而不是 AbstractSequentialList。LinkedList 和其父类一样，也是基于顺序访问。所以 LinkedList 继承了 AbstractSequentialList，但 LinkedList 并没有直接使用父类的方法，而是重新实现了一套的方法。

另外，LinkedList 还实现了 Deque (double ended queue)，Deque 又继承自 Queue 接口。这样 LinkedList 就具备了队列的功能。比如，我们可以这样使用：

```java
Queue<T> queue = new LinkedList<>();
```

除此之外，我们基于 LinkedList 还可以实现一些其他的数据结构，比如栈，以此来替换 Java 集合框架中的 Stack 类（该类实现的不好，《Java 编程思想》一书的作者也对此类进行了吐槽）。

## 成员变量

```java
// 序列号
private static final long serialVersionUID = 876323262645176354L;

// 集合大小
transient int size = 0;

// 第一个节点的指针
transient Node<E> first;

// 最后一个节点的指针
transient Node<E> last;
```

`Node`类型在[内部类](#内部类)中会展示源码。主要有当前对象、前驱节点、后继节点组成。关于 `first`和`next`变量有几个特殊的点：

* `first` 节点的前驱节点为 null，同时前驱节点为 null 的节点也为`first`节点。
* `last` 节点的后继节点为null，同时后继节点为 null 的节点也为`last`节点。
* 当 LinkedList 只有一个对象时，此对象既为 `first` 又为 `last`。

## 构造方法

```java
// 构造一个空集合
public LinkedList() {
}

// 构造一个指定集合的 LinkedList
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

## 内部类

```java
private class ListItr implements ListIterator<E> {}
private class DescendingIterator implements Iterator<E> {}
static final class LLSpliterator<E> implements Spliterator<E> {}
// 双向链表实现主要类
private static class Node<E> {
  	// 当前元素
    E item;
    // 前驱节点
    Node<E> next;
  	// 后继节点
    Node<E> prev;
		// 构造函数
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

LinkedList 双向链表主要是通过`Node`类实现。其成员变量`first`，`last`的类型都为`Node`。

## 核心方法

LinkedList继承自AbstractList，又实现了Deque。因此同时有List和Queue的方法。这里将分别分析属于List和Queue的操作并给出时间复杂度。

LinkedList有几个`linkXxx`和`unlinkXxx`的基础操作，后面实现的属于List的和Queue的几个方法都是基于这几个方法来实现的。

### LinkedList 基本方法

我们先分别看看这几个基础方法。除了`node()`方法外，其他方法的时间复杂度都为`O(1)`

#### node

```java
// 返回指定索引位置的 node 对象
Node<E> node(int index) {
    // assert isElementIndex(index);
		// 如果 index < size/2 则从0开始查找指定下标的节点
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else { // 如果 index ≥ size/2 则从后开始遍历
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

#### linkFirst

```java
// 将对象 e 链接为第一个对象
private void linkFirst(E e) {
    final Node<E> f = first;
  	// 新建 node， 前驱节点为 null 后继节点为 f(first)
    final Node<E> newNode = new Node<>(null, e, f);
  	// 将 first 引用改为新建节点
    first = newNode;
    if (f == null) // f == null 表示当前 linkedList 为空集合，添加的第一个元素 即为第一个节点又为最后一个节点
        last = newNode;
    else // f 不为空 将 f 前驱节点置为新建节点
        f.prev = newNode;
    size++; // 集合大小 +1 
    modCount++;
}
```

#### linkLast

```java
// 设置最后一个节点
void linkLast(E e) {
    final Node<E> l = last;
  	// 新建节点 前驱节点为当前 last 节点，后继节点为 null
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null) // last 为 null 说明该集合为空集合 同时将 first 引用到新建节点上
        first = newNode;
    else // last 不为空将 last 的后继节点设置为新建节点
        l.next = newNode;
    size++; // 集合大小 +1
    modCount++;
}
```

#### linkBefore

```java
// 将 e 插入到节点 succ 前面
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
  	// 取出 succ 节点的前驱节点
    final Node<E> pred = succ.prev;
  	// 新建节点，前驱节点为 succ 的前驱节点，后继节点为 succ
    final Node<E> newNode = new Node<>(pred, e, succ);
  	// 将 succ 的前驱节点替换为新建节点
    succ.prev = newNode;
    if (pred == null) // 如果 succ 之前的前驱节点为 null 则说明 succ 为 first 节点，此时 first 节点就应该设置为新建节点
        first = newNode;
    else // succ 不为 first 节点则将 succ 之前的前驱节点 pred 的后继节点设置为新建节点
        pred.next = newNode;
    size++; // 集合大小 +1
    modCount++;
}
```

#### unlinkFirst

```java
// 删除当前 first 节点
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
  	// 取出 first 节点的数据对象
    final E element = f.item;
  	// 取出 first 节点的后继节点
    final Node<E> next = f.next;
  	// 将 first 节点的数据对象以及将 first 节点的后继节点置为 null（当 first 对象不再引用任何对象，并且不被任何对象引用时，GC 会标记回收）
    f.item = null;
    f.next = null; // help GC
  	// 将 next 对象设置为 first 对象（之前的 first 对象会被 GC 回收）
    first = next;
    if (next == null) // 如果 next 为空说明集合只有一个 first 元素 此时需要将 last 也清空掉
        last = null;
    else
        next.prev = null; // 如果 next 不为空，next 即为 first 节点，将 next 节点的前驱节点置为 null
    size--; // 集合大小 -1
    modCount++;
    return element;
}
```

#### unlinkLast

```java
// 删除集合最后一个节点
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
  	// 取出last节点的数据对象
    final E element = l.item;
  	// 取出 last 节点的前驱节点
    final Node<E> prev = l.prev;
  	// 将 last数据对象以及前驱节点设置为 null 便于 GC 回收
    l.item = null;
    l.prev = null; // help GC
  	// 将 prev 节点设置为 last 节点
    last = prev; 
    if (prev == null) // prev 为 null 说明 l 节点既为 first 又为 last
        first = null; // 将 first 也设置为 null
    else
        prev.next = null; // 将 prev 后继节点置为 null 删除对 l 节点的引用
    size--; // 集合大小 -1
    modCount++;
    return element;
}
```

```java
// 删除节点
E unlink(Node<E> x) {
    // assert x != null;
  	// 取出节点 x 的数据对象，前驱节点prev和后继节点next
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

  	// prev == null 则 x 为 first，因此删除 x 将 next 设置为 first
    if (prev == null) {
        first = next;
    } else { 
      	// prev 不为 null 将 prev 的后继节点设置为 next
        prev.next = next;
      	// 删除 x 对 prev 节点的引用
        x.prev = null;
    }
		
 		// next == null 说明 x 为 last 节点 删除 x 则需要将 prev 设置为 last
    if (next == null) { 
        last = prev;
    } else {
      	// 将 next 的前驱节点设置为 prev
        next.prev = prev;
      	// 删除 x 对 next 节点的引用
        x.next = null;
    }
		// 将 x 节点的数据对象设置为 null，等待 gc 清理 x
    x.item = null;
    size--; // 将集合大小 -1
    modCount++;
    return element;
}
```

整个插入过程其实很简单，如下图示插入过程：

第一步：新建 node，newNode 的 next 设置为 succ，同时将 newNode 的 prev 设置为 succ.prev。

![新建 node](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102191405594.png)

第二步：将 succ 的 prev 设置成 newNode，同时将 succ.prev 的 next 设置成 newNode。

![设置 node](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102192027154.png)

插入的过程包含以上两个核心步骤，和另外一个分支步骤：当 succ 为 first 时，succ.prev 则为 null，newNode 将会被设置成 first。

另外，linkFirst 和 linkLast 方法和此过程核心步骤相同，区别只在于 linkFirst 的 newNode.prev = null 和 linkLast 的 newNode.next = null。此外，linkFirst 和 linkLast 需要考虑当集合为空时，newNode 既为 first 又为 last。

删除方法主要分为三步：

1. 确定要删除的 node x。

   ![确定要删除的 node x](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102200117685.png)

2. 将 x.prev.next 设置为 x.next，同时将 x.prev 的引用置为 null。

   ![第二步](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102200504310.png)

3. 将 x.next.prev 设置为 x.prev，同时将 x.next 的引用置为 null

   ![第三步](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102200814782.png)

在这核心三步外还有两个分支步骤：

1. 修改 x.prev 时需要判断 x 是否为 first，如果 x == first 则需要将 x.next 设置为 first。
2. 修改 x.next 时需要判断 x 是否为 last， 如果 x == last 则需要将 x.prev 设置为 last。

### List 方法

#### add

```java
// 添加对象  添加到集合最后面
public boolean add(E e) {
  	// 调用 本身的 linkLast 方法
    linkLast(e);
    return true;
}
// 在指定位置添加对象
public void add(int index, E element) {
  	// 检查索引
    checkPositionIndex(index);
		// 如果 index == size 说明在集合最后面插入
    if (index == size)
        linkLast(element);
    else  // 在指定节点前插入对象
        linkBefore(element, node(index));
}

public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

public boolean addAll(int index, Collection<? extends E> c) {
  	// 索引校验
    checkPositionIndex(index);
		// 将集合转换为数组
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;
		
  	// 获取 index 处的 pred succ 节点
    Node<E> pred, succ;
  	// index == size 说明在集合最后插入
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }
		// 循环 c 集合，将其中元素不断插入到当前集合中
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null) // pred == null 说明 succ 为first  将新建 node 设置为 first，再次循环 pred 则为上次循环的新建 node
            first = newNode
        else // 将新建 node 设置为 pred 节点的 next
            pred.next = newNode;
        pred = newNode; // 将 新建 node 设置为 pred ，再次循环插入的对象都将在 pred 后面
    }
		// succ == null 说明是在集合尾部插入
    if (succ == null) {
        last = pred; // 此时的 pred 为最后一个新建节点，需要将 pred 设置为 last
    } else {
        pred.next = succ;  // 集合中部插入，要讲 pred 的后继节点设置为 succ
        succ.prev = pred;  // 并把 succ 的前驱节点设置为 pred
    }

    size += numNew;  // 集合大小加上 c 的长度
    modCount++;
    return true;
}
```

List 中的 add 方法中单个元素的插入实现方式就是调用了 LinkedList 内部的 linkxx 方法。区别在于批量添加：

第一步：确定 succ 和 pred。

第二步：循环 c 集合，新建 node 逐个添加至链表。

![循环添加](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102194307203.png)

第三步：newNode 添加至链表后，将 newNode 设置为 pred。再次循环添加。

![依次循环添加](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2021/01/02/image-20210102195309931.png)

List 的 addAll 主要步骤包含以上核心三步，同时包含两个分支步骤：

1. 在首次添加元素时需要判断 newNode 是否为 first。
2. 在所有元素添加完之后需要判断 newNode 及 pred 是否为 last。

#### remove

```java
// 删除指定对象
public boolean remove(Object o) {
    if (o == null) { 
      	// 如果 o == null 则从 first 开始遍历 当遇到 第一个为空的元素时，将此元素删除。
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
      	// 从 first 开始遍历，删除一个满足 o.equals(x.item) 的元素。
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
// 删除指定索引位置的元素
public E remove(int index) {
    checkElementIndex(index);
  	// 需要确定 index 索引位置的元素，然后删除。
    return unlink(node(index));
}

// 继承 Collection 方法，循环调用 Iterator remove 方法删除 效率很低
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;
    Iterator<?> it = iterator();
    while (it.hasNext()) {
        if (c.contains(it.next())) {
            it.remove();
            modified = true;
        }
    }
    return modified;
}
```

#### set

```java
// 修改元素
public E set(int index, E element) {
    checkElementIndex(index);
  	// 查询当前索引对应的 node
    Node<E> x = node(index);
  	// 更新 node.item
    E oldVal = x.item;
    x.item = element;
  	// 返回 old value
    return oldVal;
}
```

#### get

```java
public E get(int index) {
    checkElementIndex(index);
  	// 查询当前索引对应的 node
    return node(index).item;
}
```

### Queue部分的方法

#### add

```java
public void addFirst(E e) {
    linkFirst(e);
}

public void addLast(E e) {
    linkLast(e);
}

// 队列添加方法
public boolean offer(E e) {
    return add(e);
}

public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// 栈添加方法
public void push(E e) {
    addFirst(e);
}
```

可以看出 queue 中的添加方法提供了多种，但是都是通过调用 LinkedList 本身的 linkxx 来实现的。

#### remove

```java
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

public E pop() {
    return removeFirst();
}

public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

#### get

```java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

public E element() {
    return getFirst();
}

public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

public E peekFirst() {
   final Node<E> f = first;
   return (f == null) ? null : f.item;
}
```

Queue 的所有方法都是通过已有的 LinkedList 本身的方法实现。其中的区别主要在于队列/栈的语法规范上，以及对于 null 值的处理（返回 null 或者抛出异常）。由于是实现了队列/栈，并没有提供修改方法。

## 总结

* LinkedList 底层采用的双向链表结构，支持 null 值和重复值。
* LinkedList 继承实现了 Queue，可以实现队列以及栈的功能。
* LinkedList 存储元素的节点需要额外的空间存储前驱和后继的引用，占用空间有所增加。
* LinkedList 在链表头部和尾部插入效率比较高，但在指定位置进行插入时，效率一般。
* LinkedList 在随机索引访问时效率一版，每次都要从 first/last 进行遍历。
* LinkedList 是非线程安全的集合类。

