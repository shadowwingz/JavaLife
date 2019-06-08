### Thread 和 Runnable的区别 ###

我们一般创建线程，要么继承 Thread，要么实现 Runnable 接口，那么这两种方式有什么区别呢？

- 单继承局限性

由于 Java 中是单继承，所以如果一个类继承了 Thread，那么它就无法再继承别的类了，这样会有很大的局限性。而 Runnable 是一个接口，Java 中可以实现多个接口，所以一个类实现了 Runnable 接口，还可以实现别的接口，也不影响它继承别的类。

- 资源共享

我们写个简单的例子来解释一下：

#### 继承 Thread ####

```java
public class MyThread1 extends Thread {

    private int count = 1;

    private String name;

    public MyThread1(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("线程 " + name + " " + count++);
        }
    }
}

public class MyThread2 extends Thread {

    private int count = 1;

    private String name;

    public MyThread2(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("线程 " + name + " " + count++);
        }
    }
}

MyThread1 thread1 = new MyThread1("A");
MyThread2 thread2 = new MyThread2("B");
thread1.start();
thread2.start();
```

运行结果：

```
线程 A 1
线程 B 1
线程 B 2
线程 B 3
线程 B 4
线程 B 5
线程 A 2
线程 A 3
线程 A 4
线程 A 5
```

我们定义了两个 Thread，分别是 MyThread1，MyThread2。MyThread1 和 MyThread2，每次 count++ 都是在各自的线程中进行操作。

#### 实现 Runnable 接口 ####

```java
public class MyRunnable implements Runnable {

    private int count = 1;

    private String name;

    public MyRunnable(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("线程 " + name + " " + count++);
        }
    }
}

MyRunnable runnable = new MyRunnable("A");
Thread thread1 = new Thread(runnable);
Thread thread2 = new Thread(runnable);
thread1.start();
thread2.start();
```

运行结果：

```
线程 A 1
线程 A 2
线程 A 3
线程 A 4
线程 A 5
线程 A 6
线程 A 7
线程 A 8
线程 A 9
线程 A 10
```

thread1 和 thread2 之间的变量 count 是共享的