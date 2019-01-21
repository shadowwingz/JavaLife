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
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/LinkedList%20Node.jpg"/>
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
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/LinkedList%E5%88%9D%E5%A7%8B%E5%8C%96.jpg"/>
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
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(1)%E6%B5%81%E7%A8%8B/1.jpg"/>
</p>

接着看第二句代码 `final Node<E> newNode = new Node<>(l, e, null)`，这句代码就稍稍有点复杂了。

首先执行 `new Node<>(l, e, null)` 创建新节点，并将 l （此时 l 是 null）作为新节点的前置节点，将 null 做为新节点的后置节点，由于是 new 出来的节点，所以节点会被分配内存地址，这里假设内存地址是 `0x00000001`：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(1)%E6%B5%81%E7%A8%8B/2.jpg"/>
</p>

接着看第三句代码 `last = newNode`，这句代码比较好理解，就是让 last 节点指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(1)%E6%B5%81%E7%A8%8B/3.jpg"/>
</p>

接着看，因为 l 此时为 null，所以接下来会执行代码 4，也就是 `first = newNode`，这时 first 节点也指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(1)%E6%B5%81%E7%A8%8B/4.jpg"/>
</p>

到这里，first（头节点）和 last（尾节点）都指向了新节点。`linkedList.add(1)` 这句代码也执行完了。

这里我们可以看到，链表里已经有了一个节点了，这个节点中的值是 1。

接着看 `linkedList.add(2)` 这句代码：

第一句代码 `final Node<E> l = last`，由于 last 此时指向了 newNode 节点，所以 l 也指向 newNode 节点。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/d77c47cde81fa48513bcf413779ae2f7f1a4603a/art/LinkedList/add(2)%E6%B5%81%E7%A8%8B/1.jpg"/>
</p>

第二句代码 `final Node<E> newNode = new Node<>(l, e, null)`，首先执行 `new Node<>(l, e, null)` 创建新节点，并将 l 作为新节点的前置节点，所以会把 l 的地址存在新节点的前置节点里，所以新节点会指向 l 节点，然后将 null 作为新节点的后置节点，由于是 new 出来的节点，所以节点会被分配内存地址，这里假设内存地址是 `0x00000003`（链表的节点地址在内存中不是连续的）：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(2)%E6%B5%81%E7%A8%8B/2.jpg"/>
</p>

第三句代码 `last = newNode`，这时 last 节点指向了刚刚创建出来的 newNode 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(2)%E6%B5%81%E7%A8%8B/3.jpg"/>
</p>


因为 l 此时不为 null，所以接下来执行代码 5，也就是 `l.next = newNode`，这时 l 节点的后置节点会指向刚刚创建出来的 newNode 节点（l 节点的后置节点中保存 newNode 的内存地址）：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(2)%E6%B5%81%E7%A8%8B/4.jpg"/>
</p>

这里我们可以看到，链表里现在有两个节点了。头节点的值是 1，第二个节点的值是 2。图中还有个 `prev`、`item`、`next` 都为 null 的节点，这个节点由于没有引用指向它，所以很快会被 gc 回收。

终于会变成这样：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/add(2)%E6%B5%81%E7%A8%8B/5.jpg"/>
</p>

到这里，向 LinkedList 中添加元素的过程我们就知道了。

### 删除元素 ###

我们再来看一下 LinkedList 是怎么删除元素的，先看下面一段代码：

```java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("1");
linkedList.add("2");
linkedList.remove("2");
linkedList.remove("1");
```

和添加元素稍微有点不同的是，LinkedList 存储的是 String 类型的元素，而不是 Integer 类型的元素。其实对 LinkedList 来说并没什么区别。`linkedList.add("1")` 和 `linkedList.add("2")` 的代码我们上面已经分析过了，这里就不说了。我们直接分析 `linkedList.remove("2")` 这句代码，我们看一下 `remove(Object o)` 的源码：

```java
public boolean remove(Object o) {
    if (o == null) {
        // 1
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 2
        for (Node<E> x = first; x != null; x = x.next) {
            // 3
            if (o.equals(x.item)) {
                // 4
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

因为 `"2"` 不为 `null`，所以会执行到代码 2。开始从头遍历链表，遍历的方式是创建一个节点 x 指向 first 节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/1.jpg"/>
</p>

接着会执行代码 3，判断 x 节点的 item 是否和 o 对象相等。o 对象就是我们传入的 `"2"`，x 节点的 item 是 `"1"`，所以不相等，因为 `x != null` 成立，所以执行 `x = x.next`，此时节点 x 会指向 first 节点的下一个节点：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/2.jpg"/>
</p>

接着再执行代码 3，判断 x 节点的 item 是否和 o 对象相等。此时 x 节点的 item 是 `"2"`，所以相等，接着执行代码 4，`unlink(x)`：

```java
E unlink(Node<E> x) {
    // assert x != null;
    // 1
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    // 2
    if (prev == null) {
        first = next;
    } else {
        // 3
        prev.next = next;
        x.prev = null;
    }

    // 4
    if (next == null) {
        last = prev;
    } else {
        // 5
        next.prev = prev;
        x.next = null;
    }

    // 6
    x.item = null;
    // LinkedList 元素数量 -1
    size--;
    // LinkedList 修改次数 +1
    modCount++;
    // 返回被删除的元素
    return element;
}
```

首先，看代码 1，代码 1 很简单，就是把 x 节点的 item、next、prev 全部取出来，item 是 2，next 是 null，prev 是节点 first。

接着执行代码 2，这时会判断 prev 是否为 null，因为 prev 是节点 first，不为 null，所以会进入 else，也就是代码 3，`prev.next = next`，这句代码的意思是，把 next 节点的内存地址（null）存入 prev 节点（first 节点）的 next 域中：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/3.jpg"/>
</p>

接着执行 `x.prev = null`，也就是把 x 节点的 prev 置为 null：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/4.jpg"/>
</p>

接着执行代码 4，判断 next 是否为 null，我们之前说过，next 是 null，所以会进入 if 语句，执行 `last = prev`：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/5.jpg"/>
</p>

接着执行代码 6，`x.item = null`：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/LinkedList/remove(2)%E6%B5%81%E7%A8%8B/6.jpg"/>
</p>

到这里，一个元素就被删除了。我们总结一下上面的流程，发现 LinkedList 删除元素需要 2 个步骤：

- 第一，确定要删除的元素
- 第二，把要删除的元素和相邻节点断开连接，也就是把节点的 prev，next，item 全部置为 null