# **String、StringBuffer和StringBuilder的区别**

## **类继承关系图**

![类继承关系图](http://img.blog.csdn.net/20170502095909981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ2hhcm1pbmdXb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## **String**

```Java
String s1 = "hello world";
String s2 = "hello world";
System.out.println(s1 == s2);
```

 >输出结果是 **true**

```Java
String s1 = new String("hello world");
String s2 = new String("hello world");
String s3 = "hello world";
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
System.out.println(s1.equals(s3));
```

>输出结果是 **false、true、true**

在 Java 编译好的 **class** 文件中，有个区域被称为***常量池（Constant Pool）***，存储着各种常量，其中存储字符串常量的区域被习惯地称为***字符串池（String Pool）***。在第一个例子中，有两个“hello world”常量，但是在常量池中，只会创建一个常量，也就是说，在编译好的 **class** 文件中，只能找到一个“hello world”常量。因此 **s1** 和 **s2** 指向的在常量池中同一个字符串对象，“==”比较是否是同一个对象，所以结果返回 **true。**

## **扩展知识**

1.
```Java
private final char value[];
```
字符串是用一个不可变的字符数组来存储字符串内容，当 **String** 对象创建之后，就不能修改对象中存储的内容，所以说，**String** 类型是不可变的***（immutable）***
例如

```Java
String s1 = new String("hello");
s1 = s1 + " world";
```
因为 **String** 是不可变的，这段代码会根据s1的内容和字符串常量“ world”在 ***heap*** 区创建一个新的字符串

 2.
```Java
String s1 = new String("hello world");
```

该语句创建两个 **String** 对象，一个是在编译时，创建于***字符串常量区***，一个是在运行时，创建于 ***heap*** 区。

3.**Java** 对 **String** 进行了“+”操作符重载，利用“+”可以将字符串连接。但是需要注意的是

```Java
String s1 = "hello" + " world";
```
该代码与第一点的例子不一样，编译时编译器会直接提取两个常量的内容，只生成“hello world”一个字符串，同理，无论多少个常量相加，都是生成一个对象。

4.运行时调用 **String** 的 **_intern()_** 方法可以向 **_String Pool_** 中动态添加对象。

5.**String** 创建的几种方法

 1. 使用“”。
 2. 使用 new String();
 3. 使用 new String("hello world");
 4. 使用重载操作符“+”

## **StringBuffer&StringBuilder**

**StringBuilder** 一个可变的字符序列是5.0新增的。此类提供一个与 **StringBuffer** 兼容的 API，但不保证同步。该类被设计用作 **StringBuffer** 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。如果可能，建议优先采用该类，因为在大多数实现中，它比 **StringBuffer** 要快。
**Stringbuffer** 和 **StringBuilder** 除了线程安全的区别外，其他基本一致，都继承于 **AbstractStringBuffer** 。
来看一下 **AbstractStringBuffer** 的源码

```Java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

    /**
     * This no-arg constructor is necessary for serialization of subclasses.
     */
    AbstractStringBuilder() {
    }

    /**
     * Creates an AbstractStringBuilder of the specified capacity.
     */
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }

    /**
     * Returns the length (character count).
     *
     * @return  the length of the sequence of characters currently
     *          represented by this object
     */
    @Override
    public int length() {
        return count;
    }

    /**
     * Returns the current capacity. The capacity is the amount of storage
     * available for newly inserted characters, beyond which an allocation
     * will occur.
     *
     * @return  the current capacity
     */
    public int capacity() {
        return value.length;
    }
```

 **AbstractStringBuffer** 使用的是可变的 **char** 数组，在创建 **StringBuffer** 和 **StringBuilder** 时有几种方式，其中无参的构造方法调用的是父类的构造方法，创建大小为16的 **char** 数组

```Java
public StringBuffer() {
    super(16);
}

public StringBuilder() {
    super(16);
}
```

其他的构造方法，**StringBuilder** 与之类似

```Java
public StringBuffer(int capacity) {
    super(capacity);
}

public StringBuffer(String str) {
    super(str.length() + 16);
    append(str);
}

public StringBuffer(CharSequence seq) {
    this(seq.length() + 16);
    append(seq);
}
```
其他常见方法：**append**、**insert**

</br>

# **三者区别**

### **可变性**
String 不可变，StringBuffer 、StringBuilder 可变

### **速度**
StringBuilder > StringBuffer > String


### **线程安全**
String、StringBuilder 不安全，StringBuffer 安全

# **总结**

 1. **在编译阶段就能够确定的字符串常量，完全没有必要创建 _String_ 或 _StringBuffer_ 对象。直接使用字符串常量的"+"连接操作效率最高。**

 2. **StringBuffer 对象的 _append_ 效率要高于 _String_ 对象的"+"连接操作。**

 3. **不停的创建对象是程序低效的一个重要原因。那么相同的字符串值能否在堆中只创建一个 _String_ 对象那。显然拘留字符串能够做到这一点，除了程序中的字符串常量会被 _JVM_ 自动创建拘留字符串之外，调用 _String_ 的 _intern_() 方法也能做到这一点。当调用 _intern_() 时，如果常量池中已经有了当前 _String_ 的值，那么返回这个常量指向拘留对象的地址。如果没有，则将 _String_ 值加入常量池中，并创建一个新的拘留字符串对象。**
