[toc]
<!-- D:\工作开发\VSCode\VSCode-workplace\MyAbilityUP\picture\JVM -->
#1.什么是JVM、体系结构是怎么样的？
JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。Java虚拟机包括一套字节码指令集、一组寄存器、一个栈、一个垃圾回收堆和一个存储方法域。 JVM屏蔽了与具体操作系统平台相关的信息，使Java程序只需生成在Java虚拟机上运行的目标代码（字节码）,就可以在多种平台上不加修改地运行。JVM在执行字节码时，实际上最终还是把字节码解释成具体平台上的机器指令执行。
主要的子系统构成
•类加载子系统
•运行时数据区（内存结构）
•执行引擎
<span><div style="text-align: center;">
![jvm-1](../picture/JVM/jvm-1.png)
![jvm-2](../picture/JVM/jvm-2.png)
![jvm-3](../picture/JVM/jvm-3.png)
</div></span>
#2.JVM中类加载的过程是怎么样的？
Java的每个类，在JVM中，都有一个对应的Klass类实例与之对应，存储类的元信息如：常量池、属性信息、方法信息。
##一、klass模型类的继承结构：
<span><div style="text-align: center;">
![jvm-4](../picture/JVM/jvm-4.png)
</div></span>
从继承关系上也能看出来，类的元信息是存储在原空间的
普通的Java类在JVM中对应的是instanceKlass类的实例，再来说下它的三个字类InstanceMirrorKlass：用于表示

- java.lang.Class，Java代码中获取到的Class对象，实际上就是这个C++类的实例，存储在堆区，学名镜像类
- InstanceRefKlass：用于表示java/lang/ref/Reference类的子类InstanceClassLoaderKlass：用于遍历某个加载器加载的类
  
Java中的数组不是静态数据类型，是动态数据类型，即是运行期生成的，Java数组的元信息用ArrayKlass的子类来表示：

  - TypeArrayKlass：用于表示基本类型的数组
  - ObjArrayKlass：用于表示引用类型的数组
  

##二、类加载的过程
<span><div style="text-align: center;">
![jvm-5](../picture/JVM/jvm-5.png)
</div></span>
类加载机制：JVM把class文件加载到内存，并对数据进行校验、解析和初始化，最终形成JVM可以直接使用的java类型的全过程。
###加载过程
将.class文件通过IO流的方式加载到内存当中
1、通过类的全限定名获取存储该类的class文件（没有指明必须从哪获取）
2、解析成运行时数据，即instanceKlass实例，存放在方法区
3、在堆区生成该类的Class对象，即instanceMirrorKlass实例
###链接过程
1.验证阶段：文件格式验证、元数据验证、字节码验证、符号引用验证
2.准备阶段：给类中的静态变量分配内存，并赋予初始值
如果被final修饰，在编译的时候会给属性添加ConstantValue属性，准备阶段直接完成赋值，即没有赋初值这一步
###解析阶段
将虚拟机常量池的符号引用替换成字节引用的过程
将常量池中的符号引用转为直接引用
解析后的信息存储在ConstantPoolCache类实例中
1、类或接口的解析
2、字段解析
3、方法解析
4、接口方法解析
###初始化过程
初始化过程就会对类中的静态变量初始化为指定的值,执行静态代码块，执行构造器。静态字段、静态代码段，字节码层面会生成clinit方法方法中语句的先后顺序与代码的编写顺序相关。
###使用过程
###卸载过程
###总结
1.	将.class文件加载到内存在堆中会产生一个对应的Klass类的元信息作为方法区数据访问的入口
2.	链接过程将类中的静态变量分配内存，并且赋予初始值
3.	初始化对类中的静态变量初始化为指定的值,执行静态代码块，执行构造器。


