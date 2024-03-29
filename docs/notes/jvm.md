## 介绍JVM中7个区域，然后把每个区域可能造成内存的溢出的情况说明

- **程序计数器**：看做当前线程所执行的**字节码行号指示器**。是线程**私有**的内存，且唯一一块不报OutOfMemoryError异常的内存区域。

- **Java虚拟机栈**：用于描述java方法的**内存模型**：每个方法被执行时都会同时创建一个**栈帧**用于存储**局部变量表、操作数栈、动态链接、方法出口**等信息。每一个方法被调用直至执行完成的过程就对应着一个栈帧在虚拟机中从入栈到出栈的过程。如果线程请求的**栈深度**大于虚拟机所允许的深度就报StackOverflowError, 如果虚拟机栈可以动态**扩展**，当拓展时无法申请到足够的内存会抛出OutOfMemoryError. 是线程**私有**的。

- **本地方法栈**：与虚拟机栈相似，不同的在于它是为虚拟机使用到**Native**方法服务的。会抛出StackOverflowError和OutOfMemoryError。是线程**私有**的。

- **Java堆**: 是所有线程共享的一块内存，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。如果堆上没有内存完成实例的分配就会报OutOfMemoryError.

- **方法区（永久代）**：用于存储已被虚拟机加载的**类信息、常量、静态变量、即时编译器编译后的代码**等数据。当方法区无法满足内存分配需求时，会抛出OutOfMemoryError。是共享内存。

- **运行时常量池**：用于存放编译器生成的各种字面量和符号引用，是方法区的一部分。无法申请内存时抛出OutOfMemoryError。

- **直接内存**：不是虚拟机运行时数据的一部分，也不是java虚拟机规范中定义的区域，是计算机直接的内存空间。这部分也被频繁使用，如JAVA NIO的引入基于通道和缓存区的I/O使用native函数直接分配堆外内存。如果内存不足会报OutOfMemoryError。

## GC的两种判定方法：引用计数与根搜索算法

- **引用计数**： 给对象添加一个引用计数器，每当有一个地方引用该对象时，计数器值加1，当引用失效时，计数器值减1,。任何时候计数器都为0的对象就是不可能再被使用的。它很难解决对象之间相互**循环引用**问题。

- **根搜索算法（GC Roots Traceing）:** 通过一系列名为“GC Roots”的对象作为起点，从这些节点开始向下搜索，搜索走过的路径成为**引用链**，当一个对象到GC Roots没有任何引用链相连时，则证明此对象不可用。

  GC Roots对象一般是：虚拟机栈中的引用对象，方法区中类静态属性引用的对象，方法区常量引用的对象等。

## Java中的四种引用
Java中提供这四种引用类型主要有两个目的：第一是可以让程序员通过代码的方式决定某些对象的生命周期；第二是有利于JVM进行垃圾回收。

- **强引用**：程序代码中的普通引用。如Object obj = new Object(),只要强引用存在，垃圾回收器就不会回收。在不使用对象时应及时将引用设置为null，便于垃圾回收。

- **软引用**：描述一些有用但并非必须的对象。对于软引用关联的对象在系统将要**发生内存溢出异常之前**，将会把这些对象列进回收范围之中进行第二次回收。**SoftRefence**

- **弱引用**：描述非必须对象，比软引用弱一些。被弱引用关联的对象只能**生存到下一次垃圾收集发生之前**。无论当前内存是否足够，都会回收掉只被弱引用关联的对象。**WeakRefence**

- **虚引用**：最弱的引用，不管是否有虚引用存在，完全不会对对象生存时间构成影响，也无法通过虚引用来取得一个对象实例。唯一目的是希望能够在这个对象被垃圾回收器之前收到系统通知。**PhantomReference**

相关参考：

