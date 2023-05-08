# String类源码

## 1.String类介绍

String 类是日常开发中使用最频繁的类之一，同时也是非常重要的一个类String类实现了Serializable, Comparable, CharSequence接口。

String类被**final**所修饰，也就是说String对象是**不可变类**，是**线程安全**的

## 2.成员变量

String类中包含一个不可变的char数组用来存放字符串，一个int型的变量hash用来存放计算后的哈希值。

```
//用于存储字符串
private final char value[];

//缓存String的hash值
private int hash; // Default to 0

/** use serialVersionUID from JDK 1.0.2 for interoperability 使用来自JDK 1.0.2的serialVersionUID实现互操作性*/
private static final long serialVersionUID = -6849794470754667710L;
```

## 3.构造方法

```
//不含参数的构造函数，一般没什么用，因为value是不可变量
public String() {
    this.value = new char[0];
}

//参数为String类型
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}

//参数为char数组，使用java.utils包中的Arrays类复制
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}

//从bytes数组中的offset位置开始，将长度为length的字节，以charsetName格式编码，拷贝到value
public String(byte bytes[], int offset, int length, String charsetName)
        throws UnsupportedEncodingException {
    if (charsetName == null)
        throw new NullPointerException("charsetName");
    checkBounds(bytes, offset, length);
    this.value = StringCoding.decode(charsetName, bytes, offset, length);
}

//调用public String(byte bytes[], int offset, int length, String charsetName)构造函数
public String(byte bytes[], String charsetName)
        throws UnsupportedEncodingException {
    this(bytes, 0, bytes.length, charsetName);
}
```

## 4. String常用方法

- **boolean equals(Object anObject)**

```
public boolean equals(Object anObject) {
    //如果引用的是同一个对象，返回真
    if (this == anObject) {
        return true;
    }
    //如果不是String类型的数据，返回假
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        //如果char数组长度不相等，返回假
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            //从后往前单个字符判断，如果有不相等，返回假
            while (n-- != 0) {
                if (v1[i] != v2[i])
                        return false;
                i++;
            }
            //每个字符都相等，返回真
            return true;
        }
    }
    return false;
}
```

equals方法经常用得到，它用来判断两个对象从实际意义上是否相等，String对象判断规则：

```
1. 内存地址相同，则为真。
2. 如果对象类型不是String类型，则为假。否则继续判断。
3. 如果对象长度不相等，则为假。否则继续判断。
4. 从后往前，判断String类中char数组value的单个字符是否相等，有不相等则为假。如果一直相等直到第一个数，则返回真。
```

**如果对两个超长的字符串进行比较还是非常费时间的**



- **int compareTo(String anotherString)**

```
public int compareTo(String anotherString) {
    //自身对象字符串长度len1
    int len1 = value.length;
    //被比较对象字符串长度len2
    int len2 = anotherString.value.length;
    //取两个字符串长度的最小值lim
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    //从value的第一个字符开始到最小长度lim处为止，如果字符不相等，返回自身（对象不相等处字符-被比较对象不相等字符）
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    //如果前面都相等，则返回（自身长度-被比较对象长度）
    return len1 - len2;
}
```

这个方法写的很巧妙，先从0开始判断字符大小。如果两个对象能比较字符的地方比较完了还相等，就直接返回自身长度减被比较对象长度，如果两个字符串长度相等，则返回的是0，巧妙地判断了三种情况。

- **int hashCode()**

```
public int hashCode() {
    int h = hash;
    //如果hash没有被计算过，并且字符串不为空，则进行hashCode计算
    if (h == 0 && value.length > 0) {
        char val[] = value;

        //计算过程
        //s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        //hash赋值
        hash = h;
    }
    return h;
}
```

**String类重写了hashCode方法**，Object中的hashCode方法是一个Native调用。String类的hash采用**多项式计算**得来，我们完全可以通过不相同的字符串得出同样的hash，所以两个String对象的hashCode相同，并不代表两个String是一样的。

- **boolean startsWith(String prefix,int toffset)**

```
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
    int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    //如果起始地址小于0或者（起始地址+所比较对象长度）大于自身对象长度，返回假
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    //从所比较对象的末尾开始比较
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}

public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}

public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```

起始比较和末尾比较都是比较经常用得到的方法，例如在判断一个字符串是不是http协议的，或者初步判断一个文件是不是mp3文件，都可以采用这个方法进行比较。

- **String concat(String str)**

```
public String concat(String str) {
    int otherLen = str.length();
    //如果被添加的字符串为空，返回对象本身
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```

concat方法也是经常用的方法之一，它先判断被添加字符串是否为空来决定要不要创建新的对象。

- **String replace(char oldChar,char newChar)**

```
public String replace(char oldChar, char newChar) {
    //新旧值先对比
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        //找到旧值最开始出现的位置
        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        //从那个位置开始，直到末尾，用新值代替出现的旧值
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}1234567891011121314151617181920212223242526272829
```

