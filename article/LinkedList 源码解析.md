和 ArrayList 不同，LinkedList 内部并不是用数组实现的，而是用链表实现的。

回忆一下链表的特点：

增删元素只需要移动指针，所以效率比较高，不需要像 ArrayList 一样，还要给数组扩容，然后拷贝数组。

但是查找元素时效率比较低，因为要一个节点一个节点的找。

说到节点，我们来看下 LinkedList 里的节点：

```java
private static class Node<E> {
    // 元素值
    E item;
    // 后置节点
    Node<E> next;
    // 前置节点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

用张图表示：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList%20Node.jpg"/>
</p>


从图中可以看出，item 才是节点里真正存储的数据。而 `prev` 表示的是这个节点的前一个节点的引用地址，`next` 表示的是这个节点的后一个节点的引用地址。

从 `prev` 和 `next` 也可以看出，LinkedList 内部是一个双向链表。也就是说，链表中任何一个节点都可以通过向前或者向后寻址的方法获取到前一个节点和后一个节点。

### 添加元素 ###

我们来看下 LinkedList 是怎么添加元素的，先看下面一段代码：

```java
public static void main(String[] args) {
    LinkedList<Integer> linkedList = new LinkedList<>();
    linkedList.add(1);
    linkedList.add(2);
}
```

我们逐行分析下代码是怎么执行的。首先看第一行 `LinkedList<Integer> linkedList = new LinkedList<>()`，看一下 LinkedList 的无参构造方法：

```java
public LinkedList() {
}
```

我们发现，LinkedList 的构造方法里什么也没做。不对，其实它还是做了点事情的，做了什么事呢？它把 `first` 和 `last` 都置为 null 了，注意，它并没有显示的把 `first` 和 `last` 置为 null，回忆一下 java 基础：

> 当创建一个类的对象时，引用型成员变量会被初始化为 null。


===================================================

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    // 记录原尾部节点
    final Node<E> l = last;
    // 生成新节点
    final Node<E> newNode = new Node<>(l, e, null);
    // 更新尾部节点
    last = newNode;
    // 如果原链表是空链表
    if (l == null)
        // 头节点指向新节点
        first = newNode;
    // 如果原链表不是空链表
    else
        // 尾节点的后置节点指向新节点
        l.next = newNode;
    size++;
    modCount++;
}
```

总结一下，就是让 LinkedList 的尾节点的后置节点