[Java 如何有效地避免OOM：善于利用软引用和弱引用](https://www.cnblogs.com/dolphin0520/p/3784171.html)

[深入理解JDK中的Reference原理和源码实现](https://www.cnblogs.com/throwable/p/12271653.html)

## 对象创建方法，对象的内存分配，对象的访问定位。

```
Object obj = new Object();
```

obj 保存在java栈中的局部变量表里，作为一个引用数据出现。 New Object()会在java堆上分配一块存储Object类型实例的所有数值的结构化内存，根据类型以及虚拟机实现的对象内存布局不同。这块内存是不固定的。

对象访问方式有两种：**句柄和直接指针**。

- **句柄**：在java堆中会划分出一块内存作为句柄池，reference中存储的对象是句柄地址。**而句柄中包含对象实例数据和类型数据各自的具体地址信息**。最大的好处是如果对象地址发生变化不需要改变reference的值，只需要改变句柄中实例数据指针。

- **直接指针访问**：reference直接存储对象的地址，最大的好处是**速度更快**。

## 内存溢出和内存泄漏
- **内存溢出**：通俗理解就是**内存不够**，程序所需要的内存远远超出了你虚拟机分配的内存大小，就叫内存溢出

- **内存泄露**：内存泄漏也称作“存储渗漏”，用动态存储分配函数动态开辟的空间，在**使用完毕后未释放**，结果导致**一直占据该内存单元**。直到程序结束。（其实说白了就是该内存空间使用完毕之后未回收）即所谓内存泄漏

## 内存溢出了怎么办
通过内存映像工具如jhat、jconsole等对dump出来的堆转存储快照进行分析，重点是确认内存是出现内存泄露还是内存溢出。

如果是**内存泄露**进一步使用工具查看泄露的对象到GC Roots的引用链。于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们。掌握泄露对象的信息，以及GC Roots引用链的信息，就可以比较准确定位泄露代码的位置。

如果不存在**内存泄露**，那就需要通过jinfo、Jconsole等工具分析java堆参数与机器物理内存对比是否还可以调大，从代码上检查是否存在某些对象生命周期过长，持有状态过长的情况，尝试减少程序的运行消耗。

## Java 中有内存泄露吗？
有，Java中，造成内存泄露的原因有很多种。典型的例子是**长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露**，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被回收，这就是java中内存泄露的发生场景，通俗地说，就是程序员可能创建了一个对象，以后一直不再使用这个对象，这个对象却一直被引用，即这个对象无用但是却无法被垃圾回收器回收的，这就是java中可能出现内存泄露的情况，例如，缓存系统，我们加载了一个对象放在缓存中(例如放在一个全局map对象中)，然后一直不再使用它，这个对象一直被缓存引用，但却不再被使用。

检查java中的内存泄露，一定要让程序将各种分支情况都完整执行到程序结束，然后看某个对象是否被使用过，如果没有，则才能判定这个对象属于内存泄露。（采用什么工具？）

**如果一个外部类的实例对象的方法返回了一个内部类的实例对象**，这个内部类对象被长期引用了，即使那个外部类实例对象不再被使用，但由于内部类持久外部类的实例对象，这个外部类对象将不会被垃圾回收，这也会造成内存泄露。

http://www.mamicode.com/info-detail-504269.html

## 什么时候会发生jvm堆（持久区）内存溢出
简单的来说 java的堆内存分为两块:permantspace（持久代） 和 heap space。

持久带中主要存放用于存放静态类型数据，如 Java Class, Method 等， 与垃圾收集器要收集的Java对象关系不大。

而heapspace分为年轻代和年老代:

 - 年轻代的垃圾回收叫 Young GC， 年老代的垃圾回收叫 Full GC。
 - 在年轻代中经历了N次（可配置）垃圾回收后仍然存活的对象，就会被复制到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象
 - 年老代溢出原因有  循环上万次的字符串处理、创建上千万个对象、在一段代码内申请上百M甚至上G的内存

**持久代溢出原因动态加载了大量Java类而导致溢出，以及生产大量的常量**。 

**永久代内存泄露**: 以一个部署到应用程序服务器的Java web程序来说，当该应用程序被卸载的时候，你的EAR/WAR包中的所有类都将变得无用。只要应用程序服务器还活着，JVM将继续运行，但是一大堆的类定义将不再使用，理应将它们从永久代（PermGen）中移除。如果不移除的话，我们在永久代（PermGen）区域就会有内存泄漏。

## 堆里面的分区：Eden，survivor from to，老年代，各自的特点。
新生代：朝生夕死

老年代一般是放对象和长期存活对象。当一个对象分配的内存空间大于某个阈值时或则年龄增加到一定程度（默认15岁）就进入老年代。

## OOM你遇到过哪些情况
- java.lang.OutOfMemoryError: Java heap space ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。 

- java.lang.OutOfMemoryError: PermGen space ------>java永久代溢出，即方法区溢出了，**一般出现于大量Class或者jsp页面**，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。 

- java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在**死循环或者深度递归调用**造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数**-Xss**来设置栈的大小。

## GC的收集方法的原理与特点，分别用在什么地方，如果让你优化收集方法，有什么思路？
- **标记清理**：首先标记所有需要回收的对象，在标记完成后**统一回收掉**所有被标记的对象，它的标记的对象。缺点是**效率低**，且存在**内存碎片**。主要用于老生代垃圾回收。

- **标记整理**：首先标记所有需要回收的对象，在标记完成后让所有存活的对象都向一端移动，然后直接清理掉端边界意外的内存。用于老年代。

- **复制算法**：将内存按容量划分为大小相等的一块，每次只用其中一块。当内存用完了，将还存活的对象复制到另一块内存，然后把已使用过的内存空间一次清理掉。实现简单，高效。一般用于新生代。一般是将内存分为一块较大的**Eden空间**和两块较小的**Survivor**空间。HotSpot虚拟机默认比例是**8:1**,。每次使用Eden和一块Survivor，当回收时将这两块内存中还存活的对象复制到Survivor然后清理掉刚才Eden和Survivor的空间。如果复制过程内存不够使用则向老年代分配担保。

- **分代收集算法**：根据对象的生存周期将内存划分为新生代和老年代，根据年代的特点采用最适当的收集算法。

## GC收集器有哪些？CMS收集器与G1收集器的特点。
- **Serial**: 单线程收集器，只会使用一个CPU或一条收集器线程去完成，垃圾回收工作，更重要的是在进行垃圾回收时，必须暂停其他所有的工作线程。（Stop the world）。简单高效，用于新生代。

- **ParNew**: 是Serial收集器的**多线程版本**，垃圾回收时采用多线程方式进行回收。默认情况下使用的线程数是cpu数量。除了serial收集器，目前只有它能和CMS收集器配合工作。是server模式下首选的新生代收集器。

- **Parallel Scavenge**: 使用**复制算法**收集器，也是一个并行的多线程收集器。Parallel Scavenge收集器与其他收集器关注点不同，其它收集器主要关注缩短垃圾回收时用户线程的停顿时间。而它关心**吞吐量**，即**运行用户代码时间/(运行用户代码时间 + 垃圾收集时间)**。停顿时间越短越适合需要与用户交互的程序，高吞吐量则可以最高效率的利用CPU时间。

- **Serial Old**: 老年代，单线程收集器，使用**标记整理算法**。主要有两个用途，一是和Parallel Scavenge 收集器配合使用，二是作为CMS的后备方案在并发收集器发生**Concurrent Mode Failure**时候使用。

- **Parallel Old**:并行的老年代版本收集器，使用标记整理算法。主要与Parallel Scavenge配合使用。

- **CMS**：是以获得**最短回收停顿时间为**目标的收集器，使用**标记清除算法**。整个过程包括4个：
   - **初始标记**: 标记Gc ROOTS能直接关联到的对象
   - **并发标记**：进行Roots Traceing的过程
   - **重新标记**：修正并发标记期间因用户继续工作导致标记产生变动
   - **并发清除**：并发清除数据。
   初始标记和重新标记需要stop the world. 并发标记和并发清除过程用户线程和收集器线程可以并行执行。

- **G1(Garbage First):** 基于**标记-整理算法**的收集器,不会产生空间碎片.它可以精确控制停顿,能够让使用者明确指定一个长度为M毫秒的时间片段内,消耗集上的时间不超过N秒.是不牺牲吞吐量的前提下完成低停顿的.**G1将整个java堆(新生和老生)划分为大小相同的区,并跟踪这些区上发生的变化.在后台维护一个优先列表,每次根据允许的收集时间优先回收垃圾最多的区域**.

现在公司中很多都采用了G1 垃圾回收期，建议大家多深入了解下G1，更多参考: [G1垃圾回收器](./G1垃圾回收器.md)

## Minor GC与Full GC分别在什么时候发生？
FullGC 一般是发生在老年代的GC，出现一个FullGC经常会伴随至少一次的Minor GC。速度比MinorGC慢10倍以上。

### FUll GC
FULL GC发生的情况:

- **1) 老年代空间不足**
老年代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误：java.lang.OutOfMemoryError: Java heap space.

措施:为避免以上两种状况引起的FullGC，调优时应尽量做到让对象在Minor GC阶段被回收、让对象在新生代多存活一段时间及不要创建过大的对象及数组

- **2) Permanet Generation(方法区或永久代)空间满**
 PermanetGeneration中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，Permanet Generation可能会被占满，在未配置为采用CMS GC的情况下会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息：java.lang.OutOfMemoryError: PermGen space 

 措施:为避免Perm Gen占满造成Full GC现象，可采用的方法为增大Perm Gen空间或转为使用CMS GC。

- **3) CMS GC时出现promotion failed和concurrent mode failure**
  对于采用CMS进行老年代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入老年代，而此时老年代也放不下造成的；

  concurrent mode failure: CMS在执行垃圾回收时需要一部分的内存空间并且此刻用户程序也在运行需要预留一部分内存给用户程序，如果预留的内存无法满足程序需求就出现一次"Concurrent mod failure",并触发一次Full GC。
  
  应对措施为：增大survivor space、老年代空间或调低触发并发GC的比率，

- **4) 空间分配担保**
统计得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间，Hotspot为了避免由于新生代对象晋升到老年代导致旧生代空间不足的现象，在进行Minor GC时，做了一个判断。如果之前统计所得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间，那么就直接触发Full GC。如果小于并且不允许担保失败也会发生一次Full GC。


### MinorGC

MinorGC 指发生在新生代的垃圾收集动作，非常频繁，回收速度也快。一般发生在新生代空间不足时,另外一个FullGC经常会伴随至少一次的Minor GC. 当虚拟检测晋升到到老年代的平均大小是否小于老年代剩余空间大小,如果小于并且允许担保失败,则执行Minor GC.


## 几种常用的内存调试工具：jmap、jstack、jconsole。
(如何用工具分析jvm状态)

- **jps**: 列出正在虚拟机运行的虚拟机进程，并显示虚拟机执行主类的名称，以及这些进程的本地虚拟机的唯一ID。
- **jstat** : 监视虚拟机各种运行状态信息的命令。可以显示本地或远程虚拟机进程中**类装载、垃圾收集、JIT编译、内存**等数据。
- **jinfo**: 实时查看和调整虚拟机的各项参数。
- **jmap**: 生成**堆转存储快照**，查询fianlize执行队列、java堆和永生代详细信息，如空间使用率，当前用的是那种收集器。
- **Jhat**: 和jmap搭配使用，来分析jmap生成的堆转存储快照。内置一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，可以通过浏览器查看。
- **jstack**:用于生成当前时刻**线程快照**.线程快照是当前虚拟机内每一条线程正在执行的方法堆栈的集合.生成线程快照的主要目的是为了定位线程长时间停顿的原因.如死锁、死循环、请求外部资源导致的长时间等待.
- **JConsole**: 可视化监视和管理工具,几乎包括以上工具的所有功能
- **VisualVM**

