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
transient Node<E> first;
transient Node<E> last;

public LinkedList() {
}
```

我们发现，LinkedList 的构造方法里什么也没做。不对，其实它还是做了点事情的，做了什么事呢？它把 `first` 和 `last` 节点都置为 null 了，注意，它并没有显式的把 `first` 和 `last` 置为 null，回忆一下 java 基础：

> 当创建一个类的对象时，引用型成员变量会被初始化为 null。

我们还要注意一点，first 和 last 都是节点，那么它们被置为 null，它们里面的 prev，next，item 全被置为 null。也就是这样的：


<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList%E5%88%9D%E5%A7%8B%E5%8C%96.jpg"/>
</p>

接着看 `linkedList.add(1)`，我们看一下 add 的源码：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    // 1
    final Node<E> l = last;
    // 2
    final Node<E> newNode = new Node<>(l, e, null);
    // 3
    last = newNode;
    if (l == null)
        // 4
        first = newNode;
    else
        // 5
        l.next = newNode;
    // 链表元素数量 +1
    size++;
    // 链表修改次数 + 1
    modCount++;
}
```

首先看第一句代码 `final Node<E> l = last`，由于 last 为 null，所以 l 也为 null。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/last%E4%B8%BAnull%E6%97%B6%EF%BC%8C%E8%8A%82%E7%82%B9l%E6%8C%87%E5%90%91%E8%8A%82%E7%82%B9last.jpg"/>
</p>

接着看第二句代码 `final Node<E> newNode = new Node<>(l, e, null)`，这句代码就稍稍有点复杂了。

首先执行 `new Node<>(l, e, null)` 创建新节点，并将 l （此时 l 是 null）作为新节点的前置节点，将 null 做为新节点的后置节点，由于是 new 出来的节点，所以节点会被分配内存地址，这里假设内存地址是 `0x00000001`：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/l%E4%B8%BA%E7%A9%BA%E6%97%B6LinkedList%E5%88%9B%E5%BB%BA%E6%96%B0%E8%8A%82%E7%82%B9.jpg"/>
</p>

接着看第三句代码 `last = newNode`，这句代码比较好理解，就是让 last 节点指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/last%E6%8C%87%E5%90%91newNode.jpg"/>
</p>

接着看，因为 l 此时为 null，所以接下来会执行代码 4，也就是 `first = newNode`，这时 first 节点也指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/first%E6%8C%87%E5%90%91newNode.jpg"/>
</p>

到这里，first（头节点）和 last（尾节点）都指向了新节点。`linkedList.add(1)` 这句代码也执行完了。

这里我们可以看到，链表里已经有了一个节点了，这个节点中的值是 1。

接着看 `linkedList.add(2)` 这句代码：

第一句代码 `final Node<E> l = last`，由于 last 此时指向了 newNode 节点，所以 l 也指向 newNode 节点。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/l%E6%8C%87%E5%90%91newNode.jpg"/>
</p>

第二句代码 `final Node<E> newNode = new Node<>(l, e, null)`，首先执行 `new Node<>(l, e, null)` 创建新节点，并将 l 作为新节点的前置节点，所以会把 l 的地址存在新节点的前置节点里，将 null 作为新节点的后置节点，由于是 new 出来的节点，所以节点会被分配内存地址，这里假设内存地址是 `0x00000003`（链表的节点地址在内存中不是连续的）：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/l%E4%B8%8D%E4%B8%BA%E7%A9%BA%E6%97%B6LinkedList%E5%88%9B%E5%BB%BA%E6%96%B0%E8%8A%82%E7%82%B9.jpg"/>
</p>

第三句代码 `last = newNode`，这时 last 节点指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/last%E6%8C%87%E5%90%91newNode_2.jpg"/>
</p>


因为 l 此时不为 null，所以接下来执行代码 5，也就是 `l.next = newNode`，这时 l 节点的后置节点会指向刚刚创建出来的 newNode 节点（l 节点的后置节点中保存 newNode 的内存地址）：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/l%E5%90%8E%E7%BD%AE%E8%8A%82%E7%82%B9%E6%8C%87%E5%90%91newNode.jpg"/>
</p>

这里我们可以看到，链表里现在有两个节点了。头节点的值是 1，第二个节点的值是 2。图中还有个 `prev`、`item`、`next` 都为 null 的节点，这个节点由于没有引用指向它，所以很快会被 gc 回收。

到这里，向 LinkedList 中添加元素的过程我们就知道了。