#3.	JVM中类加载种类有哪些？
JVM中有两种类型的类加载器，由C++编写的及由Java编写的。除了启动类加载器（Bootstrap Class Loader）是由C++编写的，其他都是由Java编写的。由Java编写的类加载器都继承自类java.lang.ClassLoader。
##1.	启动类加载器Boostrap ClassLoader
负责加载JRE核心类库，jre包下rt.jar,charsets.jar等，C++语言编写，无法直接访问。因为启动类加载器是由C++编写的，通过Java程序去查看显示的是null，因此，启动类加载器无法被Java程序调用。启动类加载器不像其他类加载器有实体，它是没有实体的，JVM将C++处理类加载的一套逻辑定义为启动类加载器
<span><div style="text-align: center;">
![jvm-6](../picture/JVM/jvm-6.png)
![jvm-7](../picture/JVM/jvm-7.png)
![jvm-8](../picture/JVM/jvm-8.png)
![jvm-9](../picture/JVM/jvm-9.png)
![jvm-10](../picture/JVM/jvm-10.png)
![jvm-11](../picture/JVM/jvm-11.png)
</div></span>
##2.	扩展类加载器ExtClassLoader
负责加载JRE扩展目录ext中的jar包
<span><div style="text-align: center;">
![jvm-12](../picture/JVM/jvm-12.png)
</div></span>
##3. 应用类加载器AppClassLoader
负责加载classPath路径下的类包
<span><div style="text-align: center;">
![jvm-13](../picture/JVM/jvm-13.png)
</div></span>
##4. 自定义类记载器
负责加载用户自定义下的类包，实现自定义类加载器：extends ClassLoader，重写loadClass方法进行类的加载
<span><div style="text-align: center;">
![jvm-14](../picture/JVM/jvm-14.png)
</div></span>
##5.	 JVM中类加载的类如何存储？
<span><div style="text-align: center;">
![jvm-15](../picture/JVM/jvm-15.png)
</div></span>
##6.JVM中类加载机制哪些？
JVM的类加载机制主要有如下3种：

- **全盘负责：**所谓全盘负责，就是当一个类加载器负责加载某个Class时，该Class所依赖和引用其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
  
- **双亲委派：**所谓的双亲委派，则是先让父类加载器试图加载该Class，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父加载器，依次递归，如果父加载器可以完成类加载任务，就成功返回；只有父加载器无法完成此加载任务时，才自己去加载。
双亲委派机制，其工作原理的是：如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式，即每个儿子都很懒，每次有活就丢给父亲去干，直到父亲说这件事我也干不了时，儿子自己才想办法去完成
双亲委派机制的优势：采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。其次是考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改 

- **缓存机制：**缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为很么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。
  
- **双亲委派有啥好处呢？**
它使得类有了层次的划分。就拿java.lang.Object来说，你加载它经过一层层委托最终是由Bootstrap ClassLoader来加载的，也就是最终都是由Bootstrap ClassLoader去找<JAVA_HOME>\lib中rt.jar里面的java.lang.Object加载到JVM中。
这样如果有不法分子自己造了个java.lang.Object,里面嵌了不好的代码，如果我们是按照双亲委派模型来实现的话，最终加载到JVM中的只会是我们rt.jar里面的东西，也就是这些核心的基础类代码得到了保护。因为这个机制使得系统中只会出现一个java.lang.Object。不会乱套了。你想想如果我们JVM里面有两个Object,那岂不是天下大乱了。

**如何打破双亲委派？**
- 1：自己写一个类加载器
- 2：重写loadclass方法
- 3：重写findclass方法
  
这里最主要的是重写loadclass方法，因为双亲委派机制的实现都是通过这个方法实现的，先找附加在其进行加载，如果父加载器无法加载再由自己来进行加载，源码里会直接找到根加载器，重写了这个方法以后就能自己定义加载的方式了

#4.	JVM中PC寄存器、方法区、本地方法区、堆、栈的理解？
##1） 栈
- Java栈的区域很小，只有1M，特点是存取速度很快，所以在stack中存放的都是快速执行的任务，基本数据类型的数据，和对象的引用（reference）。
- 栈内存主管程序的运行，生命周期和线程同步，线程结束，栈内存释放。对于栈来说不存在垃圾回收。栈存放：8大借本类型(整型：byte, short, int, long 字符型：char  浮点型：float, double 布尔型：boolean)+对象引用+实例的方法
- JVM只会直接对JavaStack（Java栈）执行两种操作：①以帧为单位的压栈或出栈；②通过-Xss来设置，若不够会抛出StackOverflowError异常。
- 1.每个线程包含一个栈区，栈中只保存基本数据类型的数据和自定义对象的引用(不是对象)，对象都存放在堆区中
- 2.每个栈中的数据(原始类型和对象引用)都是私有的，其他栈不能访问。
<span><div style="text-align: center;">
![jvm-16](../picture/JVM/jvm-16.png)
</div></span>

