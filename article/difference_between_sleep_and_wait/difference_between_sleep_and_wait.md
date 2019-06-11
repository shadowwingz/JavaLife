### sleep 和 wait 方法的区别

#### sleep 方法

sleep() 方法会使线程进入停滞状态（阻塞当前线程），让出 cpu 的使用。如果在一个 synchronized 块中调用 sleep 方法，线程虽然会休眠，但线程并不会释放锁，其他线程依然无法访问这个对象。

#### wait 方法

wait 方法必须在 synchronized 块中调用，调用之后线程会释放锁，此时其他线程可以访问这个对象。

### 区别：

sleep 睡眠时，不会释放锁
wait 睡眠时，会释放锁

### 验证：

```java
public class ThreadTest implements Runnable {

    int number = 10;

    public void firstMethod() {
        synchronized (this) {
            number += 100;
            System.out.println(number);
        }
    }

    public void secondMethod() throws InterruptedException {
        synchronized (this) {
            Thread.sleep(2000);
//            wait(2000);
            number *= 200;
        }
    }

    @Override
    public void run() {
        firstMethod();
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadTest threadTest = new ThreadTest();
        Thread thread = new Thread(threadTest);
        thread.start();
        threadTest.secondMethod();
    }
}
```

如果执行 `Thread.sleep(2000)`，执行结果：

> 2100

如果执行 `wait(2000)`，执行结果：

> 110

### 分析

> 执行 `Thread.sleep(2000)`

首先，会执行 `secondMethod` 方法，`secondMethod` 方法此时是作为一个普通方法来执行的，也就是说，并不是在一个新的线程中执行的，而是在主线程中执行的。在 `secondMethod` 方法中，会取得锁，然后睡眠 2 秒，此时并【不会】释放锁，接着，ThreadTest 线程启动，调用 firstMethod 方法，ThreadTest 对象想取得锁，但此时锁被主线程占用，所以 ThreadTest 对象无法取得锁。等 `secondMethod` 方法执行完后，number = 200 * 10 = 2000，主线程释放锁，ThreadTest 对象取得锁，number = number + 100 = 2100。打印 number。

> 执行 `wait(2000)`

首先，会执行 `secondMethod` 方法，`secondMethod` 方法此时是作为一个普通方法来执行的，也就是说，并不是在一个新的线程中执行的，而是在主线程中执行的。在 `secondMethod` 方法中，会取得锁，然后睡眠 2 秒，此时【会】释放锁，接着，ThreadTest 线程启动，调用 firstMethod 方法，ThreadTest 对象会取得锁，number = number + 100 = 110，打印 number。等 `secondMethod` 方法睡眠完后，number = number * 200 = 22000，但是此时不会再打印 number 了。