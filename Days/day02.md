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

### 已读文章

- [拉勾-Java 源码剖析 34 讲-第01讲：String 的特点是什么？它有哪些重要的方法？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1761)
- [在java中String类为什么要设计成final？](https://www.zhihu.com/question/31345592)
- [Why String is Immutable or Final in JavaWhy is String class declared final in Java?](https://javarevisited.blogspot.com/2010/10/why-string-is-immutable-or-final-in-java.html#axzz7P7dHi4io)
- [== 和 equals 的区别是什么？](https://zhuanlan.zhihu.com/p/338350987)
- [深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)
- [Java 性能优化之 String 篇](https://www.yelcat.cc/index.php/archives/863/)
- [Java编译器中对String对象的优化](https://blog.csdn.net/tolcf/article/details/45578771)
- [String、StringBuffer和StringBuilder的区别](https://segmentfault.com/a/1190000022038238)