## GC 是什么？为什么要有 GC
GC 是垃圾收集的意思，内存处理是编程人员容易出现问题的地方，忘记或者错误的内存回收会导致程序或系统的不稳定甚至崩溃，Java 提供的 GC 功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的，Java 语言没有提供释放已分配内存的显示操作方法。Java 程序员不用担心内存管理，因为垃圾收集器会自动进行管理。要请求垃圾收集，可以调用下面的方法之一：System.gc() 或Runtime.getRuntime().gc() ，但 JVM 可以屏蔽掉显示的垃圾回收调用。

垃圾回收可以有效的防止内存泄露，有效的使用可以使用的内存。垃圾回收器通常是作为一个单独的低优先级的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收，程序员不能实时的调用垃圾回收器对某个对象或所有对象进行垃圾回收。在 Java 诞生初期，垃圾回收是 Java 最大的亮点之一，因为服务器端的编程需要有效的防止内存泄露问题，然而时过境迁，如今 Java 的垃圾回收机制已经成为被诟病的东西。移动智能终端用户通常觉得 iOS 的系统比 Android 系统有更好的用户体验，其中一个深层次的原因就在于 Android 系统中垃圾回收的不可预知性。

##  JVM 加载 class 文件的原理机制
JVM 中类的装载是由类加载器（ClassLoader） 和它的子类来实现的，Java 中的类加载器是一个重要的 Java 运行时系统组件，它负责在运行时查找和装入类文件中的类。

