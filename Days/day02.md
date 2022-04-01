## Introduction - Day 2

### String.Class

1. 构造方法
2. equals()
3. hashCode()
4. 其它重要方法

---

> 1.8 String内部实际存储结构char数组
~~~java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
    //......其他内容
}
~~~

#### 1. 构造方法
String源码中包含下面几个重要的构造方法
~~~java
 /**
 * Initializes a newly created {@code String} object so that it represents
 * the same sequence of characters as the argument; in other words, the
 * newly created string is a copy of the argument string. Unless an
 * explicit copy of {@code original} is needed, use of this constructor is
 * unnecessary since Strings are immutable.
 *
 * @param  original
 *         A {@code String}
 */
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}

/**
 * Allocates a new {@code String} so that it represents the sequence of
 * characters currently contained in the character array argument. The
 * contents of the character array are copied; subsequent modification of
 * the character array does not affect the newly created string.
 *
 * @param  value
 *         The initial value of the string
 */
public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
}

/**
 * Allocates a new string that contains the sequence of characters
 * currently contained in the string buffer argument. The contents of the
 * string buffer are copied; subsequent modification of the string buffer
 * does not affect the newly created string.
 *
 * @param  buffer
 *         A {@code StringBuffer}
 */
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}

/**
 * Allocates a new string that contains the sequence of characters
 * currently contained in the string builder argument. The contents of the
 * string builder are copied; subsequent modification of the string builder
 * does not affect the newly created string.
 *
 * <p> This constructor is provided to ease migration to {@code
 * StringBuilder}. Obtaining a string from a string builder via the {@code
 * toString} method is likely to run faster and is generally preferred.
 *
 * @param   builder
 *          A {@code StringBuilder}
 *
 * @since  1.5
 */
public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
}

以及
* @see  #String(byte[], int, int, java.lang.String)
* @see  #String(byte[], int, int, java.nio.charset.Charset)
* @see  #String(byte[], int, int)
* @see  #String(byte[], java.lang.String)
* @see  #String(byte[], java.nio.charset.Charset)
* @see  #String(byte[])
~~~

#### 2. equals() 比较两个字符串是否相等
~~~java
 /**
 * Compares this string to the specified object.  The result is {@code
 * true} if and only if the argument is not {@code null} and is a {@code
 * String} object that represents the same sequence of characters as this
 * object.
 *
 * @param  anObject
 *         The object to compare this {@code String} against
 *
 * @return  {@code true} if the given object represents a {@code String}
 *          equivalent to this string, {@code false} otherwise
 *
 * @see  #compareTo(String)
 * @see  #equalsIgnoreCase(String)
 */
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
~~~
String 类型重写了 Object 中的 equals() 方法，equals() 方法需要传递一个 Object 类型的参数值， 
在比较时会先通过 instanceof 判断是否为 String 类型，如果不是则会直接返回 false，instanceof 的使用如下：

当判断参数为 String 类型之后，会循环对比两个字符串中的每一个字符，当所有字符都相等时返回 true，否则则返回 false。

还有一个和 equals() 比较类似的方法 equalsIgnoreCase()，它是用于忽略字符串的大小写之后进行字符串对比。

#### 3. hashCode()
~~~
/**
 * Returns a hash code for this string. The hash code for a
 * {@code String} object is computed as
 * <blockquote><pre>
 * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
 * </pre></blockquote>
 * using {@code int} arithmetic, where {@code s[i]} is the
 * <i>i</i>th character of the string, {@code n} is the length of
 * the string, and {@code ^} indicates exponentiation.
 * (The hash value of the empty string is zero.)
 *
 * @return  a hash code value for this object.
 */
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
~~~