- **栈帧**:JVM Stack（Stack 或虚拟机栈、线程栈、栈）中存放的就是栈帧、方法栈（栈帧包括了局部变量表+操作数栈+动态链接+返回地址）。
局部变量表：局部变量表（Local Variables Table）也可以称之为本地变量表，它包含在一个独立的栈帧中。顾名思义，局部变量表主要用于存储方法参数和定义在方法体内的局部变量
- **操作数栈**：每一个独立的栈帧中除了包含局部变量表以外，还包含一个后进先出（Last-In-First-Out）的操作数栈，也可以称之为表达式栈（Expression Stack）。操作数栈和局部变量表在访问方式上存在着较大差异，操作数栈并非采用访问索引的方式来进行数据访问的，而是通过标准的入栈和出栈操作来完成一次数据访问。每一个操作数栈都会拥有一个明确的栈深度用于存储数值，一个32bit的数值可以用一个单位的栈深度来存储，而2个单位的栈深度则可以保存一个64bit的数值，当然操作数栈所需的容量大小在编译期就可以被完全确定下来，并保存在方法的Code属性中。
- **动态链接**：每一个栈帧内部除了包含局部变量表和操作数栈之外，还包含一个指向运行时常量池中该栈帧所属方法的引用，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接（Dynamic Linking）。
在运行时常量池，一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含一项信息，那就是常量池表（Constant Pool Table），那么运行时常量池就是字节码文件中常量池表的运行时表示形式。在一个字节码文件中，描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用（Symbolic Reference）来表示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。
- **返回地址**：一个方法在执行的过程中将会产生两种调用结果：一种是方法正常调用完成，而另外一种则是方法异常调用完成。如果是方法正常调用完成，那么这就意味着，被调用的当前方法在执行的过程中将不会有任何的异常被抛出，并且方法在执行的过程中一旦遇见字节码返回指令时，将会把方法的返回值返回给它的调用者，不过一个方法在正常调用完成之后究竟需要使用哪一个返回指令还需要根据方法返回值的实际数据类型而定。
在字节码指令中，返回指令包含ireturn（当返回值是boolean、byte、char、short和int类型时使用）、lreturn(从当前方法返回long)、freturn(从当前方法返回float)、dreturn(从当前方法返回double)以及areturn(从当前方法返回对象引用)，另外还有一个return指令供声明为void的方法、实例初始化方法、类和接口的初始化方法使用。
<span><div style="text-align: center;">
![jvm-17](../picture/JVM/jvm-17.png)
</div></span>

**JVM指令中ireturn作用**：
1.局部变量表指针恢复成Main方法的局部变量表指针 
2. 操作数栈方法的当前指针恢复成Main方法的操作数栈方法指针
3需要将运行时数据区中的程序计数器改成invokervirtual指令
4.将add方法中所占用的栈帧内存全部回收

##2） 堆
- 类的对象放在heap（堆）中，所有的类对象都是通过new方法创建，创建后，在stack（栈）会创建类对象的引用（内存地址）。
- JVM将所有对象的实例（即用new创建的对象）（对应于对象的引用（引用就是内存地址））的内存都分配在堆上，堆所占内存的大小由-Xmx指令和-Xms指令来调节。
- 加上JVM参数-verbose:gc -Xms10M -Xmx10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+HeapDumpOnOutOfMemoryError，就能很快报出OOM异常（内存溢出异常）：Exception in thread "main" java.lang.OutOfMemoryError: Java heap space并且能自动生成Dump

<span><div style="text-align: center;">
![jvm-18](../picture/JVM/jvm-18.png)
</div></span>

