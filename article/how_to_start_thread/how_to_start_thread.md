### 创建多线程的三种方式 ###

#### 1. 继承 Thread 类 ####

新建一个类，继承自 Thread，然后重写它的 `run` 方法。

```java
public class FirstThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

FirstThread thread = new FirstThread();
thread.start();
```

#### 2. 实现 Runnable 接口 ####

新建一个类，实现 Runnable 接口，然后在 Runnable 接口的 run 方法中实现自己的逻辑。

```java
public class SecondThread implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

Thread thread = new Thread(new SecondThread());
thread.start();
```

#### 3. Callable ####

Callable 我们可能用到不是太多，它和 Runnable 有点相似，我们看看它的定义：

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

可以看到，Callable 接口中只有一个 call 方法，和 Runnable 类似，Runnable 接口中也只有一个 run 方法。但是有个不同，Callable 是支持泛型的，泛型类型就是客户程序传递进来的 V 类型。另外，call 方法有返回值，而 Runnable 的 run 方法没有返回值。

我们来看下它的用法：

```java
public class ThirdThread implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName());
        return 1;
    }
}

FutureTask<Integer> ft = new FutureTask<Integer>(new ThirdThread());
Thread thread = new Thread(ft);
thread.start();
```

新建一个类，实现 Callable 接口，泛型类型指定为 Integer。然后把创建出的对象传递到 FutureTask 中，再把 FutureTask 对象传递到 Thread 中。

刚刚我们说了，Callable 是有返回值的，怎么看到返回值呢？调用 FutureTask 的 get 方法就可以看到返回值了：

```java
FutureTask<Integer> ft = new FutureTask<Integer>(new ThirdThread());
System.out.println(ft.get());
```

### 区别和联系 ###

`继承 Thread 类` 和 `实现 Runnable 接口` 相比，继承 Thread 类无法共享资源，而实现 Runnable 接口可以。

`实现 Runnable 接口` 和 `实现 Callable 接口` 相比，实现 Runnable 接口没有返回值，而 Callable 接口有返回值。运行 Callable 任务可以拿到一个 Future 对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过 Future 对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。 