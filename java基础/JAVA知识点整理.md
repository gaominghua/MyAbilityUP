目录

[一、 Java基础	2](#_Toc51061681)

[1.	Java中八大数据基本类型？	2](#_Toc51061682)

[2.	Java中序列化和反序列化底层原理？	2](#_Toc51061683)

[3.	Java中值传递和引用传递？	5](#_Toc51061684)

[4.	Java中强引用、软引用、弱引用、虚引用？	6](#_Toc51061685)

[5.	Java中Integer和int的区别？	10](#_Toc51061686)

[6.	LinkList和ArryList有什么区别？	10](#_Toc51061687)

[7.	Java中什么是深拷贝、浅拷贝？	11](#_Toc51061688)

[8.	Java中字符串常量池以及intern()方法？	15](#_Toc51061689)

[9.	Java中String、StringBuffer、StringBuilder区别？	17](#_Toc51061690)

[10.	Java中String类hashcode()原理？	20](#_Toc51061691)

[11.	Java中equal和“==”的区别？	21](#_Toc51061692)

[12.	Java中equal和hashcode()的联系？	22](#_Toc51061693)

[13.	Java中重写和重载的区别？	22](#_Toc51061694)

[14.	Java中final和finally和finalize的区别？	22](#_Toc51061695)

[15.	Java中comparable和comparator的作用和区别？	22](#_Toc51061696)

[16.	Java中Arrays.sort()底层原理？	23](#_Toc51061697)

[17.	Java中Collections.sort()底层原理？	24](#_Toc51061698)

[18.	Java中普通类和抽象类的区别？	25](#_Toc51061699)

[19.	Java中抽象类和接口的异同？	26](#_Toc51061700)

[20.	jdk1.8中关于HashMap 的实现跟 jdk1.7 的几点差别？	26](#_Toc51061701)

[21.	HashSet底层是什么,Hashmap的底层原理又是什么？	28](#_Toc51061702)

[22.	HashMap中的负载因子为什么是0.75？	30](#_Toc51061703)

[23.	JDK8中的HashMap什么时候将链表转化为红⿊树？	30](#_Toc51061704)

[24.	ConcurrentHashMap原理以及和HashMap和HashTbale、HashSet和TreeSet的区别？	31](#_Toc51061705)

[25.	JDK8中HashMap的put⽅法的实现过程？	34](#_Toc51061706)

[26.	JDK8中HashMap的get⽅法的实现过程	35](#_Toc51061707)

[27.	Java中常见的Error和Exception有哪些？	35](#_Toc51061708)

[28.	Java8新特性有哪些？	40](#_Toc51061709)

# Java基础

# Java中八大数据基本类型？

PS:括号内对应的是包装类

整型：byte、int、short、long**(Byte Short Integer Long)**

浮点型：double 、float**(Double Float)**

字符型：char**(Character)**

布尔类型：boolean**(Boolean)**

# Java中序列化和反序列化底层原理？

## 什么是序列化和反序列化

-   Java序列化是指把Java对象转换为字节序列的过程，而Java反序列化是指把字节序列恢复为Java对象的过程；

-   序列化：对象序列化的最主要的用处就是在传递和保存对象的时候，保证对象的完整性和可传递性。序列化是把对象转换成有序字节流，以便在网络上传输或者保存在本地文件中。序列化后的字节流保存了Java对象的状态以及相关的描述信息。序列化机制的核心作用就是对象状态的保存与重建。

-   反序列化：客户端从文件中或网络上获得序列化后的对象字节流后，根据字节流中所保存的对象状态及描述信息，通过反序列化重建对象。

-   本质上讲，序列化就是把实体对象状态按照一定的格式写入到有序字节流，反序列化就是从有序字节流重建对象，恢复对象状态

## 为什么要序列化和反序列化

java程序中要往磁盘中写入一个数据，这个数据如果是一个普通字符串，那么没有问题，所有机器都能正常识别字符串，即使需要转成对应的字节，计算机也知道怎么将字符串转成对应的字节（码表），**但是如果是一个Java对象那就麻烦了，磁盘并不知道你传递的是一个Java对象，换句话说，磁盘不知道按照什么格式把Java对象转换成对应的字节**

我们知道Java对象本质上是一个class字节码，磁盘并不知道怎么将这个字节码写入到磁盘中，按何种方式去写，所以需要"标识"一下，告诉磁盘：“我是个Java对象，你要按这种方式写入到磁盘中”，只不过"按这种方式写入到磁盘"。美曰：“按这种方式序列化到磁盘”，因此实现Serializable接口只是标识一下"我是个Java对象"

## Java如何实现序列化和反序列化

1.  java.io.ObjectOutputStream：表示对象输出流；

它的writeObject(Object
obj)方法可以对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中；

1.  java.io.ObjectInputStream：表示对象输入流；

它的readObject()方法源输入流中读取字节序列，再把它们反序列化成为一个对象，并将其返回；

## 实现序列化的要求

只有实现了Serializable或Externalizable接口的类的对象才能被序列化，否则抛出异常！

假定一个User类，它的对象需要序列化，可以有如下三种方法：

（1）若User类仅仅实现了Serializable接口，则可以按照以下方式进行序列化和反序列化

ObjectOutputStream采用默认的序列化方式，对User对象的非transient的实例变量进行序列化。  
ObjcetInputStream采用默认的反序列化方式，对对User对象的非transient的实例变量进行反序列化。

（2）若User类仅仅实现了Serializable接口，并且还定义了readObject(ObjectInputStream
in)和writeObject(ObjectOutputSteam out)，则采用以下方式进行序列化与反序列化。

ObjectOutputStream调用User对象的writeObject(ObjectOutputStream
out)的方法进行序列化。  
ObjectInputStream会调用User对象的readObject(ObjectInputStream
in)的方法进行反序列化。

（3）若User类实现了Externalnalizable接口，且User类必须实现readExternal(ObjectInput
in)和writeExternal(ObjectOutput out)方法，则按照以下方式进行序列化与反序列化。

ObjectOutputStream调用User对象的writeExternal(ObjectOutput
out))的方法进行序列化。  
ObjectInputStream会调用User对象的readExternal(ObjectInput
in)的方法进行反序列化。

## 相关注意事项

1、序列化时，只对对象的状态进行保存，而不管对象的方法；

2、当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

3、当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

4、并非所有的对象都可以序列化，至于为什么不可以，有很多原因了，比如：

安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的；

资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现；

5、声明为static和transient类型的成员数据不能被序列化。因为static代表类的状态，transient代表对象的临时数据。

6、序列化运行时使用一个称为 serialVersionUID
的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。为它赋予明确的值。显式地定义serialVersionUID有两种用途：

在某些场合，希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有相同的serialVersionUID；

在某些场合，不希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有不同的serialVersionUID。

7、Java有很多基础类已经实现了serializable接口，比如String,Vector等。但是也有一些没有实现serializable接口的；

8、如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因；

# Java中值传递和引用传递？

1.  值传递：值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。简单来说就是直接zd复制了一份数据过去，因为是直接复制，所以这种方式在传递时如果数据量非常大的话，运行效率自然就变低了，所以java在传递数据量很小的数据是值传递，比如java中的各种基本类型：int,float,double,boolean等类型的

2.  [引用传递](https://www.baidu.com/s?wd=%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)：[引用传递](https://www.baidu.com/s?wd=%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)其实就弥补了上面说的不足，如果每次传参数的时候都复制一份的话回，如果这个参数占用的内存空间太大的话，运行效率会很底下，所以引答用传递就是直接把[内存地址](https://www.baidu.com/s?wd=%E5%86%85%E5%AD%98%E5%9C%B0%E5%9D%80&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)传过去，也就是说[引用传递](https://www.baidu.com/s?wd=%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)时，操作的其实都是源数据，这样的话修改有时候会冲突，记得用逻辑弥补下就好了，具体的数据类型就比较多了，比如Object，二维数组，List，Map等除了基本类型的参数都是引用传递

# Java中强引用、软引用、弱引用、虚引用？

## 强引用

1.  **强引用不会被GC回收，并且在java.lang.ref里也没有实际的对应类型。**

举个例子来说：Object obj = new
Object();这里的obj引用便是一个强引用，不会被GC回收。

## 软引用

**被软引用的对象，如果内存空间足够，垃圾回收器是不会回收它的，如果内存空间不足，垃圾回收器将回收这些对象占用的内存空间。软件引用对应着java.lang.ref.SoftReference类，一个对象如果要被软引用，只需将其作为参数传入SoftReference类的构造方法中就行了。**

软引用在JVM报告内存不足的时候才会被GC回收，否则不会回收，正是由于这种特性,软引用在caching中用处广泛。

**软引用的用法：**

## 弱引用

**弱引用与前面的软引用相比，被弱引用了的对象拥有更短的内存时间（也就是生命周期）。垃圾回收器一旦发现了被弱引用的对象，不管当前内存空间是不是足够，都会回收它的内存，弱引用对应着java.lang.ref.WeakReference类**

一个对象如果想被弱引用，只需将其作为参数传入WeakReference类的构造方法中就行了。当GC一但发现了弱引用对象，将会释放WeakReference所引用的对象。弱引用使用方法与软引用类似，但回收策略不同。

## 虚引用

**虚引用不是一种真实可用的引用类型，完全可以视为一种“形同虚设”的引用类型。如果一个对象仅持有虚引用，那么就和没有任何引用一样。在任何时候都可能被垃圾回收器回收。虚引用必须和引用队列(ReferenceQueue)联合使用。**

# Java中Integer和int的区别？

1、Integer是int的包装类，int则是java的一种基本数据类型

2、Integer变量必须实例化后才能使用，而int变量不需要

3、Integer实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值  
4、Integer的默认值是null，int的默认值是0

# LinkList和ArryList有什么区别？

## ①ArrayList扩容机制

ArrayList的扩容主要发生在向ArrayList集合中添加元素的时候。由add()方法的分析可知添加前必须确保集合的容量能够放下添加的元素。主要经历了以下几个阶段：

-   第一，在add()方法中调用ensureCapacityInternal(size +
    1)方法来确定集合确保添加元素成功的最小集合容量minCapacity的值。参数为size+1，代表的含义是如果集合添加元素成功后，集合中的实际元素个数。换句话说，集合为了确保添加元素成功，那么集合的最小容量minCapacity应该是size+1。在ensureCapacityInternal方法中，首先判断elementData是否为默认的空数组，如果是，minCapacity为minCapacity与集合默认容量大小中的较大值。

-   第二，调用ensureExplicitCapacity(minCapacity)方法来确定集合为了确保添加元素成功是否需要对现有的元素数组进行扩容。首先将结构性修改计数器加一；然后判断minCapacity与当前元素数组的长度的大小，如果minCapacity比当前元素数组的长度的大小大的时候需要扩容，进入第三阶段。

-   第三，如果需要对现有的元素数组进行扩容，则调用grow(minCapacity)方法，参数minCapacity表示集合为了确保添加元素成功的最小容量。在扩容的时候，首先将原元素数组的长度增大1.5倍（oldCapacity
    + (oldCapacity \>\> 1)），然后对扩容后的容量与minCapacity进行比较：①
    新容量小于minCapacity，则将新容量设为minCapacity；②新容量大于minCapacity，则指定新容量。最后将旧数组拷贝到扩容后的新数组中。

# Java中什么是深拷贝、浅拷贝？

首先需要明白，浅拷贝和深拷贝都是针对一个已有对象的操作。那先来看看浅拷贝和深拷贝的概念。

在 Java
中，除了基本数据类型（元类型）之外，还存在类的实例对象这个引用数据类型。而一般使用
『 =
』号做赋值操作的时候。对于基本数据类型，实际上是拷贝的它的值，但是对于对象而言，其实赋值的只是这个对象的引用，将原对象的引用传递过去，他们实际上还是指向的同一个对象。

**浅拷贝：**如果在拷贝这个对象的时候，只对基本**数据类型进行了拷贝，而对引用数据类型只是进行了引用的传递**，而没有真实的创建一个新的对象，则认为是浅拷贝

**深拷贝:**在对引用数据类型进行拷贝的时候，创建了一个新的对象，并且复制其内的成员变量。

所以到现在，就应该了解了，所谓的浅拷贝和深拷贝，只是在拷贝对象的时候，对类的实例对象这种引用数据类型的不同操作而已。

**案例(浅拷贝)：**

1.  首先创建一个 class 为 FatherClass ，对其实现
    Cloneable接口，并且重写clone()方法。

2.  然后先正常 new 一个 FatherClass 对象，再使用 clone() 方法创建一个新的对象。

**案例(深拷贝)：**

总结：

引用：在栈内存中新增一个值(obj1)，但是与被复制的值(obj)实际引用的同一个堆地址,修改obj1的name和children都会影响obj；

浅拷贝：只复制原对象的第一层，并不包含子对象，修改obj2的name不会影响obj，修改obj2的children会影响obj；

深拷贝： 将原对象所有的属性全复制过来，修改obj3的name和children都不会影响obj。

# Java中字符串常量池以及intern()方法？

JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中(1,7之前在永久代中)，并且 JDK 1.8
中已经不存在永久代的结论。

# Java中String、StringBuffer、StringBuilder区别？

String：被声明final类，内部变量被final修饰。String类是不可变类，即一旦一个String对象被创建以后，包含在这个对象中的字符序列是不可改变的，直至这个对象被销毁。

应用场景：再字符串不经常发生改变的业务场景优先考虑String，例如：变量声明，少量的字符串拼接操作。

**String类的源码中有对 intern()方法的详细介绍, 翻译过来的意思是**: 当调用
intern()方法时, 首先会去常量池中查找是否有该字符串对应的引用,
如果有就直接返回该字符串; 如果没有, 就会在常量池中注册该字符串的引用,
然后返回该字符串

这里解释一下：

①执行完new String("a")+new
String("b")之后，堆中有a、b、ab对象，（但是请注意字符串常量池却没有ab字符串，类似的还有new
String("XX")+"YY"或者str+“ZZ”，（append相加也不会）简单来说相加的形式只要其中有非字面量，那么就不会在常量池创建这个相加的结果字符串）

②之后执行str1.intern(）方法，因为字符串常量池没有“ab”的引用或者字符串，那么将这个引用放入字符串常量池，这里就是str1的引用地址

③接着执行str2=“ab”，现在常量池找是否有“ab”的引用或者字符串，在第二步中，“ab”的引用已经放到了字符串常量池，那么这里就会直接返回这个引用，即str1的指向地址，所以str1和str2指向同一个地址，引用相同，返回true；

在执行new
String("hello")的时候，除了在堆中创建hello对象外，字符串常量池也会创建一个hello字符串，所以在str7.intern()的时候，并没有像第一种情况一样把引用放入常量池，因为已经有了，str8=“hello”，将str8指向字符串常量池的“hello”字符串，str7指向堆中的引用，所以str7和str8引用不同，返回false

**StringBuffer：**实现原理基于可修改的Char数组,默认大小16，继承AbstractStringBuilder，线程安全。

好处：每次添加字符串会判断内存块能否存放下字符，如果不够会开辟一块新的更大的内存块，将原来的内存块内容和新的内存块内容放在新的空间。

应用场景：再频繁的进行字符串的运算(如拼接、替换、删除等操作)，并且运行在多线程环境下。例如：XML解析，HTTP参数解析与封装。

**StringBuilder：**实现原理基于可修改的Char数组,默认大小16，继承AbstractStringBuilder，线程不安全。

应用场景：再频繁的进行字符串的运算(如拼接、替换、删除等操作)，并且运行在单线程环境下。例如：SQL语句拼接、JSON封装。

## String、StringBuffer、StringBuilder原理

### String原理

String对象一旦创建，其值是不能修改的，如果要修改，会重新开辟内存空间来存储修改之后的对象，即修改了
String 的引用。因为 String
的底层是用数组来存值的，数组长度不可改变这一特性导致了上述问题。‌如果我们在实际开发过程中需要对某个字符串进行频繁的修改，使用
String 就会造成内存空间的浪费

Java String类为什么是final的？

-   1.为了实现字符串池

-   2.为了线程安全

-   3.为了实现String可以创建HashCode不可变性

### StringBuffer原理

StringBuffer 和 String
类似，底层也是用一个数组来存储字符串的值，并且数组的默认长度为 16，即一个空的
StringBuffer 对象，数组长度为 16。实例化一个 StringBuffer 对象即创建了一个大小为
16 个字符的字符串缓冲区。

但是当我们调用有参构造函数创建一个 StringBuffer 对象时，数组长度就不再是 16
了，而是根据当前对象的值来决定数组的长度，数组的长度为“当前对象的值的长度+16”。所以一个
StringBuffer 创建完成之后，有 16
个字符的空间可以对其值进行修改。如果修改的值范围超出了 16
个字符，会先检查StringBuffer 对象的原 char
数组的容量能不能装下新的字符串，如果装不下则会对 char 数组进行扩容。

**那StringBuffer是怎样进行扩容的呢？**

扩容的逻辑就是创建一个新的 char
数组，将现有容量扩大一倍再加上2，如果还是不够大则直接等于需要的容量大小。扩容完成之后，将原数组的内容复制到新数组，最后将指针指向新的
char 数组。

### StringBuffer原理

StringBuilder 和 StringBuffer 拥有同一个父类
AbstractStringBuilder，同时实现的接口也是完全一样，都实现了java.io.Serializable,
CharSequence 两个接口。

## 总结：

（1）如果要操作少量的数据用 String；

（2）多线程操作字符串缓冲区下操作大量数据 StringBuffer；

（3）单线程操作字符串缓冲区下操作大量数据 StringBuilder。

# Java中String类hashcode()原理？

public int hashCode() {  
int h = hash;  
if (h == 0 && value.length \> 0) {  
char val[] = value;  
  
for (int i = 0; i \< value.length; i++) {  
h = 31 \* h + val[i];  
}  
hash = h;  
}  
return h;  
}

**hash
：**第一次调用hashcode()方法时，会赋值当前String的hash值给hash，第二次及以后调用时就不用再走计算了，这也是为什么要进行一个if判断了

**h ：**最终的hash值 默认为0

**value ：**当前string的char数组

**val[i] ：**当前String的第i个字符对应的int值

重点是for循环里面的逻辑：h = 31 \* h + val[i];

**选择质数31的原因？**

①第一，31是一个不大不小的质数，是作为 hashCode
乘子的优选质数之一。选择一个合适的质数可以降低hash算法冲突的概率

②第二、31可以被 JVM 优化，31 \* i = (i \<\< 5) - i

**我拿一个字符串做举例吧**

String a = "11"；

当a第一次调用hashcode()方法时，进入方法里面，hash的值为0

走到if判断时，条件成立，声明val数组，这个val就是{'1','1'}，一个char数组

然后往下走到for循环，

当i = 0 时：h = 31 \* 0 + val[0]; --------val[0]='1'
因为参与加法运算，char会转成int，而'1'的值为49

计算出结果为h = 49;

当 i = 1时，h = 31 \* 49 + val[1];

计算出结果为 h = 1568 ；

所以字段串 "11" 的hashcode值为1568。

# Java中equal和“==”的区别？

1.  **equals()方法**

用来比较的是两个对象的内容是否相等，由于所有的类都是继承自java.lang.Object类的，所以适用于所有对象，如果没有对该方法进行覆盖的话，调用的仍然是Object类中的方法，而Object中的equals方法返回的却是==的判断；

1.  **"=="比较**

比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。

# Java中equal和hashcode()的联系？

①equal相等（相同）的对象必须具有相等Hashcode码

②如果两个对象hashcode相同，那么他们不一定相等。（可能是Hash冲突）

在集合中，比如HashSet中，要求放入的对象不能重复，怎么判定呢？

首先会调用hashcode，如果hashcode相等，则继续调用equals，也相等，则认为重复。

# Java中重写和重载的区别？

**重写：**重写发生在子类和父类之间，重写要求子类被重写的方法与父类被重写的方法有相同的返回值，而且在功能上优于父类

**重载：**重载是发生在一个类种，具体相同名称的方法，不同的参数，例如参数的类型不同，参数的个数不同等。重载对返回类型没有特别的要求

# Java中final和finally和finalize的区别？

**final:**用于声明属性、方法、类、分别表示属性不可变、方法不可覆盖、类不可继承

**Finally**：是异常处理语句结构的一部分，表示总是执行

**Finalize：**是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法。可以覆盖此方法提供垃圾收集时的其他资源。

# Java中comparable和comparator的作用和区别？

**Comparable接口：**该接口只提供了一个ComparaTo()方法，该方法可以可以给两个对象排序。compareTo(Object
o)就是比较两个值，如果当前对象大于输入对象Object
，返回1，等于返回0，小于返回-1。

**Comparable** 是一个**排序接口**，实现了该接口的类，表示该类支持排序功能，重写
compareTo 方法可使程序按照我们的意愿对数组或列表进行排序

**Comparator接口**提供两个方法包含Compare()和equals(),其中Compare(Object
o1,Object
o2)包含两个对象如果o1\<o2返回负数，o1==o2返回0，o1\>o2返回正数。【第一个参数和第二个参数比较是升序，第二个参数和第一个参数比较是降序】

Equals()方法需要一个对象作为输入参数，它决定输入参数是否和comparator相等。只有当输入参数也是一个comparator并且和当前的comparator的排序结果相同的时候，该方法返回True。

**Comparator** 是一个**比较器接口**，如果我们需要对类进行排序，而该类并没有实现
Comparable 接口，那么我们可以通过实现 Comparator 接口来保持类的排序能力

# Java中Arrays.sort()底层原理？

在数组的数量小于47的情况下使用插入排序,在大于或等于47或少于286会进入快速排序（双轴快排）大于286采用归并排序。在判断少于286之前还有一个操作：

这里主要作用是看他数组具不具备结构：实际逻辑是分组排序，每降序为一个组，像1,9,8,7,6,8。9到6是降序，为一个组，然后把降序的一组排成升序：1,6,7,8,9,8。然后最后的8后面继续往后面找。  
每遇到这样一个降序组，++count，当count大于MAX_RUN_COUNT（67），被判断为这个数组不具备结构（也就是这数据时而升时而降），然后送给之前的sort(里面的快速排序)的方法（The
array is not highly structured,use Quicksort instead of merge sort.）  
如果count少于MAX_RUN_COUNT（67）的，说明这个数组还有点结构，就继续往下走下面的归并排序

# Java中Collections.sort()底层原理？

java.util.Collections
是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全等操作。
然后还有混排（shuffle）、反转（reverse）、替换所有的元素（fill）、拷贝（copy）、返回Collection中最小元素（min）、返回Collection中最大元素（max）、返回指定源列表中最后一次出现指定目标列表的起始位置（
lastIndexOfSubList ）、返回指定源列表中第一次出现指定目标列表的起始位置（
IndexOfSubList ）、根据指定的距离循环移动指定列表中的元素（rotate）;

事实上Collections.sort方法底层就是调用的Arrays.sort方法Collections 中的sort()
方法如下：

**legacyMergeSort (a,c)：归并排序**

**TimSort.sort() ： Timsort 排序，结合了合并排序（merge
sort）和插入排序（insertion sort）而得出的排序算法**

Timsort的核心过程:

TimSort
算法为了减少对升序部分的回溯和对降序部分的性能倒退，将输入按其升序和降序特点进行了分区。排序的输入的单位不是一个个单独的数字，而是一个个的块-分区。其中每一个分区叫一个run。针对这些
run 序列，每次拿一个 run 出来按规则进行合并。每次合并会将两个 run合并成一个
run。合并的结果保存到栈中。合并直到消耗掉所有的 run，这时将栈上剩余的
run合并到只剩一个 run 为止。这时这个仅剩的run 便是排好序的结果。

综上述过程，Timsort算法的过程包括 :

1、如何数组长度小于某个值，直接用二分插入排序算法

2、找到各个run，并入栈

3、按规则合并run

# Java中普通类和抽象类的区别？

1.  抽象类不能被实例化

2.  抽象类可以有抽象方法，抽象方法只需申明，无需实现

3.  含有抽象方法的类必须申明为抽象类

4.  抽象的子类必须实现抽象类中所有抽象方法，否则这个子类也是抽象类

5.  抽象方法不能被声明为静态

6.  抽象方法不能用private修饰

7.  抽象方法不能用final修饰

# Java中抽象类和接口的异同？

**接口和抽象类的不同点：**

（1）抽象类可以有构造方法，接口中不能有构造方法。

（2）抽象类中可以有普通成员变量，接口中没有普通成员变量

（3）抽象类中可以包含静态方法，接口中不能包含静态方法

（4） 一个类可以实现多个接口，但只能继承一个抽象类。

（5）接口可以被多重实现，抽象类只能被单一继承

1.  如果抽象类实现接口，则可以把接口中方法映射到抽象类中作为抽象方法而不必实现，而在抽象类的子类中实现接口中方法

**接口和抽象类的相同点：**

(1) 都可以被继承

(2) 都不能被实例化

(3) 都可以包含方法声明

(4) 派生类必须实现未实现的方法

# jdk1.8中关于HashMap 的实现跟 jdk1.7 的几点差别？

1. JDK8中使⽤了红⿊树

2.
JDK7中链表的插⼊使⽤的头插法（扩容转移元素的时候也是使⽤的头插法，头插法速度更快，⽆需遍
历链表，但是在多线程扩容的情况下使⽤头插法会出现循环链表的问题，导致CPU飙升），JDK8中链表使⽤的尾插法（JDK8中反正要去计算链表当前结点的个数，反正要遍历的链表的，所以直接使⽤尾
插法）

3.
JDK7的Hash算法⽐JDK8中的更复杂，Hash算法越复杂，⽣成的hashcode则更散列，那么hashmap
中的元素则更散列，更散列则hashmap的查询性能更好，JDK7中没有红⿊树，所以只能优化Hash算法使得元素更散列，⽽JDK8中增加了红⿊树，查询性能得到了保障，所以可以简化⼀下Hash算法，毕竟Hash算法越复杂就越消耗CPU

4.
扩容的过程中JDK7中有可能会重新对key进⾏哈希（重新Hash跟哈希种⼦有关系），⽽JDK8中没有这部分逻辑

5.
JDK8中扩容的条件和JDK7中不⼀样，除开判断size是否⼤于阈值之外，JDK7中还判断了tab[i]是否为空，不为空的时候才会进⾏扩容，⽽JDK8中则没有该条件了

6. JDK8中还多了⼀个API：putIfAbsent(key,value)

7.
JDK7和JDK8扩容过程中转移元素的逻辑不⼀样，JDK7是每次转移⼀个元素，JDK8是先算出来当前位

# HashSet底层是什么,Hashmap的底层原理又是什么？

底层是HashMap？hashMap底层数组+链表

另外：当hashmap里面设置了一个阈值，只有只有当这个阈值达到最大时候才会将链表变成红黑树生效，提高检索的效率！

# HashMap中的负载因子为什么是0.75？

1、
假如负载因子定为1（最大值），那么只有当元素填慢数组长度的时候才会选择去扩容，虽然负载因子定为1可以最大程度的提高空间的利用率，但是我们的查询效率会变得低下（因为链表查询比较慢）

结论：所以当加载因子比较大的时候：节省空间资源，耗费时间资源

2、加入负载因子定为0.5（一个比较小的值），也就是说，知道到达数组空间的一半的时候就会去扩容。虽然说负载因子比较小可以最大可能的降低hash冲突，链表的长度也会越少，但是空间浪费会比较大

**结论：所以当加载因子比较小的时候：节省时间资源，耗费空间资源但是我们设计程序的时候肯定是会在空间以及时间上做平衡，那么我们能就需要在时间复杂度和空间复杂度上做折中，选择最合适的负载因子以保证最优化。所以就选择了0.75这个值，Jdk那帮工程师一定是做了大量的测试，得出的这个值吧\~**

# JDK8中的HashMap什么时候将链表转化为红⿊树？

这个题很容易答错，⼤部分答案就是：当链表中的元素个数⼤于8时就会把链表转化为红⿊树。但是其实还

有另外⼀个限制：当发现链表中的元素个数⼤于8之后，还会判断⼀下当前数组的⻓度，如果数组⻓度⼩于

64时，此时并不会转化为红⿊树，⽽是进⾏扩容。只有当链表中的元素个数⼤于8，并且数组的⻓度⼤于

等于64时才会将链表转为红⿊树。

# ConcurrentHashMap原理以及和HashMap和HashTbale、HashSet和TreeSet的区别？

## HashSet

-   HashSet是基于HashMap实现的，默认构造函数是构建一个初始容量为16，负载因子为0.75
    的HashMap。封装了一个 HashMap 对象来存储所有的集合元素，所有放入 HashSet
    中的集合元素实际上由 HashMap 的 key 来保存，而 HashMap 的 value 则存储了一个
    PRESENT，它是一个静态的 Object 对象。

-   当我们试图把某个类的对象当成 HashMap的 key，或试图将这个类的对象放入 HashSet
    中保存时，重写该类的equals(Object obj)方法和 hashCode()
    方法很重要，而且这两个方法的返回值必须保持一致：当该类的两个的 hashCode()
    返回值相同时，它们通过 equals() 方法比较也应该返回
    true。通过这两个方法hashCode()和equals()保证元素的唯一性，方法自动生成

-   HashSet的其他操作都是基于HashMap的。只关心key，其中value是恒定得一个常量Object对象，不关心

## TreeSet

-   底层数据是红黑二叉树(平衡二叉树)结构

-   TreeSet类是按照从根节点开始，按照从左、中、右的原则依此取出元素

-   TreeSet是SortedSet接口的唯一实现类，TreeSet可以确保集合元素处于排序状态。TreeSet支持两种排序方式，自然排序
    和定制排序，其中自然排序为默认的排序方式。向TreeSet中加入的应该是同一个类的对象。TreeSet判断两个对象不相等的方式是两个对象通过equals方法返回false，或者通过CompareTo方法比较没有返回0

-   Java提供了一个Comparable接口，该接口里定义了一个compareTo(Object
    obj)方法，该方法返回一个整数值，实现了该接口的对象就可以比较大小。obj1.compareTo(obj2)方法如果返回0，则说明被比较的两个对象相等，如果返回一个正数，则表明obj1大于obj2，如果是
    负数，则表明obj1小于obj2。自然排序是根据集合元素的大小，以升序排列，如果要定制排序，应该使用Comparator接口，实现
    int compare(T o1,T o2)方法

## HashTable

-   底层数组+链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化

-   初始size为11，扩容：newsize = olesize\*2+1

-   计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length

## HashMap

-   底层数组+链表实现，可以存储null键和null值，线程不安全

-   初始size为16，扩容：newsize = oldsize\*2，size一定为2的n次幂

-   扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入

-   插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）

-   当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀

-   计算index方法：index = hash & (tab.length – 1)

## ConcurrentHashMap1.7结构：

-   Segment 数组、HashEntry 组成，和 HashMap 一样，仍然是数组加链表

-   HashEntry是用volatile是修饰的：

-   保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。（实现可见性）

-   禁止进行指令重排序。（实现有序性）

-   volatile 只能保证对单次读/写的原子性。i++ 这种操作不能保证原子性。

-   优点：它在进行put和get的时候就会先进行segment定位，segment数量就是数组的大小，初始为16.可以允许16个线程同时操作。

-   缺点： 数组加链表的方式，查询的时候，还得遍历链表，会导致效率很低

## ConcurrentHashMap1.8结构：

-   CAS + synchronized 来保证并发安全性。

-   同样在节点中把值和next采用了volatile去修饰，保证了可见性，并且也引入了红黑树，在链表大于一定值的时候会转换（默认是8）

-   put：根据 key 计算出 hashcode 。判断是否需要进行初始化。即为当前 key
    定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS
    尝试写入，失败则自旋保证成功。如果当前位置的 hashcode == MOVED ==
    -1,则需要进行扩容。如果都不满足，则利用 synchronized 锁写入数据。如果数量\>
    TREEIFY_THRESHOLD 则要转换为红黑树。

**总结：**

其实可以看出JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+红黑树。

1.  数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。  
    2.保证线程安全机制：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。  
    3.锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。  
    4.链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。  
    5.查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。

# JDK8中HashMap的put⽅法的实现过程？

1. 根据key⽣成hashcode

2. 判断当前HashMap对象中的数组是否为空，如果为空则初始化该数组

3. 根据逻辑与运算，算出hashcode基于当前数组对应的数组下标i

4. 判断数组的第i个位置的元素（tab[i]）是否为空

a. 如果为空，则将key，value封装为Node对象赋值给tab[i]

b. 如果不为空：

>   i. 如果put⽅法传⼊进来的key等于tab[i].key，那么证明存在相同的key

>   ii. 如果不等于tab[i].key，则：

>   1.
>   如果tab[i]的类型是TreeNode，则表示数组的第i位置上是⼀颗红⿊树，那么将key和

>   value插⼊到红⿊树中，并且在插⼊之前会判断在红⿊树中是否存在相同的key

>   2.
>   如果tab[i]的类型不是TreeNode，则表示数组的第i位置上是⼀个链表，那么遍历链表寻
>   找是否存在相同的key，并且在遍历的过程中会对链表中的结点数进⾏计数，当遍历到最
>   后⼀个结点时，会将key,value封装为Node插⼊到链表的尾部，同时判断在插⼊新结点之
>   前的链表结点个数是不是⼤于等于8，如果是，则将链表改为红⿊树。

iii.
如果上述步骤中发现存在相同的key，则根据onlyIfAbsent标记来判断是否需要更新value值，
然后返回oldValue

5. modCount++

6. HashMap的元素个数size加1

# JDK8中HashMap的get⽅法的实现过程

1. 根据key⽣成hashcode

2. 如果数组为空，则直接返回空

3.
如果数组不为空，则利⽤hashcode和数组⻓度通过逻辑与操作算出key所对应的数组下标i

4. 如果数组的第i个位置上没有元素，则直接返回空

5.
如果数组的第1个位上的元素的key等于get⽅法所传进来的key，则返回该元素，并获取该元素的value

6. 如果不等于则判断该元素还有没有下⼀个元素，如果没有，返回空

7. 如果有则判断该元素的类型是链表结点还是红⿊树结点

a. 如果是链表则遍历链表

b. 如果是红⿊树则遍历红⿊树

8. 找到即返回元素，没找到的则返回空

# Java中常见的Error和Exception有哪些？

在Java中，异常和错误同属于一个类：Throwable。异常和错误都是Java异常处理重要的类，且各自都包含大量子类。

Exception（异常）是应用程序中出现的可预测，可恢复的问题，通常是轻度或中度的问题。

Error（错误）大多表示运行应用程序时比较严重的错误。大多错误与编程者编写的程序无关，可能是代码运营时jvm出现的问题。例如，当
JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。

## java.lang.StackOverflowError

## java.lang.OuoifMemoryError:java heap space

## java.lang.OuoifMemoryError:GC overhead limit exceeded

## java.lang.OuoifMemoryError:Direct buffer memory

## java.lang.OuoifMemoryError:unable to create new native thread

解决方案:Linux扩大线程的个数

## java.lang.OuoifMemoryError:metaspace

# Java8新特性有哪些？

## Lambda表达式

**Lambda表达式使用前：**

**改进1：**

**改进2：**

**改进3：**

### 特性①：访问外层局部变量

### 特性②：访访问对象字段与静态变量

## 函数式接口(仅仅只包含唯一一个抽象方法的接口)

使用Lambda的前提是使用函数式接口，四大原生函数式接口：Consumer、Function、Predicate、Supplier，作用简化编程

**Function：函数式接口有一个参数并且返回一个结果，可以和其他函数组合的默认方法。**

**Predicate：断定型接口，有一个输入参数返回值只能式Boolean值**

**Consumer：消费型接口，消费者只有输入没有返回值**

**Supplier：供给型接口，返回一个任意范型的值，和Function接口不同的是该接口没有任何参数**

## 方法与构造函数引用

**引用的方式:**

objectName(对象)::instanceMethod(实例方法)

ClassName(类名)::staticMethod(静态方法)

ClassName(类名)::instanceMethod(实例方法)

## Date API

### ①Clock

Clock 类提供了访问当前日期和时间的方法，Clock 是时区敏感的，可以用来取代
**System.currentTimeMillis()** 来获取当前的微秒数。某一个特定的时间点也可以使用
Instant 类来表示，Instant 类也可以用来创建旧版本的java.util.Date 对象。

### ②Timezones(时区)

在新API中时区使用 ZoneId 来表示。时区可以很方便的使用静态方法of来获取到。
抽象类ZoneId（在java.time包中）表示一个区域标识符。
它有一个名为getAvailableZoneIds的静态方法，它返回所有区域标识符。

### ③LocalTime(本地时间)

### ④LocalDate(本地日期)

### ⑤LocalDateTime(本地日期时间)
