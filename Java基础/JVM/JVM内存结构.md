
我们在开发 Java 程序的过程基本不用关心 Java 运行时的内存管理，是因为 Java 程序在运行时内存都由虚拟机来进行管理。Java 虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干个不同的数据区域，我们称之为`运行时数据区域`。

## 运行时数据区域

根据《Java虚拟机规范（Java SE 7版）》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域。

![JVM内存结构](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2020/08/08/JVM%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.png)



我们可以通过不同的几个维度对上图稍作一下分析：

1. JVM 内存结构由虚拟机栈、本地方法栈、方法区、堆、程序计数器组成。
2. 方法区和堆为线程共享内存，虚拟机栈。本地方法栈和程序计数器是线程独享的内存。
3. JVM 不同内存区域所占大小不同，其中堆内存最大，程序计数器占内存最小。

### 程序计数器（Program Counter Register）

程序计数器是一块较小的内存空间是，它可以看做是当前线程所执行的字节码的行号指示器。是线程私有的，每条线程都会有一个独立的程序计数器。是为了在多线程情况下，线程切换后能够恢复到正确的执行位置。此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

程序计数器在线程执行不同方法时储存的内容会有所不同：如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的虚拟机字节码的内存地址；如果正在执行的是一个 native 方法，这个计数器值则为**未定义（Undefined）**。

