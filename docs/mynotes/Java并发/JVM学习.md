JVM理解？java8虚拟机有什么更新？

什么是OOM？什么是StackOverflowError?有哪些方法分析？

JVM的常用参数调优有哪些？

类加载器的认识？



**本地方法栈**

- 多线程 Thread  start方法调用底层 native方法 start0 只有声明，没有实现，在执行引擎执行时，调用本地方法库。不能连续两次start（非法线程状态异常）

程序计数器 

- 记录方法之间的调用和执行情况，类似排版值日表
- 用来存储下一行指令的二地址，也即将要执行的指令代码
- 它是当前线程所执行的字节码的行号指示器

方法区  线程私有，存在垃圾回收

- 存储每一个类的结构信息（模板） 
- 实现：永久代  元空间



**栈管运行 堆管存储**  



**Java虚拟机栈**  

-  不存在垃圾回收，线程私有，线程结束，栈释放

- 8中基本数据类型+对象的引用变量+实例方法 都在栈内存分配

- 栈帧存储的数据（栈帧  --java方法 ）

  - 本地变量 ：输入参数 输出参数 以及方法内的变量
  - 栈操作 ：记录出栈 入栈的 操作；
  - 栈帧数据：包括类文件、方法等

  

- 递归调用可导致StackOverflowError  错误

**Java堆**（heap）

堆内存大小可以调节

Java7之前，**堆逻辑上分为： 新生+老年+永久**

Java8 永久区换成了元空间

**物理上：新生+老年**

- 新生代 NEW
  - Eden区
  - Survivor 0 Space
  - Survivor 1 Space
- 老年代 OLD
- 元空间\永久代 （堆的逻辑部分）物理上---方法区（非堆） 几乎没有垃圾回收，加载rt.jar包
- 元空间不在JVM，在物理内存  字符串池、类的静态变量放在堆中

YGC  新生区GC (轻量级GC，Eden基本全部清除，幸存 ->  交换至Survivor 0 Space->Survivor 1 Space->活过15次GC->老年区->老年区满了->Full GC ->多次FullGC 老年区依旧空间不足->OOM堆内存溢出)

S0 = from

S1 = to

Eden:from:to = 8:1:1  (1/3)

Old (2/3)

MinorGC  复制->清空->互换

第一次GC：eden

**复制算法**：

- 第一次GC：eden->from
- Eden 、 from 复制到to，年龄+1 ,年龄到了15次->老年代区



from区和to区，他们的位置名称不固定，每次GC后会交换，**GC之后有交换，谁空谁是to**



**对象的生命周期**





**堆、栈、方法区 关系**：堆中存放访问类原数据的地址，reference存储的就直接是对象的地址



类加载器

BootstrapClassLoader

ExtendsClassLoader

ApplicationClassLoader

双亲委派模型  沙箱安全机制



**堆参数调优**：

-Xms -Xmx  堆初始化 堆最大化  -Xms默认物理内存1/64  -Xmx默认物理内存1/4

**生产环境配成一样大小，避免GC和应用程序争抢内存，理论值的峰值、峰谷忽高忽低。**

查看CPU核数 System.out.println(Runtime.getRuntime().availableProcessors());

-Xmn NEW区 一般不调

如何调：

VM options 

-Xms1024m -Xmn1024m -XX:+PrintGCDetails 

-XX:+PrintGCDetails 输出详细GC收集日志信息

-XX:MaxTenuringThreshold 设置对象在新生代中存活的次数

```
String str = "sakfjkajlfka";

while(true){
	sre+= new Random().nextInt(999999999)+ new Random().nextInt(888888888);
}

byte[] bytes = new byte[40*1024*1024*1024];

```

**GC是什么**（分代收集算法）

- 次数频繁收集Young区
- 次数较少收集Old区
- 基本不动元空间

非三个区同时回收，主要发生在Young区

FullGC比MinorGC慢10以上  （空间扫描范围大）

GC四大算法：

- 引用计数法
- 复制算法    费空间  新生代
- 标记清除算法 Mark-Sweep 老年代 标记出要回收的垃圾，然后统一回收  两次扫描，耗时严重 节约空间，但出现内存碎片 
- 标记整理算法 Mark-Compact  老年代   理论上最好，但时间花销大



**JMM  Java内存模型**

- 可见性
- 原子性
- 有序性

volatile是java虚拟机提供的轻量级的同步机制

- 保证可见性
- 不保证原子性
- 禁止指令重排序

JMM控制线程间的通信