##3） 方法区
- method（方法区）又叫静态区，存放所有的①类（class），②静态变量（static变量），③静态方法，④常量和⑤成员方法。
- 方法区又叫静态区，跟堆一样，被所有的线程共享。方法区中存放的都是在整个程序中永远唯一的元素。这也是方法区被所有的线程共享的原因
- 虚拟机的体系结构：①Java栈，② 堆，③PC寄存器，④方法区，⑤本地方法栈，⑥运行常量池。而方法区保存的就是一个类的模板，堆是放类的实例（即对象）的。
- 方法区的大小由-XX:PermSize和-XX:MaxPermSize来调节，类太多有可能撑爆永久代。静态变量或常量也有可能撑爆方法区。
  
<span><div style="text-align: center;">
![jvm-19](../picture/JVM/jvm-19.png)
</div></span>

##4）PC寄存器
- PC寄存器（ PC register ）：每个线程启动的时候，都会创建一个PC（Program Counter，程序计数器）寄存器。
- PC寄存器里保存有当前正在执行的JVM指令的地址。 每一个线程都有它自己的PC寄存器，也是该线程启动时创建的。保存下一条将要执行的指令地址的寄存器是：PC寄存器。
- PC寄存器的内容总是指向下一条将被执行指令的地址，这里的地址可以是一个本地指针，也可以是在方法区中相对应于该方法起始指令的偏移量。

##5）本地方法栈区
- 保存native方法进入区域的地址

#5.JVM中对象实例化在内存中的过程？
**简单类对象的实例化过程：**
1、在方法区加载类；
2、在栈内存申请空间，声明变量P；
3、在堆内存中开辟空间，分配对象地址；
4、在对象空间中，对对象的属性进行默认初始化，类成员变量显示初始化；
5、构造方法进栈，进行初始化；
6、初始化完成后，将堆内存中的地址赋给引用变量，构造方法出栈；

**子类对象的实例化过程：**
1、在方法区先加载父类，再加载子类；
2、在栈中申请空间，声明变量P；
3、在堆内存中开辟空间，分配对象地址；
4、在对象空间中，对对象的属性（包括父类的属性）进行默认初始化；
5、子类构造方法进栈；
6、显示初始化父类的属性；
7、父类构造方法进栈，执行完毕出栈；
8、显示初始化子类的属性；
9、初始化完毕后，将堆内存中的地址值赋给引用变量P，子类构造方法出栈；

#6.	堆内存为什么分为新生代老年代？
因为以前的垃圾回收器是基于分代理念来实现的，所以在需要将对分成两个代。
为什么新生代占堆内存1/3，老年代占2/3？
老年代：存储大对象(对象的大小超过edlen区1/2)、经历15次GC还存在的对象、备用空间(空间担保)
<span><div style="text-align: center;">
![jvm-19](../picture/JVM/jvm-20.png)
</div></span>
因为有的对象寿命长，有的对象寿命短。应该将寿命长的对象放在一个区，寿命短的对象放在一个区。不同的区采用不同的垃圾收集算法。寿命短的区清理频次高一点，寿命长的区清理频次低一点。提高效率

**为什么要有Survivor区？**
如果没有Survivor区，那么Eden每次满了清理垃圾，存活的对象被迁移到老年区，老年区满了，就会触发Full GC，Full GC是非常耗时的
**解决办法：**
增加老年代内存，那么老年代清理频次减少，但清理一次花费时间更长。
减少老年代内存，老年代一次FullGC时间更少，频率增加。
都不行，只有再加一层Survivor。将Eden区满了的对象，添加到Survivor区，等对象反复清理几遍之后都没清理掉，再放到老年区，这样老年区的压力就会小很多。即Survivor相当于一个筛子，筛掉生命周期短的，将生命周期长的放到老年代区，减少老年代被清理的次数。

#7.元空间(JAVA8)和永久代(JAVA7)的区别？
<span><div style="text-align: center;">
![jvm-21](../picture/JVM/jvm-21.png)
</div></span>