由于 Java 的跨平台性，经过编译的 Java 源程序并不是一个可执行程序，而是一个或多个类文件。当 Java 程序需要使用某个类时，JVM 会确保这个类已经被加载、连接(验证、准备和解析)和初始化。类的加载是指把类的 .class 文件中的数据读入到内存中，通常是创建一个字节数组读入 .class 文件，然后产生与所加载类对应的 Class 对象。加载完成后，Class 对象还不完整，所以此时的类还不可用。当类被加载后就进入连接阶段，这一阶段包括验证、准备(为静态变量分配内存并设置默认的初始值)和解析(将符号引用替换为直接引用)三个步骤。最后 JVM 对类进行初始化，包括：

1. 如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；
2. 如果类中存在初始化语句，就依次执行这些初始化语句。

类的加载是由类加载器完成的，类加载器包括：**启动类加载器（BootStrap）、扩展加载器（Extension）、应用程序加载器（Application）和用户自定义类加载器（java.lang.ClassLoader的子类）**。从JDK 1.2开始，类加载过程采取了父亲委托机制(PDM)。PDM 更好的保证了 Java 平台的安全性，在该机制中，JVM 自带的 Bootstrap 是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。JVM 不会向 Java 程序提供对 Bootstrap 的引用。下面是关于几个类加载器的说明：

- **Bootstrap**：启动类加载器，一般用本地代码实现，负责加载JVM基础核心类库。加载存放在<JAVA_HOME>/lib目录中的类库（如rt.jar）；
- **Extension ClassLoader**：扩展加载器， 负责加载<JAVA_HOME>/lib/ext目录中的
，或被java.ext.dirs 系统属性所指定的目录中加载类库，它的父加载器是 Bootstrap；
- **Application ClassLoader**：应用程序加载器，其父类是Extension。它是应用最广泛的类加载器。它从环境变量 classpath 或者系统属性 java.class.path 所指定的目录中记载类，是用户自定义加载器的默认父加载器。

缺点: 

- 双亲委派模型很好地解决了各个类加载器的基础类统一问题(越基础的类由越上层的加载器进行加载)，基础类之所以被称为“基础”，是因为它们总是作为被调用代码调用的API。但是，如果基础类又要调用用户的代码时，双亲委派模型无法满足要求。 因为Bootstrap加载器无法找到永不代码类。

为了解决这个困境，Java设计团队只好引入了一个不太优雅的设计：**线程上下文件类加载器(Thread Context ClassLoader)**。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个；如果在应用程序的全局范围内都没有设置过，那么这个类加载器默认就是应用程序类加载器。了有线程上下文类加载器，JNDI服务使用这个线程上下文类加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载动作，这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型，但这也是无可奈何的事情。**Java中所有涉及SPI的加载动作基本上都采用这种方式**，例如JNDI,JDBC,JCE,JAXB和JBI等。 Dubbo的SPI也是采用这种机制实现。

## 类加载的五个过程：加载、验证、准备、解析、初始化。
- **加载**: 根据全限定名来获取定义类的二进制字节流,然后将该字节流所代表的静态结构转化为方法区的运行时数据结构,最后在生成一个代表该类的Class对象,作为方法区这些数据的访问入口.

