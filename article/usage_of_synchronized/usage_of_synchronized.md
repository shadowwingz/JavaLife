synchronzied 是一种同步锁，它可以用来

- 修饰一个代码块
- 修饰一个方法
- 修饰一个静态方法
- 修饰一个类

下面，我们来看看各有什么区别

#### 修饰一个代码块 ####

当一个线程访问一个对象中的 `synchronized(this)` 时，其他想要访问该对象的线程将被阻塞。

```java
public class SyncThread implements Runnable {

    private static int count;

    public SyncThread() {
        count = 0;
    }

    @Override
    public void run() {
        synchronized (this) {
            for (int i = 0; i < 5; i++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

public class Test {
    public static void main(String[] args) {
        SyncThread syncThread = new SyncThread();
        Thread thread1 = new Thread(syncThread, "SyncThread1");
        Thread thread2 = new Thread(syncThread, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

执行结果：

```
SyncThread1:0
SyncThread1:1
SyncThread1:2
SyncThread1:3
SyncThread1:4
SyncThread2:5
SyncThread2:6
SyncThread2:7
SyncThread2:8
SyncThread2:9
```

可以看到，结果是串行执行的，thread1 先执行，执行完了之后 thread2 再执行。

thread1 先访问 synchronized 代码块并加锁，thread1 执行完 synchronized 代码块之后释放锁，thread2 才能访问 synchronized 代码块。

当然，前提是两个线程访问的是同一个对象，加锁才有用。我们修改下代码：

```java
SyncThread syncThread1 = new SyncThread();
SyncThread syncThread2 = new SyncThread();
Thread thread1 = new Thread(syncThread1, "SyncThread1");
Thread thread2 = new Thread(syncThread2, "SyncThread2");
thread1.start();
thread2.start();
```

我们让 thread1 访问 syncThread1 对象，thread2 访问 syncThread2 对象。

执行结果：

```
SyncThread1:0
SyncThread2:1
SyncThread2:2
SyncThread1:3
SyncThread1:4
SyncThread2:4
SyncThread1:5
SyncThread2:5
SyncThread2:6
SyncThread1:6
```

可以看到，两个线程是同时执行的。这是因为 thread1 执行的是 syncThread1 对象的同步代码块，而 thread2 执行的是 syncThread2 对象的同步代码块。两把锁互不干扰，所以两个线程可以同时执行。

#### 修饰一个方法 ####

写法：

```java
public synchronized void method() {
    // todo
}
```

等同于：

```java
public void method() {
    synchronized (this) {
        // todo
    }
}
```

这里 synchronized 修饰的是一个普通方法，并不是静态方法。普通方法的特点是锁的是对象，也就是说，假设两个线程去访问两个对象，锁就不起作用了。静态方法的特点是锁的是类，无论创建多少个对象，锁都会起作用。

#### 修饰一个静态方法 ####

写法：

```java
public synchronized static void method() {
    // todo
}
```

我们来验证一下，修饰静态方法，锁的是类。

```java
public class SyncThread implements Runnable {

    private static int count;

    public SyncThread() {
        count = 0;
    }

    @Override
    public void run() {
        method();
    }

    public synchronized static void method() {
        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(Thread.currentThread().getName() + ":" + (count++));
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Test {
    public static void main(String[] args) {
        SyncThread syncThread1 = new SyncThread();
        SyncThread syncThread2 = new SyncThread();
        Thread thread1 = new Thread(syncThread1, "SyncThread1");
        Thread thread2 = new Thread(syncThread2, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

这里我们创建了两个对象，syncThread1 和 syncThread2，分别让 thread1 和 thread2 去访问。

执行结果：

```
SyncThread1:0
SyncThread1:1
SyncThread1:2
SyncThread1:3
SyncThread1:4
SyncThread2:5
SyncThread2:6
SyncThread2:7
SyncThread2:8
SyncThread2:9
```

两个线程是串行的，这也证实了我们刚才说的，静态方法的锁，锁的是类，无论创建多少个对象，都相当于用的是同一个锁。

#### 修饰一个类 ####

写法：

```java
public void method() {
    synchronized (xx.class) {
        // todo
    }
}
```

修饰一个类的效果，和静态方法的效果是一样的，都是对类加锁，我们来验证一下：

```java
public class SyncThread implements Runnable {

    private static int count;

    public SyncThread() {
        count = 0;
    }

    @Override
    public void run() {
        method();
    }

    public void method() {
        synchronized (SyncThread.class) {
            for (int i = 0; i < 5; i++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

public class Test {
    public static void main(String[] args) {
        SyncThread syncThread1 = new SyncThread();
        SyncThread syncThread2 = new SyncThread();
        Thread thread1 = new Thread(syncThread1, "SyncThread1");
        Thread thread2 = new Thread(syncThread2, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

执行结果：

```
SyncThread1:0
SyncThread1:1
SyncThread1:2
SyncThread1:3
SyncThread1:4
SyncThread2:5
SyncThread2:6
SyncThread2:7
SyncThread2:8
SyncThread2:9
```

可以看到，修饰类，和修饰静态方法一样，都是对类加锁。