#8.	谈谈JVM中堆内存的理解？
**一个jvm只有一个堆内存，堆内存的大小是可以调节的，类加载器读取了类文件后，一般会把类、方法、常量、变量保存我们所有引用类型的真实变量。**
堆内存中细分为三个区域：
##1.	新生区(伊甸园区、幸存0区、幸存1区) young/new
特点：
新生代又可以进一步划分为一个Eden区和两个Survivor区，Eden是进行内存分配的地方，是一块连续的空闲内存区域，在里面进行内存分配速度非常快，因为不需要进行可用内存块的查找。新对象总是在Eden区中生成，只有经受住了一定的考验后才能顺利的进入到Survivor区中。把Survivor区划分为2块，也是也是为了满足垃圾回收的需要，因为在新生代中经历了回收未必就能进入老年代中。系统总是把对象放在Eden区和一个Survivor区，在垃圾回收时，根据其存活时间被复制到另一个Survivor区或者老年代中，则这之前的Survivor去和Eden区中剩下的都是需要被回收的对象，只对这两个区域进行清除即可，两个Survivor区是交替使用，循环往复，在下一次垃圾回收时，之前被清除的存活区又用来放置存活下来的对象了。一般来说，年轻代区域较小，而且大部分对象是需要进行清除的，采用了"复制算法"进行垃圾回收

##2.	养老区 old
特点：
在新生代中经历了N次回收后仍然没有被清除的对象，就会被放到老年代中，都是生命周期较长的对象。对于老年代和永久代，采用标记-整理的算法。标记的过程是找出当前还存活对对象，并进行标记；清除则是遍历整个老年区，找已经标记的对象并进行清除，然后把存活的对象移动到整个内存区的一端，使得另一端是一块连续的空间，方便进行内存分配和复制。

  - Minor GC：当新对象生成，但在Eden申请空间失败时就会触发，对Eden进行GC，清除掉非存活的对象，并且把存活的对象移动到Survivor区中的其中一块。前面提到的考验就是Minor GC，也就是说对象经过了Minor GC才能够进入到存活区中。这种形式的GC只会在新生代中进行，因为大部分对象都是从Eden区开始的，同时Eden不会分配的太大，所以对Eden区的GC会非常地频繁。
  - Full GC：对整个堆进行整理，包括了新生代老年代和永久代。Full GC要对整个块进行回收，所以要比Minor GC慢很多，因此应该尽可能的减少Full GC的次数。
  GC垃圾回收主要是在伊甸园区和养老区。假设内存满了会抛出OOM异常，堆内存不够
  在JDK8以后，永久存储区改名为（元空间）
  <span><div style="text-align: center;">
![jvm-22](../picture/JVM/jvm-22.png)
</div></span>

##3.永久区 perm
  <span><div style="text-align: center;">
![jvm-23](../picture/JVM/jvm-23.png)
</div></span>

#9.	谈什么是GCRoot？
GCRoot是过一系列名为“GC Roots”的对象作为起始点，从“GC Roots”对象开始向下搜索，如果一个对象到“GC Roots”没有任何引用链相连，说明此对象可以被回收。
<span><div style="text-align: center;">
![jvm-24](../picture/JVM/jvm-24.png)
</div></span>

#10.谈谈JVM中的轻Minor GC、Major GC、Major GC？
新生成的对象首先放到年轻代Eden区，当Eden空间满了，触发Minor GC，存活下来的对象移动到Survivor0区，Survivor0区满后触发执行Minor GC，Survivor0区存活对象移动到Suvivor1区，这样保证了一段时间内总有一个survivor区为空。经过多次Minor GC仍然存活的对象移动到老年代。
老年代存储长期存活的对象，占满时会触发Major GC=Full GC，GC期间会停止所有线程等待GC完成，所以对响应要求高的应用尽量减少发生Major GC，避免响应超时。
Minor GC ： 清理年轻代 ()
Major GC ： 清理老年代
Full GC ： 清理整个堆空间，包括年轻代和永久代（jdk.1,8中无永久代）
所有GC都会停止应用所有线程
<span><div style="text-align: center;">
![jvm-25](../picture/JVM/jvm-25.png)
![jvm-26](../picture/JVM/jvm-26.png)
![jvm-27](../picture/JVM/jvm-27.png)
![jvm-28](../picture/JVM/jvm-28.png)
</div></span>

