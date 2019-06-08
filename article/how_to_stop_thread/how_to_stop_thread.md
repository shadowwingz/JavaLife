### 如何在 Java 中正确的终止一个线程 ###

看到这个问题，我们的第一反应就是 Thread 的 `stop` 方法，但是 `stop` 方法比较暴力，被结束的线程会被立即停止，不管它有没有执行完它的任务，而且也没有时间处理一些善后工作，比如回收内存。

那么怎么才能更好的结束一个线程？

我们可以从另一个角度来思考这个问题：我们可以控制何时让线程的任务停止执行，至于线程是否停止，我们可以不用关心。

```java
public class MyThread extends Thread {

    private volatile boolean finished = false;

    public void stopMe() {
        finished = true;
    }

    @Override
    public void run() {
        super.run();
        while (!finished) {
            System.out.println("==");
        }
        System.out.println("stop");
    }
}

MyThread thread = new MyThread();
thread.start();
Thread.sleep(1000);
thread.stopMe();
```

通过条件变量 `finished` 控制线程的执行，线程内部检查 `finished` 变量状态，外部改变 `finished` 变量值可控制任务停止执行。为保证线程间的即时通信，需要使用 `volatile` 关键字或锁，确保读线程与写线程间变量状态一致。