#### 要求 ####

让 3 个线程按照顺序打印 ABC

#### 分析 ####

要让线程按顺序打印，需要用到 join 方法，join 方法的作用是让 join 后面的代码等调用 join 的线程执行完了再执行。比如调用了 thread1.join，那么后面的代码需要等到 thread1 执行完毕才能执行。

#### 代码 ####

```java
public class Task implements Runnable {

    private String name;

    public Task(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name);
    }
}

public class JoinTest {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Task("A"));
        Thread thread2 = new Thread(new Task("B"));
        Thread thread3 = new Thread(new Task("C"));
        thread1.start();
        // thread1 执行完后，后面的代码才能执行
        thread1.join();
        thread2.start();
        // thread2 执行完后，后面的代码才能执行
        thread2.join();
        thread3.start();
        // thread3 执行完后，后面的代码才能执行
        thread3.join();
    }
}
```

执行结果：

```
A
B
C
```