#11.谈谈JVM中垃圾回收算法？
##1）引用计数法
原理：假设有一个对象A，任何一个对象对A的引用，那么对象A的引用计数器+1，当引用失败时，对象A的引用计数器就-1，如果对象A的计数器的值为0，那说明对象A没有引用了，可以被回收。
<span><div style="text-align: center;">
![jvm-29](../picture/JVM/jvm-29.png)
</div></span>
##2）复制算法(年轻代主要使用算法)
原理：复制算法的核心就是，将原有的内存空间一分为二，每次只用其中的一块，在垃圾回收时，将正在使用的对象复制到另外一个内存空间中，然后将该内存空间清空，交换两个内存的角色，完成垃圾回收。
优点：在垃圾对象多的情况下，效率较高。清理后，内存无碎片。
缺点：在垃圾对象少的情况下，不适用，如：老年代内存。分配的2块内存空间，在同一个时刻，只能使用一般，内存使用率较低。
最佳使用场景：存活度较低的地方（新生区）

<span><div style="text-align: center;">
![jvm-30](../picture/JVM/jvm-30.png)
</div></span>

-XX:MaxTenuringThreshold 设置对象在新生代中最大的存活次数,最大值15,并行回收机制默认为15,CMS默认为4。每经过一次YGC，年龄加1，当survivor区的对象年龄达到TenuringThreshold时，表示该对象是长存活对象，就会直接晋升到老年代

<span><div style="text-align: center;">
![jvm-31](../picture/JVM/jvm-31.png)
</div></span>

##3）标记清除法
原理：
“标记-清除”算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。
优点：不需要额外的空间；
缺点：
（1）效率问题：两次扫描，标记和清除过程的效率都不高；
（2）空间问题：标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，碎片过多会导致大对象无法分配到足够的连续内存，从而不得不提前触发GC。

<span><div style="text-align: center;">
![jvm-32](../picture/JVM/jvm-32.png)
</div></span>

##4）标记压缩（整理）法
原理：根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

<span><div style="text-align: center;">
![jvm-33](../picture/JVM/jvm-33.png)
![jvm-34](../picture/JVM/jvm-34.png)
</div></span>

#12.GC垃圾收集器种类
<span><div style="text-align: center;">
![jvm-35](../picture/JVM/jvm-35.png)
![jvm-36](../picture/JVM/jvm-36.png)
![jvm-37](../picture/JVM/jvm-37.png)
![jvm-38](../picture/JVM/jvm-38.png)
![jvm-39](../picture/JVM/jvm-39.png)
![jvm-40](../picture/JVM/jvm-40.png)
![jvm-41](../picture/JVM/jvm-41.png)
![jvm-42](../picture/JVM/jvm-42.png)
![jvm-43](../picture/JVM/jvm-43.png)
![jvm-44](../picture/JVM/jvm-44.png)
![jvm-45](../picture/JVM/jvm-45.png)
![jvm-46](../picture/JVM/jvm-46.png)
![jvm-47](../picture/JVM/jvm-47.png)
![jvm-48](../picture/JVM/jvm-48.png)
![jvm-49](../picture/JVM/jvm-49.png)
![jvm-50](../picture/JVM/jvm-50.png)
![jvm-51](../picture/JVM/jvm-51.png)
![jvm-52](../picture/JVM/jvm-52.png)
![jvm-53](../picture/JVM/jvm-53.png)

</div></span>

#13.什么是G1垃圾收集器？
G1垃圾收集器：整体上采用标记-整理算法、局部采用复制算法不会产生内存碎片
<span><div style="text-align: center;">
![jvm-54](../picture/JVM/jvm-54.png)
![jvm-55](../picture/JVM/jvm-55.png)
![jvm-56](../picture/JVM/jvm-56.png)
![jvm-57](../picture/JVM/jvm-57.png)
![jvm-58](../picture/JVM/jvm-58.png)
![jvm-59](../picture/JVM/jvm-59.png)
![jvm-60](../picture/JVM/jvm-60.png)
</div></span>

G1垃圾收集器的四个步骤：
<span><div style="text-align: center;">
![jvm-61](../picture/JVM/jvm-61.png)
![jvm-62](../picture/JVM/jvm-62.png)
</div></span>