> Q：Java多线程执行native方法时程序计数器为空，那么线程切换后如何找到之前执行到哪里了？
>
> A: 对native方法而言，它的方法体并不是由Java字节码构成的，自然无法应用上述的“Java字节码地址”的概念。所以JVM规范规定，如果当前执行的方法是native的，那么pc寄存器的值**未定义**——是什么值都可以。Java线程总是需要以某种形式映射到OS线程上。映射模型可以是1:1（原生线程模型）、n:1（绿色线程 / 用户态线程模型）、m:n（混合模型）。
>
> 以HotSpot VM的实现为例，它目前在大多数平台上都使用1:1模型，也就是每个Java线程都直接映射到一个OS线程上执行。此时，native方法就由原生平台直接执行，并不需要理会抽象的JVM层面上的“pc寄存器”概念——原生的CPU上真正的PC寄存器是怎样就是怎样。就像一个用C或C++写的多线程程序，它在线程切换的时候是怎样的，Java的native方法也就是怎样的。 -Java线程总是需要以某种形式映射到OS线程上。映射模型可以是1:1（原生线程模型）、n:1（绿色线程 / 用户态线程模型）、m:n（混合模型）。
>
> 以HotSpot VM的实现为例，它目前在大多数平台上都使用1:1模型，也就是每个Java线程都直接映射到一个OS线程上执行。此时，native方法就由原生平台直接执行，并不需要理会抽象的JVM层面上的“pc寄存器”概念——原生的CPU上真正的PC寄存器是怎样就是怎样。就像一个用C或C++写的多线程程序，它在线程切换的时候是怎样的，Java的native方法也就是怎样的。 - [RednaxelaFX](https://www.zhihu.com/question/40598119/answer/87381512)

### 本地方法栈（Native Method Stack）

本地方法栈是为虚拟机使用到的 Native 方法服务。关于 Native 方法,官方给的说明是""。即在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。本地方法栈区域会抛出`StackOverflowError`和`OutOfMemoryError`异常

> Native Method: 
>
> A native method is a Java method whose implementation is provided by non-java code.
>
> ```java
>  public static native double getDouble(Object array, int index)
>         throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
> ```
>
> 这些方法的声明描述了一些非java代码在这些java代码里看起来像什么样子。

### 虚拟机栈（VM Stack）

Java 虚拟机栈是线程私有的，描述的是 Java 方法执行过程的内存模型：每个方法在执行同时都会创建一个栈帧。每个方法从调用到执行完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。栈帧中储存局部变量表、操作数栈、动态链接、方法出口等信息。

<img src="https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2020/08/image-20200807023603387.png" alt="Java 栈" style="zoom:50%;" />

#### 局部变量表（Local Variable Table）

是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量。存放了编译时期可知的各种基本数据类型（*boolean、byte、char、short、int、float、long、double*）和对象的引用。局部变量表在编译阶段就确定了需要分配的最大容量。

局部变量表的容量以变量槽（Variable Slot）为最小单位。每个 Slot 都应该能存放一个 boolean、byte、char、short、int、float、reference 或 returnAddress 类型的数据。

虚拟机通过索引定位的方式使用局部变量表，索引值的范围是从 0 开始至局部变量表最大的 Slot 数量。

下面我们看一个具体的代码示例：

<div id="code"></div>

```java
public int calc() {
    int a = 100;
    int b = 200;
    int c = 300;
    return (a + b) * c;
  }		
```

上述代码反编译后：

```java
  public int calc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        100
         2: istore_1
         3: sipush        200
         6: istore_2
         7: sipush        300
        10: istore_3
        11: iload_1
        12: iload_2
        13: iadd
        14: iload_3
        15: imul
        16: ireturn
      LineNumberTable:
        line 10: 0
        line 11: 3
        line 12: 7
        line 13: 11
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      17     0  this   Lai/advance/common/VariableLocal;
            3      14     1     a   I
            7      10     2     b   I
           11       6     3     c   I


```

我们注意到反编译后的代码第五行`locals=4`就说明了我们我们局部变量表的大小为 4 个 Slot，最下面的`LocalVariableTable` 展示了局部变量表里面具体的内容。对于实例方法（非 static）局部变量表第 0 位索引的 Slot 默认值实例的引用，也就是我们一直使用的`this`关键，其余局部变量依次排序。

#### 操作数栈

操作数栈也称操作栈，它是一个后入先出（Last In First Out,LIFO）栈。同局部变量表一样，操作数栈的最大深度也在编译时期写入到 `Code`属性中，具体可参考上述[代码示例](#code)中反编译后`stack=2`。

在一个方法刚开始执行的时候，这个方法的操作数栈是空的，在方法执行过程中，会有各种字节码指令往操作数栈中写入和提取内容，也就是说出栈/入栈操作。

#### 动态链接

动态链接是程序在运行期间将字节码中的符号引用转化为直接引用的过程。Class 文件的常量池中有大量的符号引用，字节码中的方法的调用指令调用常量池中存放的字面量符号，而这些字面量符号指向具体的方法。

* 部分符号在类加载或者第一次使用的时候就转化为直接引用，这种称之为静态链接。
* 部分符号引用在运行期间转化为直接引用,这种转化为**动态链接**

#### 方法返回地址

一个方法开始执行后只有两种方式可以退出这个方法。

##### 正常完成出口（Normal Method Invocation Completion）

执行引擎遇到任意一个方法返回的字节码指令，根据返回指令来决定是否将返回值和返回值的类型传递给上层的方法调用者。

一般来说，方法正常退出时，调用者的 PC 计数器的值可以作为返回地址，栈帧中很可能会保存这个计数器值。

##### 异常完成出口（Abrupt Method Invocation Completion）

在方法执行过程中遇到了异常，并且这个异常没有在方法体内得到处理。异常完成出口的方式是不会给上层调用者产生任何返回值的。

异常退出时，返回地址是要通过异常处理表来确定的，所以栈帧中一般不会保存这部分信息。

### 堆（Heap）

Java 堆是 Java 虚拟机所管理的内存中最大的一块。堆为所有线程共享的内存区域，在虚拟机启动时创建。堆唯一的目的就是存放对象的实例，几乎所有的对象都在堆内存区域存放。此区域也是发生 OOM 的重灾区。

![Java 堆](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9oZWxsby5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70-20200807161258296.jpeg)

堆的内存结构可以划分为**新生代**和**老年代**，默认`1:2`，其中新生代又被细分为 `Eden` 和两个个`Survivor`区域，两个`Servivor`分别以`from` `to` 来进行区分，Eden:from:to 默认为 `8：1：1`。

**JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的。**

### 方法区（Method Area）

方法区和堆一样，是各个线程共享的内存区域，它用于储存已被虚拟机加载的类信息（InstanceKlass）、常量、静态变量、及时编译期编译后的代码等数据(比如spring 使用IOC或者AOP创建bean时，或者使用cglib，反射的形式动态生成class信息等)。对于方法区的具体实现，会根据不同的 JVM 以及不同的版本而有所区别。在目前已经发布的JDK 1.7的HotSpot中，已经把原本放在永久代的字符串常量池移出。根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。



#### 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用。这部分内容将在类加载后进入方法区的运行时常量池中存放。运行时常量池具备动态性，在运行期间也可以将新的常量放入池中，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

> 补充说明：
>
> 1. 方法区在逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”（Non-Heap）。
>
> 2. 永久代（Permanent Generation）：永久带是 Hotspot 虚拟机独有的概念，因为 Hotspot 虚拟机把 GC 分代收集扩展至方法区，或者说用永久代来实现方法区。而对于其他虚拟机（BEA JRockit IBM J9等）是没有永久代的概念的。
>
> 3. 同样对于 Hotspot 虚拟机在jdk1.6、jdk1.7、jdk1.8 版本对方法区的实现时有所不同的：
>
>    1. 永久带的概念存在小于等于1.7 版本。对于字符串常量池来讲 1.7 以前的版本存放于方法区，1.7版本字符串常量池和类的静态变量存放于堆中，符号引用(Symbols)转移到了native heap。
>    2. 1.8 没有了永久带的概念，方法区又元空间（Metaspace）来实现，不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。
>    3. 1.8 元空间在储存内容方面和 1.7 没有发生改变，依然静态变量和字符串常量池在堆内存放，符号引用在 Native Heap 存放，元空间只存放储类和类加载器的元数据信息。只是在内存限制、垃圾回收等机制上改变较大。元空间的出现就是为了解决突出的类和类加载器元数据过多导致的OOM问题。
>
> 4. JDK 8 中永久代向元空间的转换
>
>    1. 字符串存在永久代中，容易出现性能问题和内存溢出。
>    2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
>    3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
>    4. Oracle 可能会将HotSpot 与 JRockit 合二为一。
>    
>    



## 总结

本文只是讲述了一些 JVM 的内存结构，以及不同内存区域的基本概念，对于不同内存区域的参数调整，以及会出现怎么样的异常等内容会专门用一篇文章来讲解。上面我们已经讲明白了JVM 内存区域的划分以及概念，下面根据脑图归纳总结一下：

![JVM 内存结构xmid](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2020/08/JVM%20%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84xmid.png)


参考：

[1] 周志明.深入理解Java虚拟机：JVM高级特性与最佳实践.北京:机械工业出版社,2013

[2] [RednaxelaFX](https://www.zhihu.com/people/rednaxelafx).[Java多线程执行native方法时程序计数器为空，那么线程切换后如何找到之前执行到哪里了？](https://www.zhihu.com/question/40598119/answer/87381512)

[3] [RednaxelaFX](https://www.zhihu.com/people/rednaxelafx).[JVM符号引用转换直接引用的过程?](https://www.zhihu.com/question/50258991/answer/120450561)

[4] [secbro2 ](https://juejin.im/user/3368559359568696).[JVM之内存结构详解](https://juejin.im/post/6844903969349697543)