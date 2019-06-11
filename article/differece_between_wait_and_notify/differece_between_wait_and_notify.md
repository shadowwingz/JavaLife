### wait 和 notify 的区别 ###

#### wait ####

wait 方法会释放对象锁，并阻塞当前线程，线程要想继续执行，需要等待 notify 来唤醒

#### notify ####

唤醒一个处于 wait 状态的线程，并拿到对象锁

### 总结 ###

严格来说，wait 和 notify 不应该说有什么区别，它们本身就是对应的。

- wait 释放锁，notify 获取锁
- wait 阻塞线程，notify 让线程继续执行

一次 notify 只能唤醒一个处于 wait 状态的线程，假如有两个线程处于 wait 状态，调用一次 notify，只会唤醒其中的一个线程，具体唤醒哪一个线程由虚拟机控制。