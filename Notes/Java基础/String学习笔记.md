# 1.简介

Java中的String是字符串类

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
```

String类实现了3个接口：

- Serializable ，这个序列化接口没有任何方法和域，仅用于标识序列化的语意。

- Comparable，这个接口只有一个方法：

```java
	public int compareTo(T o);
```

- CharSequence，该接口是一个只读的字符序列。包括length( )，charAt(int index)，subSequence(int start, int end) 这几个API接口。

  值得一提的是，StringBuffer和StringBuild也是实现了该接口。

# 2.特性

- 不变性：是一个`immutable`模式的对象，不变模式的主要作用是当一个对象需要被多线程共享并频繁访问时，可以保证数据的一致性

- 常量池优化：String对象创建后，会在字符串常量池进行缓存，下次创建同样的对象时，会直接返回缓存的引用

**思考：为什么要设计成不可变类呢？**

主要是考虑效率和安全性

- 效率：缓存的是hashcode，String不可变，所以hashcode不变。这样缓存才有意义，不必重新计算。

  这样，创建同样对象时，才方便根据hashcode快速从常量池中获取缓存的字符串引用。

- 安全性：String常被作为网络连接，文件操作等参数类型，倘若可改变，会出现意想不到的结果。

# 3.创建方式

这里先简单介绍下JVM的内存分布，如下图所示：

<img src="111" alt="img" style="zoom: 67%;" />

## 3.1直接赋值

在方法区中字符串常量池中创建对象

```java
String str = "allen716";
```

当用此方法创建字符串对象时，会先去方法区的常量池中查找是否有“allen716”。

当存在时，直接返回常量池里的引用；当不存在时，会在字符创常量池中创建一个对象并返回引用。

**拓展：**

疑问：创建对象本来应该都在堆区。

解答：我们可以把字符串常量池当做一个 HashSet，它存储的是对象的引用，并不存储对象实例。对象还是在堆			上的。

## 3.2构造器创建对象

在堆内存创建对象

```java
String str = new String("allen716");

```

# 4.字符串的拼接

String的不可变性，导致String字符串拼接时效率较低。

为此，JDK1.0还引入了StringBuffer，并且JDK1.5中还引入了StringBuilder。

三者之间比较如下：

- String：不可变，线程安全。适用场景：少量连接操作，如：

  ```java
  String a = "aaa";
  String b = "bbb";
  String c = "ccc";
  String d = a + b + c; //少量字符串拼接
  ```

  **特别说明：**

  以“String d = a + b + c”为例，这里进行拼接时，底层原理如下：

  （1）先以a的值为准，生成一个StringBuilder对象；

  （2）调用StringBuidler.append(b).append(c)，进行连接操作；

  （3）调用StringBuilder.toString( )，将StringBuilder转化成String，赋值给d

- StringBuffer：可变，线程安全。适用场景：多线程下有大量连接操作时。

- StringBuilder：可变，线程不安全。适用场景：单线程下有大量连接操作时。

**运行速度：StringBuilder > StringBuffer > String**

# 5.面试题汇总

**1.为什么String的是不可变的？**

​	因为存储数据的char数组是使用final进行修饰的，所以不可变。

​	源码：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

**2.刚才说到String是不可变，但是下面的代码运行完，却发生变化了，这是为啥呢？**

```java
public class Demo {
    public static void main(String[] args) {
        String str = "allen";
        str = str + "716";
        System.out.println(str);
    }
}
```

在使用"+"进行拼接的时候，实际上jvm是初始化了一个StringBuilder进行拼接的。

相当于编译后的代码如下：

public class Demo {

```java
public static void main(String[] args) {
    String str = "allen";
    StringBuilder builder =new StringBuilder();
    builder.append(str);
    builder.append("716");
    str = builder.toString();
    System.out.println(str);
}
```

}

最后一步，是调用StringBuilder的toString( )方法，生成一个新的String对象。

相当于把旧str的引用指向的新的String对象。



**3.String类可以被继承吗？**

不可以，因为String类使用final关键字进行修饰，所以不能被继承。

并且StringBuilder，StringBuffer也是如此都被final关键字修饰。



**4.为什么String Buffer是线程安全的？**

在StringBuffer类中，常用的方法都使用了synchronized 进行同步，所以是线程安全的。然而StringBuilder并没有。也由于这个原因，所以运行速度：StringBuilder > StringBuffer的原因了。



**5.下面的程序中，输出的是什么？**

```java
public class Demo {
    public static void main(String[] args) {
        String str = null;
        str = str + "";
        System.out.println(str);
    }
}
```

答案是 null。从之前我们了解到使用 ”+“ 进行拼接实际上是会转换为StringBuilder，使用append方法进行拼接。所以我们看看append方法实现逻辑就明白了。

（1）StringBuilder实际调用的是AbstractStringBuilder的append（）方法：

```java
@Override
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```

（2）AbstractStringBuilder的的append（）方法：

```java
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```

（3）当（2）中入参str为null时，调用appendNull方法：

```java
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

这里返回的是"null"