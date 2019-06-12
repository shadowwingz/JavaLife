### 什么是生产者/消费者模型 ###

生产者/消费者模型描述的是有一块缓冲区作为仓库，生成者将产品放入仓库，消费者从仓库中取出产品。有几点需要注意：

- 生产者生产的时候消费者不能消费
- 消费者消费的时候生产者不能生产
- 缓冲区空的时候消费者不能消费
- 缓冲区满的时候生产者不能生产

### 优点 ###

- 解耦。

生产者和消费者之间并不直接调用，而是会通过缓冲区来调用。这样，生产者或者消费者的代码一旦发生变动，对彼此的影响会比较小。

#### 利用 wait / notify 实现生产者/消费者模型 ####

刚刚我们说了，生产者/消费者模型有一个缓冲区，那我们就用一个容量为 5 的 List 作为缓冲区，List 为空表示缓冲区空，List 中元素数量为 5 的时候表示缓冲区满。

生产者，当缓冲区满的时候就 wait，不生产了，等消费者消费完通知再生产，如果缓冲区是空的，就生产。

```java
public class Producer extends Thread {

    // 缓冲区
    private List<String> msgList = new ArrayList<>();

    public Producer(List<String> msgList) {
        this.msgList = msgList;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (msgList) {
                // 缓冲区中有 5 个产品时，表示缓冲区满了，这时停止生产
                if (msgList.size() == 5) {
                    System.out.println("缓冲区满，停止生产");
                    try {
                        // wait 会休眠当前线程，并释放对象锁，这样消费者才能获取锁，才能消费
                        msgList.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 当生产者从 wait 中醒来时，此时消费者已经把缓冲区的产品消费完了，这时继续生产
                System.out.println("生产者生产");
                msgList.add("产品");
                // 生产产品后，唤醒消费者来消费
                msgList.notify();
            }
        }
    }
}
```

消费者，当缓冲区为空的时候就 wait，不消费了，等生产者生产完通知再消费，如果缓冲区是满的，就消费。

```java
public class Consumer extends Thread {

    // 缓冲区
    private List<String> msgList;

    public Consumer(List<String> msgList) {
        this.msgList = msgList;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (msgList) {
                // 缓冲区为空时，表示缓冲区已经没有产品了，这时停止消费
                if (msgList.isEmpty()) {
                    System.out.println("缓冲区空，停止消费");
                    try {
                        // wait 会休眠当前线程，并释放对象锁，这样生产者才能获取锁，才能生产
                        msgList.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 当消费者从 wait 中醒来时，此时生产者已经把产品都生产出来了，放到缓冲区了，这时继续消费
                System.out.println("消费者消费");
                msgList.remove(0);
                // 消费产品后，唤醒生产者来生产
                msgList.notify();
            }
        }
    }
}
```

主函数：

```java
public class Test {
    public static void main(String[] args) {
        List<String> msgList = new ArrayList<>();
        Producer producer = new Producer(msgList);
        Consumer consumer = new Consumer(msgList);
        producer.start();
        consumer.start();
    }
}
```

运行结果：

```
生产者生产
生产者生产
生产者生产
生产者生产
生产者生产
缓冲区满，停止生产
消费者消费
消费者消费
消费者消费
消费者消费
消费者消费
缓冲区空，停止消费
生产者生产
生产者生产
生产者生产
生产者生产
生产者生产
缓冲区满，停止生产
......
```