#### 4. 其它重要方法
~~~
compareTo()： 比较两个字符串
indexOf()：查询字符串首次出现的下标位置
lastIndexOf()：查询字符串最后出现的下标位置
contains()：查询字符串中是否包含另一个字符串
toLowerCase()：把字符串全部转换成小写
toUpperCase()：把字符串全部转换成大写
length()：查询字符串的长度
trim()：去掉字符串首尾空格
replace()：替换字符串中的某些字符
split()：把字符串分割并返回字符串数组
join()：把字符串数组转为字符串
~~~

**以及对应的问题**
- 为什么 String 类型要用 final 修饰？
- == 和 equals 的区别是什么？
- String 和 StringBuilder、StringBuffer 有什么区别？
- String 的 intern() 方法有什么含义？
- String 类型在 JVM（Java 虚拟机）中是如何存储的？编译器对 String 做了哪些优化？

---
由问题引起的新概念
--常量池
> 常量池jdk1.7以前在永生代, jdk1.7后挪除永生代，换成元空间。常量池便放在Java堆里面

常量池是有一个池子，是由大小的，跟进intern（native方法）源码可知
> 如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回

大体实现结构就是: 
JAVA 使用 jni 调用c++实现的StringTable的intern方法, StringTable的intern方法跟Java中的HashMap的实现是差不多的, 只是不能自动扩容。默认大小是1009

在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。
在jdk7中，StringTable的长度可以通过一个参数指定： -XX:StringTableSize=99991

使用intern可以减少对象的生成，但会增加时间 （这是因为程序中每次都是用了 new String 后，然后又进行 intern 操作的耗时时间）

inter的不当使用 -- （fastjson1.1.24版本以前）
#### String 在 JVM 的存储结构
一般而言，Java 对象在虚拟机的结构如下：
  - 对象头（object header）：8 个字节
  - Java 原始类型数据：如 int, float, char 等类型的数据，各类型数据占内存如表 1。
  - 引用（reference）：4 个字节
  - 填充符（padding）
    如果对于 String（JDK 6）的成员变量声明如下：
~~~java
private final char value[];
private final int offset;
private final int count;
private int hash;
~~~
那么因该如何计算该 String 所占的空间？
首先计算一个空的 char 数组所占空间，在 Java 里数组也是对象，因而数组也有对象头，故一个数组所占的空间为对象头所占的空间加上数组长度，即 8 + 4 = 12 字节 , 经过填充后为 16 字节。
那么一个空 String 所占空间为：
对象头（8 字节）+ char 数组（16 字节）+ 3 个 int（3 × 4 = 12 字节）+1 个 char 数组的引用 (4 字节 ) = 40 字节。
因此一个实际的 String 所占空间的计算公式如下：
> 8*( ( 8+2*n+4+12)+7 ) / 8 = 8*(int) ( ( ( (n) *2 )+43) /8 )

其中，n 为字符串长度。

String.split() 或 String.substring()
在 JDK 1.6 中 String.substring(int, int) 的源码为：
~~~java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
    throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > count) {
    throw new StringIndexOutOfBoundsException(endIndex);
    }
    if (beginIndex > endIndex) {
    throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
    }
    return ((beginIndex == 0) && (endIndex == count)) ? this :
    new String(offset + beginIndex, endIndex - beginIndex, value);
}
~~~
调用的 String 构造函数源码为：
~~~java
String(int offset, int count, char value[]) {
    this.value = value;
    this.offset = offset;
    this.count = count;
}
~~~
String.substring() 所返回的 String 仍然会保存原始 String

因此有关通过 String.split() 或 String.substring() 截取 String 的操作的结论如下：
> 对于从大文本中截取少量字符串的应用， String.substring() 将会导致内存的过度浪费。
> 对于从一般文本中截取一定数量的字符串，截取的字符串长度总和与原始文本长度相差不大，现有的 String.substring() 设计恰好可以共享原始文本从而达到节省内存的目的

办法有很多种，在此我们采取比较直观的一种，即再次调用 newString 构造一个的仅包含截取出的字符串的 String，我们可调用 String. toCharArray () 方法：
> String newString = new String(smallString.toCharArray());