这个方法也有讨巧的地方，例如最开始先找出旧值出现的位置，这样节省了一部分对比的时间。replace(String oldStr,String newStr)方法通过正则表达式来判断。

- **String trim()**

```
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    //找到字符串前段没有空格的位置
    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    //找到字符串末尾没有空格的位置
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    //如果前后都没有出现空格，返回字符串本身
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```

- **String intern()**

```
public native String intern();1
```

intern方法是Native调用，它的作用是在方法区中的常量池里通过equals方法寻找等值的对象，如果没有找到则在常量池中开辟一片空间存放字符串并返回该对应String的引用，否则直接返回常量池中已存在String对象的引用。

将引言中第二段代码

```
//String a = new String("ab1");
//改为
String a = new String("ab1").intern();123
```

则结果为为真，原因在于a所指向的地址来自于常量池，而b所指向的字符串常量默认会调用这个方法，所以a和b都指向了同一个地址空间。

- **int hash32()**

```
private transient int hash32 = 0;
int hash32() {
    int h = hash32;
    if (0 == h) {
       // harmless data race on hash32 here.
       h = sun.misc.Hashing.murmur3_32(HASHING_SEED, value, 0, value.length);

       // ensure result is not zero to avoid recalcing
       h = (0 != h) ? h : 1;

       hash32 = h;
    }

    return h;
}
```

在JDK1.7中，Hash相关集合类在String类作key的情况下，不再使用hashCode方式离散数据，而是采用hash32方法。这个方法默认使用系统当前时间，String类地址，System类地址等作为因子计算得到hash种子，通过hash种子在经过hash得到32位的int型数值。

- **其他方法**

```
public int length() {
    return value.length;
}
public String toString() {
    return this;
}
public boolean isEmpty() {
    return value.length == 0;
}
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}
```

## 5.关于不可变类

### **(1)  什么是不可变类**

所谓不可变类，就是创建该类的实例后，该实例的属性是不可改变的，java提供的包装类和java.lang.String类都是不可变类。当创建它们的实例后，其实例的属性是不可改变的。

需要注意的是，对于如下代码

```
String s="abc";
s="def";12
```

你可能会感到疑惑，不是说String是不可变类吗，这怎么可以改变呢，平常我也是这样用的啊。请注意，s是字符串对象的”abc”引用，即引用是可以变化的，跟对象实例的属性变化没有什么关系，这点请注意区分。

### **(2) String类被设计成不可变的原因**

- **字符串常量池的需要**

字符串常量池(String pool, String intern pool, String保留池) 是Java方法区中一个特殊的存储区域, 当创建一个String对象时,假如此字符串值已经存在于常量池中,则不会创建一个新的对象,而是引用已经存在的对象。
如下面的代码所示,将会在堆内存中只创建一个实际String对象.
代码如下:

```
String s1 = "abcd"; 
String s2 = "abcd"; 12
```

假若字符串对象允许改变,那么将会导致各种逻辑错误,比如改变一个对象会影响到另一个独立对象. 严格来说，这种常量池的思想,是一种优化手段.

```
String s1= "ab" + "cd"; 
String s2= "abc" + "d"; 12
```

- 也许这个问题违反新手的直觉, 但是考虑到现代编译器会进行常规的优化, 所以他们都会指向常量池中的同一个对象. 或者,你可以用 jd-gui 之类的工具查看一下编译后的class文件.
- **允许String对象缓存HashCode**

Java中String对象的哈希码被频繁地使用, 比如在hashMap 等容器中。

字符串不变性保证了hash码的唯一性,因此可以放心地进行缓存.这也是一种性能优化手段,意味着不必每次都去计算新的哈希码.

- **安全性**

String被许多的Java类(库)用来当做参数,例如 网络连接地址URL,文件路径path,还有反射机制所需要的String参数等, 假若String不是固定不变的,将会引起各种安全隐患。
假如有如下的代码:

```
boolean connect(string s){
    if (!isSecure(s)) {
throw new SecurityException();
}
    // 如果在其他地方可以修改String,那么此处就会引起各种预料不到的问题/错误
    causeProblem(s);
}12345678
```

- **线程安全**
  因为字符串是不可变的，所以是多线程安全的，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。

总体来说, String不可变的原因包括 设计考虑,效率优化问题,以及安全性这三大方面.

### (3) 如何实现一个不可变类

既然不可变类有这么多优势，那么我们借鉴String类的设计，自己实现一个不可变类。
不可变类的设计通常要遵循以下几个原则：

1. 将类声明为final，所以它不能被继承。
2. 将所有的成员声明为私有的，这样就不允许直接访问这些成员。
3. 对变量不要提供setter方法。
4. 将所有可变的成员声明为final，这样只能对它们赋值一次。
5. 通过构造器初始化所有成员，进行深拷贝(deep copy)。
6. 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝。