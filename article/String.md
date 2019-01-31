### 概览 ###

String 算是 Javaer 用的最多的类了。String 是 final 类型等的，也就是说，String 是无法被继承的，也无法重写它的函数。

```java
String

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
}
```

可以看到 String 被声明成 final 类型的，String 内部有一个 value 数组，这个 value 数组就是用来保存字符串的。也就是说，String 底层是用数组实现的。

value 数组也是 final 类型的，当用 final 修饰引用时，引用的指向无法被修改：

```java
final byte[] value = new byte[1];
value = new byte[2];
```

比如上面的代码，是修改 final 类型的 value 数组的引用，会报错。

但我们也知道，用 final 修饰引用时，引用的指向无法被修改，但是指向的对象是可以被修改的。

```java
final byte[] value = new byte[1];
value[0] = 1;
```

上面的代码，是修改 final 类型的 value 数组，可以编译通过。

但是 String 类中又没有提供修改 value 数组的方法，所以 value 数组相当于无法被修改。因此可以说，String 是不可变的。

我们这里说的 String 不可变，指的是 String 对象里面的 value 数组一旦确定下来，就不会再改变了。

有的童鞋可能会疑惑了，我平常用 String，直接给 String 对象赋个值，它不就变了。

```java
String s = "a";
s = "b";
System.out.println(s);

输出：b
```



从输出结果可以看出，s 的确是改变了，那为什么我们说 String 是不可变的。因为 s 只是 String 对象的引用，引用只是一个 4 字节的数据，里面存放了它所指向的对象（"a"）的地址。当我们执行 `s = "b"`，实际上是把 "b" 对象的地址赋值给了 s，现在 s 指向的是 "b" 对象，原来的对象 "a" 还在内存中。



### 不可变的好处 ###

那么，再思考一下，String 为什么要设计成 final 呢，不可变有什么好处？

1. 可以缓存 hash 值。

我们先看看 String 的 hash 值是怎么计算的：

```java
String # hashCode

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
```

可以看到，String 是用 value 数组通过一定的规则来生成一个hash 值。那么，value 数组一旦变化，hash 值肯定也会变。这样就没法缓存 hash 值了。所以 String 不可变，hash 值就不可变，只需要计算一次 String 的 hash 值就行了。

2. String Pool 的需要

说到 String Pool，我们来说下什么是 String Pool，String Pool 的中文名叫`字符串常量池`，它是 Java 堆内存中一个特殊的存储区域，怎么特殊呢？

```java
String s1 = "abcd";
String s2 = "abcd";
System.out.println(s1 == s2);
```

上面代码的输出结果是 true。对引用来说，`==` 比较的是引用所指向对象在堆中的内存地址，也就是说，上面代码是在比较 s1 指向的字符串对象和 s2 所指向的字符串对象的内存地址是否相同。而结果是 true，说明相同。

我们画图解析下，先看第一句代码 `String s1 = "abcd"`



<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/StringPool/1.jpg"/>
</p>



首先，在堆（Heap）内存的字符串常量池中创建了一个 `"abcd"`字符串对象，然后 String 声明了一个指针 `s1`，这个指针是存储在栈（Stack）上的。然后指针指向 `"abcd"` 对象，或者说，指针 `s1` 中存放了 `abcd` 字符串对象的地址。



接着看第二句代码，`String s2 = "abcd"`：



<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/StringPool/2.jpg"/>
</p>

首先，程序会判断堆（Heap）内存的字符串常量池中是否有 `"abcd"`字符串对象，然后发现有这个对象。然后 String 声明了一个指针 `s1`，这个指针是存储在栈（Stack）上的。然后指针指向字符串常量池中已有的 `"abcd"` 对象，或者说，指针 `s2` 中存放了 `abcd` 字符串对象的地址。



我们发现，s1 和 s2 指向的是同一个对象。所以 s1 == s2 这句代码的执行结果是 true。我们也可以发现字符串常量池的作用，它的作用就是缓存字符串，如果发现已经有了这个字符串对象，就直接拿来用，不用再创建了。



到这里，String Pool（字符串常量池）就介绍完了。



如果 String 被设计成可变的，那么 String Pool 就没什么用了。看下这段代码：

```java
String s1 = "abcd";
String s2 = "abcd";
s1 = "adc";
```

