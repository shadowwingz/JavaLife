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
