ArrayList 算是我们用的最多的集合了，先看下它的不带参的构造方法：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

在这个构造方法里，ArrayList 把它内置的一个空数组 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 赋值给了 `elementData` 变量。也就是说，当我们调用 `ArrayList list = new ArrayList()` 时，在 ArrayList 内部也构建好了一个空数组。

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