举一个极端例子，假设要从一个字符串中获取所有连续的非空子串，字符串长度为 n，如果用 JDK 本身提供的 String.substring() 方 法，则总共的连续非空子串个数为：

>n+(n-1)+(n-2)+... +1 = n*(n+1)/2 =O(n2)


由于每个子串所占的空间为常数，故空间复杂度也为 O(n2)。

如果用本文建议的方法，即构造一个内容相同的新的字符串，则所需空间正比于子串的长度，则所需空间复杂度为：
> 1*n+2*(n-1)+3*(n-2)+... +n*1 = (n3+3*n2+2*n)/6 = O(n3)

所以，从以上定量的分析看来，当需要截取的字符串长度总和大于等于原始文本长度，本文所建议的方法带来的空间复杂度反而高了，而现有的 String.substring() 设计恰好可以共享原始文本从而达到节省内存的目的。反之，当所需要截取的字符串长度总和远小于原始文本长度时，用本文所推荐的方法将在很大程度上节省内存，在大文本数据处理中其优势显而易见


---
#### JDk9对String的优化 
> 到了JDK9，String中字符串的存储不再用char数组了，改用byte数组。而且新增了一个coder的成员变量 和 COMPACT_STRINGS

~~~java
public final class String
implements java.io.Serializable, Comparable<String>, CharSequence {

    @Stable
    private final byte[] value;

    private final byte coder;
    
    @Native static final byte LATIN1 = 0;
    @Native static final byte UTF16  = 1;
    
    static final boolean COMPACT_STRINGS;
}
~~~

在程序中，绝大多数字符串只包含英文字母数字等字符，使用Latin-1编码，一个字符占用一个byte。如果使用char，一个char要占用两个byte，会占用双倍的内存空间。

但是，如果字符串中使用了中文等超出Latin-1表示范围的字符，使用Latin-1就没办法表示了。这时JDK会使用UTF-16编码，那么占用的空间和旧版（使用char[]）是一样的。

coder变量代表编码的格式，目前String支持两种编码格式Latin-1和UTF-16。Latin-1需要用一个字节来存储，而UTF-16需要使用2个字节或者4个字节来存储。

据说这一改进方案是JDK的开发人员用大数据和人工能智能，调研了成千上万的应用程序的heapdump信息后，得出：大部分的String都是以Latin-1字符编码来表示的，只需要一个字节存储就够了，两个字节完全是浪费。

COMPACT_STRINGS属性则是用来控制是否开启String的compact功能。默认情况下是开启的。可以使用-XX:-CompactStrings参数来对此功能进行关闭。


**改进的好处**
改进的好处是非常明显的，首先如果项目中使用Latin-1字符集居多，内存的占用大幅度减少，同样的硬件配置可以支撑更多的业务。

当内存减少之后，进一步导致减少GC次数，进而减少Stop-The-World的频次，同样会提升系统的性能。
### Thanks

- [拉勾-Java 源码剖析 34 讲-第01讲：String 的特点是什么？它有哪些重要的方法？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1761)
- [在java中String类为什么要设计成final？](https://www.zhihu.com/question/31345592)
- [Why String is Immutable or Final in JavaWhy is String class declared final in Java?](https://javarevisited.blogspot.com/2010/10/why-string-is-immutable-or-final-in-java.html#axzz7P7dHi4io)
- [== 和 equals 的区别是什么？](https://zhuanlan.zhihu.com/p/338350987)
- [深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)
- [Java 性能优化之 String 篇](https://www.yelcat.cc/index.php/archives/863/)
- [Java编译器中对String对象的优化](https://blog.csdn.net/tolcf/article/details/45578771)
- [String、StringBuffer和StringBuilder的区别](https://segmentfault.com/a/1190000022038238)
- [JDK9对String字符串的新一轮优化，不可不知](https://mp.weixin.qq.com/s/p1Q5AZWETUtajqtY2GUMtA)