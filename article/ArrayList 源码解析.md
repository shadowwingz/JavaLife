ArrayList 算是我们用的最多的集合了，先看下它的不带参的构造方法：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

在这个构造方法里，ArrayList 把它内置的一个空数组 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 赋值给了 `elementData` 变量。也就是说，当我们调用 `ArrayList list = new ArrayList()` 时，在 ArrayList 内部也构建好了一个空数组。从这里我们可以发现，ArrayList 内部是用数组实现的。

### 增 ###

再看一下它的 `add(E e)` 方法：

```java
private static final int DEFAULT_CAPACITY = 10;

public boolean add(E e) {
    // size 是 ArrayList 当前已有元素容量
    // size + 1 是 ArrayList 为了能装下 add 的这个元素，必须拥有的最小容量
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 把新元素添加到 ArrayList 中
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    // 如果 elementData 是空数组
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 确定一个最小容量，
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    // 最终确定 ArrayList 容量
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    // 如果刚刚确定的最小容量比 ArrayList 现有容量大，
    // 就说明，ArrayList 需要扩容才能装的下这个元素，                                      
    if (minCapacity - elementData.length > 0)
        // 给 ArrayList 扩容
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    // 旧容量
    int oldCapacity = elementData.length;
    // oldCapacity >> 1 等同于 oldCapacity / 2
    // 新容量是旧容量的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    // ArrayList 扩容 1.5 倍
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

总结一下，当添加一个元素的时候，ArrayList 会判断当前的容量够不够装下这个新元素，如果不够，就扩容 1.5 倍。说到扩容，我们看下扩容的具体实现，在上面的代码中，ArrayList 是通过 `Arrays.copyOf` 方法来扩容的，在内部会调用 `System.arraycopy` 方法：

```
System # arraycopy

// src 是源数组
// scrPos 是源数组的起始位置
// dest 是目标数组，在这里是扩容后的数组
// destPos 是目标数组的起始位置
// length 是要复制的长度
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

从名字上看，这个方法是用来拷贝数组的，具体的拷贝方式直接用一张图来解释。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/arraycopy%20%E5%9B%BE%E8%A7%A3.jpg"/>
</p>


再看下 `addAll` 方法，这个方法是用来增加一个集合，也就是把一个 ArrayList 添加到另一个 ArrayList 中：

```java
public boolean addAll(Collection<? extends E> c) {
    // 集合转换为数组
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

我们发现，传进来的 ArrayList 被 `toArray` 方法转换成了数组，然后再检查需不需要扩容，然后调用 `System.arraycopy` 把传进来的 ArrayList 追加到源数组后面，接着更新 ArrayList 的大小。

### 删 ###

```java
public boolean remove(Object o) {
    // 要删除的对象如果为 null
    if (o == null) {
        // 遍历
        for (int index = 0; index < size; index++)
            // 如果发现有元素为 null
            if (elementData[index] == null) {
                // 就删除这个元素
                fastRemove(index);
                return true;
            }
    } else { // 要删除的对象不为 null
        for (int index = 0; index < size; index++)
            // 如果发现有元素和要删除的元素相同
            if (o.equals(elementData[index])) {
                // 就删除这个元素
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    // 要移动的元素数量
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 拷贝数组
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

首先，ArrayList 会遍历它的所有元素，然后和要删除的元素比较，不管这个要删除的元素是不是 null，如果发现有元素和要删除的元素相同，就删除。删除调用的是 `System.arraycopy` 方法。有的童鞋可能会疑惑，删除就删除，干嘛要拷贝数组呢？

首先，让我们想一想，如果是我们来实现这个删除元素功能，我们会怎么实现。我的第一反应是直接把索引为 index 的元素置为 null。但是这样会有一个问题，我们是要把索引为 index 的元素给删除掉，现在仅仅是把它置为 null 而已，元素还在那里。我们调用 list.get(index) 还是能获取到那个元素。所以这个方法不可行。

ArrayList 的方法是从源数组中要删除的元素的后一位开始，拷贝后面的数组，拷贝到源数组中要删除的元素的前一位，这样，要删除的元素就在拷贝过程中被丢弃了。这样描述可能比较抽象，用张图解释下：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/JavaLife/master/art/ArrayList%20%E5%88%A0%E9%99%A4%E5%85%83%E7%B4%A0.jpg"/>
</p>

### 改 ###

ArrayList 修改元素的方法是 `set` 方法：

```java
// index 是待修改元素的索引
// element 是新元素
public E set(int index, E element) {
    // 检查索引是否合法
    rangeCheck(index);

    // 取出待修改元素
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

E elementData(int index) {
    return (E) elementData[index];
}
```

可以看到，修改元素的逻辑还是比较简单的，先检查待修改元素的索引，是否小于 ArrayList 中已有元素的数量，如果不小于，就抛异常。

如果索引合法，就先取出待修改元素，然后把新元素赋值给对应索引。最后返回待修改元素。

### 查 ###

ArrayList 查询元素的方法是 `get` 方法：

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

E elementData(int index) {
    return (E) elementData[index];
}
```

可以看到，查询元素的逻辑也是比较简单，先检查索引是否合法，如果合法，就返回索引对应的元素。

## 总结 ##

ArrayList 的内部实现使用的是数组，所以查询元素和修改元素的效率很高，而增加元素和删除元素的效率较低，因为在增加或者删除的过程中都要拷贝一次数组。