- **验证**:主要时为了确保class文件的字节流中包含的信息符合当前虚拟机的要求,并且不会危害虚拟机自身的安全.包含四个阶段的验证过程:
  - **文件格式验证**:保证输入的字节流能够正确地解析并存储在方法区之内,格式上符合描述一个java类型信息的要求
  - **元数据验证**:字节码语义信息的验证,以保证描述的信息符合java语言规范.验证点有:这个类是否有父类等.
  - **字节码验证**:主要是进行数据流和控制流分析,保证被校验类的方法在运行时不会做出危害虚拟机安全的行为.
  - **符号引用验证**:对符号引用转化为直接引用过程的验证.

- **准备**:为类变量分配内存并设置变量的初始值,这些内存在方法区进行分配.
- **解析**:将虚拟机常量池中的符号引用转化为直接引用的过程.解析主要是针对类或接口、字段、类方法、类接口方法四类.
- **初始化**:执行静态变量的赋值操作以及静态代码块,完成初识化.初始化过程保证了父类中定义的初始化优先于子类的初始化.但接口不需要执行父类的初始化.


##  双亲委派模型
除了顶层的启动类加载器外,其余的类加载器都应当有自己的父类加载器.顺序依次是:

- Bootstrap ClassLoader: 启动类加载器,加载java_home/lib中的类
- Extension ClassLoader: 扩展类加载器,加载java_home/lib/ext目录下的类库
- Application ClassLoader: 应用程序类加载器,加载用户类路径上指定类库.

双亲委派模型的工作原理是:如果一个类加载器受到了类加载请求,它首先不会自己去尝试加载这个类,而把这个请求委派给父类加载器去完成,每一层次的类加载器都是如此,因此所有的加载请求最终都应该传送到顶层的启动类加载器中,只有当父类加载器反馈自己无法完成加载请求时,加载器才尝试自己加载.这种方式保证了Oject类(JDK 核心类)在各个加载器加载环境中都是同一个类.

## 分派：静态分派与动态分派。
多态性特征的一些最基本的体现. **静态类型是编译期可知的,动态类型是在运行时可知**.Human h =new Man(); Human是静态类型,Man时动态类型.

所有依赖于静态类型定位方法执行版本的分派动作称作**静态分派**,最典型的应用是方法重载.静态分派发生在编译阶段。

**动态分派**是根据动态类型来确定执行的版本,所以只有到运行时才能确定具体的执行方法版本.典型的代表时重写.其过程如下:

- 1) 首先找到操作数栈栈顶的第一个元素所执向对象的实际类型,记做C.
- 2) 如果在类型C中找到和常量中的描述符和简单名称都相符的方法,则进行范围权限校验.如果通过则返回该方法的直接引用,否则抛出IllegalAccessError异常.
- 3) 否则按照继承关系从下往上一次对C的各个父类进行第2步的搜索和验证过程.
- 4) 如果始终没有找到就抛出AbstractMethodError异常.
方法的接受者和方法的参数统称方法宗量,根据分配基于多少中宗量可以分为单分派和多分派.java是静态多分派,动态分派属于单分派.

**动态分派的实现:**
动态分派时非常频繁的动作,而且动态分派的方法版本选择过程需要运行时在类的方法元数据中搜索合适的目标方法,因此出于性能的考虑,在方法区中建立一个**虚方法表**,用来保存各个方法的实际入口地址.如果某个方法的子类中没有被重写,那么子类的虚方法表里面的地址入口和父类相同方法的地址入口是一致的.都是指向父类的实现入口,如果子类中重写了这个方法,子类方法表中的地址将会被替换为指向子类实现版本的入口地址.虚方法表在类加载的连接阶段进行初始化.

## Jvm 自动内存管理（什么时候触发 gc ）
http://jeromecen1021.blog.163.com/blog/static/18851527120117274624888/
FULL GC 和 Minor GC 的触发时间
程序员不能具体控制时间，系统在不可预测的时间调用System.gc()函数的时候；当然可以通过调优，用NewRatio控制newObject和oldObject的比例，用MaxTenuringThreshold 控制进入oldObject的次数，使得oldObject 存储空间延迟达到full gc,从而使得计时器引发gc时间延迟OOM的时间延迟，以延长对象生存期。

