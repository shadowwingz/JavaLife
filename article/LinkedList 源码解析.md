和 ArrayList 不同，LinkedList 内部并不是用数组实现的，而是用链表实现的。

回忆一下链表的特点：

增删元素只需要移动指针，所以效率比较高，不需要像 ArrayList 一样，还要给数组扩容，然后拷贝数组。

但是查找元素时效率比较低，因为要一个节点一个节点的找。

我们先看下构造方法：

```java
public LinkedList() {
}
```

我们发现，LinkedList 的构造方法里什么也没做。

再看下 LinkedList 里的节点：

```
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

从前置节点和后置节点可以看出，LinkedList 内部是一个双向链表。

### 增 ###

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
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```