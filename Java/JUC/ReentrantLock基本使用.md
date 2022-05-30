# ReentrantLock

## 基本语法

```java
	// 获取锁
	reentrantLock.lock();
	try {
    	// 临界区
    } finally {
    	// 释放锁
    	reentrantLock.unlock();
    }
```

## 可重入

- 可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁

- 如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

```java
    static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        method1();
    }
    public static void method1() {
        lock.lock();
        try {
            System.out.println("execute method1");
            method2();
        } finally {
            lock.unlock();
        }
    }
    public static void method2() {
        lock.lock();
        try {
            System.out.println("execute method2");
            method3();
        } finally {
            lock.unlock();
        }
    }
    public static void method3() {
        lock.lock();
        try {
            System.out.println("execute method3");
        } finally {
            lock.unlock();
        }
    }
```

```java
17:59:11.862 [main] c.TestReentrant - execute method1 
17:59:11.865 [main] c.TestReentrant - execute method2 
17:59:11.865 [main] c.TestReentrant - execute method3
```

## 可打断

```java
	ReentrantLock lock = new ReentrantLock();
    Thread t1 = new Thread(() -> {
        System.out.println("t1启动...");
        try {
            // 着重注意此处
            lock.lockInterruptibly();
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.out.println("t1等锁的过程中被打断");
            return;
        }
        try {
            System.out.println("t1获得了锁");
        } finally {
            lock.unlock();
        }
    }, "t1");
    lock.lock();
    System.out.println("主线程获得了锁");
    t1.start();
    try {
        Thread.sleep(1000);
        t1.interrupt();
        System.out.println("主线程执行打断");
    } finally {
        lock.unlock();
    }
```

```java
主线程获得了锁
t1启动...
主线程执行打断
t1等锁的过程中被打断
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at cat.monster.code.first.RTEst.lambda$main$0(RTEst.java:11)
	at java.lang.Thread.run(Thread.java:748)
```

如果将t1线程不适用lockInterruptibly()，而使用lock()

```java
        Thread t1 = new Thread(() -> {
            System.out.println("t1启动...");
            lock.lock();
            try {
                System.out.println("t1获得了锁");
            } finally {
                lock.unlock();
            }
        }, "t1");
```

```java
主线程获得了锁
t1启动...
主线程执行打断
t1获得了锁
```

即使主线程执行了打断，也不会让t1中断

## 锁超时

```java
ReentrantLock lock = new ReentrantLock();
Thread t1 = new Thread(() -> {
    System.out.println("t1启动...");
    if (!lock.tryLock()) {
        System.out.println("t1获取锁失败");
        return;
    }
    try {
        System.out.println("t1获得了锁");
    } finally {
        lock.unlock();
    }
}, "t1");
lock.lock();
System.out.println("主线程获得了锁");
t1.start();
try {
    Thread.sleep(1000);
} finally {
    lock.unlock();
}
```

lock.tryLock()会去尝试获得锁，返回true/false。

tryLock()还有一个重载方法lock.tryLock(long timeout, TimeUnit unit)

```java
ReentrantLock lock = new ReentrantLock();
Thread t1 = new Thread(() -> {
    System.out.println("t1启动...");
    try {
        if (!lock.tryLock(1, TimeUnit.SECONDS)) {
            System.out.println("t1获取锁失败");
            return;
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    try {
        System.out.println("t1获得了锁");
    } finally {
        lock.unlock();
    }
}, "t1");
lock.lock();
System.out.println("主线程获得了锁");
t1.start();
try {
    Thread.sleep(500);
} finally {
    lock.unlock();
}
```

```java
主线程获得了锁
t1启动...
t1获得了锁
```

## 公平锁

ReentrantLock 默认是不公平的，如果要实现公平，就需要花费资源去维护，所以公平锁会有一定的性能损耗

公平锁一般没有必要，会降低并发度，后面分析原理时会讲解

```java
        ReentrantLock lock = new ReentrantLock(false);
        lock.lock();
        for (int i = 0; i < 500; i++) {
            new Thread(() -> {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " running...");
                } finally {
                    lock.unlock();
                }
            }, "t" + i).start();
        }
        // 1s 之后去争抢锁
        Thread.sleep(1000);
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " start...");
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " running...");
            } finally {
                lock.unlock();
            }
        }, "强行插入").start();
        lock.unlock();
```

```java
t0 running...
强行插入 start...
强行插入 running...
t1 running...
t2 running...
t3 running...
```

如果是非公平锁，强行插入，有机会在中间输出

改为公平锁后

```java
ReentrantLock lock = new ReentrantLock(true);
```

```java
t496 running...
t497 running...
t498 running...
t499 running...
强行插入 running...
```

### 使用tryLock解决哲学家就餐问题

```java
public static void main(String[] args) throws InterruptedException {
        Chopstick c1 = new Chopstick("1");
        Chopstick c2 = new Chopstick("2");
        Chopstick c3 = new Chopstick("3");
        Chopstick c4 = new Chopstick("4");
        Chopstick c5 = new Chopstick("5");

        new Thread(new Person("第一位",c1,c2)).start();
        new Thread(new Person("第二位",c2,c3)).start();
        new Thread(new Person("第三位",c3,c4)).start();
        new Thread(new Person("第四位",c5,c1)).start();
    }

    static class Chopstick extends ReentrantLock {
        String name;

        public Chopstick(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Chopstick{" +
                    "name='" + name + '\'' +
                    '}';
        }
    }


    static class Person implements Runnable {
        String name;
        Chopstick left;
        Chopstick right;

        public Person(String name, Chopstick left, Chopstick right) {
            this.name = name;
            this.left = left;
            this.right = right;
        }

        private void eat() {
            System.out.println(name + "eating...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void run() {
            while (true) {
                // 尝试获取左手的筷子
                if (left.tryLock()) {
                    try {
                        // 尝试获取有手的筷子
                        if (right.tryLock()) {
                            try {
                                // 左右手筷子都拿到了
                                eat();
                            } finally {
                                // 未获取到就礼让他人
                                right.unlock();
                            }
                        }
                    } finally {
                        // 未获取到就礼让他人
                        left.unlock();
                    }
                }

            }
        }
    }
```

```java
第一位eating...
第三位eating...
第三位eating...
第一位eating...
第四位eating...
第二位eating...
第一位eating...
第三位eating...
第三位eating...
第一位eating...
第三位eating...
第一位eating...
第一位eating...
第三位eating...
```



## 多条件变量

synchronized 中也有条件变量，就是我们讲原理时那个 waitSet 休息室，当条件不满足时进入 waitSet 等待

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比

- synchronized 是那些不满足条件的线程都在一间休息室等消息
- 而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒

使用要点：

- await 前需要获得锁
- await 执行后，会释放锁，进入conditionObject等待
- await 的线程被唤醒（或打断、或超时）需重新竞争 lock 锁
- 竞争 lock 锁成功后，从 await 后继续执行