## GC停顿原因，如何降低停顿
## JVM如何调优、参数怎么调

## jvm的体系结构及各个部分的职责
JVM都有两种机制，一个是装载具有合适名称的类(类或是接口)，包含类的装载 连接 初始化的过程叫做**类装载子系统**；另外的一个负责执行包含在已装载的类或接口中的指令，叫做**运行引擎**。每个JVM又包括方法区、堆、Java栈、程序计数器和本地方法栈这五个部分

- JVM的每个实例都有一个它自己的方法域和一个堆，运行于JVM内的所有的线程都共享这些区域；
- 当虚拟机装载类文件的时候，它解析其中的二进制数据所包含的类信息，并把它们放到方法域中；
- 当程序运行的时候，JVM把程序初始化的所有对象置于堆上；
- 而每个线程创建的时候，都会拥有自己的程序计数器和Java栈，其中程序计数器中的值指向下一条即将被执行的指令，线程的Java栈则存储为该线程调用Java方法的状态；
- 本地方法调用的状态被存储在本地方法栈，该方法栈依赖于具体的实现。

http://blog.csdn.net/dongdong_java/article/details/24797307
http://blog.csdn.net/longyulu/article/details/8350622

## 如果想不被 GC 怎么办
可以先说那些对象可以被GC,然后说java对象会不会回收，决定于是否还被引用，不被引用了就有可能被GC回收，一直被引用着就不会被回收. 

2. jvm性能调优都做了什么
4. 介绍GC 和GC Root不正常引用。
5. 自己从classload 加载方式，加载机制说开去，从程序运行时数据区，讲到内存分配，讲到String常量池，讲到JVM垃圾回收机制，算法，hotspot。反正就是各种扩展
7. 数组多大放在 JVM 老年代（不只是设置 PretenureSizeThreshold ，问通常多大，没做过一问便知）
8. 老年代中数组的访问方式
9. GC 算法，永久代对象如何 GC ， GC 有环怎么处理
9. jvm 如何分配直接内存??
10. new 对象如何不分配在堆而是栈上?
11. 常量池解析

## 运行期优化
## 最佳实践


## Student s= new Student(),在内存中做了那些事情
1. 加载Student.class 文件进内存
2. 在栈内存为s开辟空间
3. 在堆内存为Student对象开辟空间
4. 学生对象的成员变量进行显示初始化
5. 通过构造方法对学生对象变量赋值
6. 学生对象初始完毕，把对象地址赋值给s变量

## 常用参数配置
|参数|   含义|      默认值	|		备注|
|-|-|-|-|
|-Xms | 表示初始堆大小 |默认为物理内存的1/64(<1GB)|	默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.|
|-Xmx |	最大堆大小|	物理内存的1/4(<1GB)|	(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制|
|-Xmn	|年轻代大小(1.4or lator)	 |	|注意：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的整个堆大小=年轻代大小 + 年老代大小 + 持久代大小.增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8|
|-XX:NewSize|	设置年轻代大小(for 1.3/1.4)| | |
|-XX:MaxNewSize|	年轻代最大值(for 1.3/1.4)	|||
|-XX:PermSize|	设置持久代(perm gen)初始值|	物理内存的1/64||
|-XX:MaxPermSize|	设置持久代最大值|	物理内存的1/4	||
|-Xss|	每个线程的堆栈大小	|| JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长）和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话-Xss is translated in a VM flag named ThreadStackSize一般设置这个值就可以了。|
|-XX:NewRatio| 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代) |||
|-XX:NewRatio |=4表示年轻代与年老代所占比值为1:4, 年轻代占整个堆栈的1/5 Xms = Xmx并且设置了Xmn的情况下，该参数不需要进行设置。|||
|-XX:SurvivorRatio|	Eden区与Survivor区的大小比值||	 	设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10|
|-XX:MaxTenuringThreshold|	垃圾最大年龄| 如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代. 对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率该参数只有在串行GC时才有效.|
|-XX:PretenureSizeThreshold|	对象超过多大是直接在旧生代分配|	0	单位字节 |新生代采用Parallel Scavenge GC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象.|



