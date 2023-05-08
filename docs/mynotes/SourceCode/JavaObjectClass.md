# Java Object类

### 1.概述

Object类是类层次结构的根，Java中所有的类从根本上都继承自这个类。Object类是Java中其他所有类的祖先，没有Object类Java面向对象无从谈起。作为其他所有类的基类，Object具有哪些属性和行为，是Java语言设计背后的思维体现。

Object类是Java中**唯一**没有父类的类。其他所有的类，包括标准容器类，比如数组，都继承了Object类中的方法。

Object类位于java.lang包中，java.lang包包含着Java最基础和核心的类，在编译时会自动导入。Object类**没有定义属性**，一共有**13个方法**。

### 2.构造器

```
	public Object()
```

大部分情况下，Java中通过形如 new A(args..)形式创建一个属于该类型的对象。其中A即是类名，A(args..)即此类定义中相对应的构造函数。通过此种形式创建的对象都是通过类中的构造函数完成。为体现此特性，Java中规定：在类定义过程中，对于未定义构造函数的类，默认会有一个无参数的构造函数，作为所有类的基类，Object类自然要反映出此特性，在源码中，未给出Object类构造函数定义，但实际上，此构造函数是存在的。

当然，并不是所有的类都是通过此种方式去构建，也自然的，并不是所有的类构造函数都是public。

### 3.方法

**protected Object clone() throws CloneNotSupportedException**
这个方法比较特殊：
首先，使用这个方法的类必须实现java.lang.Cloneable接口，否则会抛出CloneNotSupportedException异常。
Cloneable接口中不包含任何方法，所以实现它时只要在类声明中加上implements语句即可。
第二个比较特殊的地方在于这个方法是protected修饰的，覆写clone()方法的时候需要写成public，才能让类外部的代码调用。

**boolean equals(Object obj)**
Object类中的equals()方法如下：

```
public boolean equals(Object obj)
{
    return (this == obj);
}
```

对于Object类的equals()方法来说，它判断调用equals()方法的引用于传进来的引用是否一致，即这两个引用是否指向的是同一个对象。即Object类中的equals()方法等价于== 。只有当继承Object的类覆写（override）了equals()方法之后，继承类实现了用equals()方法比较两个对象是否相等，才可以说equals()方法与==的不同。
equals()方法需要具有如下特点：
自反性（reflexive）：任何非空引用x，x.equals(x)返回为true。
对称性（symmetric）：任何非空引用x和y，x.equals(y)返回true当且仅当y.equals(x)返回true。
传递性（transitive）：任何非空引用x和y，如果x.equals(y)返回true，并且y.equals(z)返回true，那么x.equals(z)返回true。
一致性（consistent）：两个非空引用x和y，x.equals(y)的多次调用应该保持一致的结果，（前提条件是在多次比较之间没有修改x和y用于比较的相关信息）。
约定：对于任何非空引用x，x.equals(null)应该返回为false。

并且覆写equals()方法时，应该同时覆写hashCode()方法，反之亦然。



**int hashCode()**
当你覆写（override）了equals()方法之后，必须也覆写hashCode()方法，反之亦然。这个方法返回一个整型值（hash code value），如果两个对象被equals()方法判断为相等，那么它们就应该拥有同样的hash code。
Object类的hashCode()方法为不同的对象返回不同的值，Object类的hashCode值表示的是对象的地址。
hashCode的一般性契约（需要满足的条件）如下：
1.在Java应用的一次执行过程中，如果对象用于equals比较的信息没有被修改，那么同一个对象多次调用hashCode()方法应该返回同一个整型值。应用的多次执行中，这个值不需要保持一致，即每次执行都是保持着各自不同的值。
2.如果equals()判断两个对象相等，那么它们的hashCode()方法应该返回同样的值。
3.并没有强制要求如果equals()判断两个对象不相等，那么它们的hashCode()方法就应该返回不同的值,即两个对象用equals()方法比较返回false，它们的hashCode可以相同也可以不同。但是，应该意识到，为两个不相等的对象产生两个不同的hashCode可以改善哈希表的性能。



**String toString()**

```
public String toString()

{
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

当打印引用，如调用System.out.println()时，会自动调用对象的toString()方法，打印出引用所指的对象的toString()方法的返回值，因为每个类都直接或间接地继承自Object，因此每个类都有toString()方法。

### 4.源码解析

```
package java.lang;     
public class Object {     
      
   /* 一个本地方法，具体是用C（C++）在DLL中实现的，然后通过JNI调用。*/      
    private static native void registerNatives();     
  /* 对象初始化时自动调用此方法*/    
    static {     
        registerNatives();     
    }     
   /* 返回此 Object 的运行时类。*/    
    public final native Class<?> getClass();     
    
/*   
hashCode 的常规协定是：   
1.在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。    
2.如果根据 equals(Object) 方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果。    
3.如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么对这两个对象中的任一对象上调用 hashCode 方法不要求一定生成不同的整数结果。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。   
*/    
    
    public native int hashCode();     
    
    
    public boolean equals(Object obj) {     
    return (this == obj);     
    }     
    
    /*本地CLONE方法，用于对象的复制。*/    
    protected native Object clone() throws CloneNotSupportedException;     
    
    /*返回该对象的字符串表示。非常重要的方法*/    
    public String toString() {     
    return getClass().getName() + "@" + Integer.toHexString(hashCode());     
    }     
    
   /*唤醒在此对象监视器上等待的单个线程。*/    
    public final native void notify();     
    
   /*唤醒在此对象监视器上等待的所有线程。*/    
    public final native void notifyAll();     
    
    
/*在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待。换句话说，此方法的行为就好像它仅执行 wait(0) 调用一样。    
当前线程必须拥有此对象监视器。该线程发布对此监视器的所有权并等待，直到其他线程通过调用 notify 方法，或 notifyAll 方法通知在此对象的监视器上等待的线程醒来。然后该线程将等到重新获得对监视器的所有权后才能继续执行。*/    
    public final void wait() throws InterruptedException {     
    wait(0);     
    }     
    
    
    
   /*在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量前，导致当前线程等待。*/    
    public final native void wait(long timeout) throws InterruptedException;     
    
    /* 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。*/    
    public final void wait(long timeout, int nanos) throws InterruptedException {     
        if (timeout < 0) {     
            throw new IllegalArgumentException("timeout value is negative");     
        }     
    
        if (nanos < 0 || nanos > 999999) {     
            throw new IllegalArgumentException(     
                "nanosecond timeout value out of range");     
        }     
    
    if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {     
        timeout++;     
    }     
    
    wait(timeout);     
    }     
    
    /*当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。*/    
    protected void finalize() throws Throwable { }     
}
```

