# 三个线程交替顺序打印ABC

## 题目描述

建立三个线程A、B、C，A线程打印10次字母A，B线程打印10次字母B,C线程打印10次字母C，但是要求三个线程同时运行，并且实现交替打印，即按照ABCABCABC的顺序打印。



## 1. 使用synchronized关键字

使用同步代码块和wait、notify方法控制三个线程的执行次序。

目标：ThreadA->ThreadB->ThreadC->ThreadA->···

每个线程必须同时持有两个对象锁，才能进行打印操作：一个对象锁时`prev`，是前一个线程所对应的对象锁，作用是保证当前线程一定是在前一个线程完成后（即前一个线程释放其对应的对象锁）才开始执行。还有一个锁是`self`即自身对象锁。主要是为了控制执行的顺序，必须先持有`prev`锁，再申请自身的对象锁`self`，两者都有的情况下再打印。之后首先调用`self.notify()`唤醒下一个等待线程，再调用`prev.wait()`立即释放`prev`对象锁，当前线程进入休眠。

```java
public class Test {
  public static class ThreadPrinter implements Runnable {
    private String name;
    private Object prev;
    private Object self;

    public ThreadPrinter(String name, Object prev, Object self) {
      this.name = name;
      this.prev = prev;
      this.self = self;
    }

    @Override
    public void run() {
      int count = 10;
      while (count > 0) { // 多线程并发，不能用if，必须用while循环
        synchronized (prev) { // 先获取prev锁
          synchronized (self) { // 再获取self锁
            System.out.print(name);
            count--;

            self.notifyAll(); // 唤醒其他线程竞争self锁，此时self锁并未立即释放
          }
          try {
            if (count == 0) { // 如果count = 0，表示这是最后一次打印操作，通过notifyAll操作释放对象锁
              prev.notifyAll();
            } else {
              prev.wait(); // 立即释放prev锁，当前线程休眠，等待唤醒
            }
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }
  }

  public static void main(String[] args) throws InterruptedException {
      Object a = new Object();
      Object b = new Object();
      Object c = new Object();
      ThreadPrinter pa = new ThreadPrinter("A", c, a);
      ThreadPrinter pb = new ThreadPrinter("B", a, b);
      ThreadPrinter pc = new ThreadPrinter("C", b, c);

      new Thread(pa).start();
      Thread.sleep(10);
      new Thread(pb).start();
      Thread.sleep(10);
      new Thread(pc).start();
      Thread.sleep(10);
  }
}
```



## 2. 使用Lock和state标志

使用状态（State）模式

```java
public class Lock_State_ABC {
    private static Lock lock = new ReentrantLock();
    /** 通过state的值来确定是哪个线程打印 {@state} */
    private static int state = 0;


    static class ThreadA extends Thread{
        @Override
        public void run(){
      for (int i = 0; i < 10; ) {
          try{
              lock.lock();
              while(state % 3 == 0){
                  System.out.println("A");
                  state++;
                  i++;
              }
          }
          finally{
              lock.unlock();
                 }
            }
        }
    }

    static class ThreadB extends Thread{
        @Override
        public void run(){
            for (int i = 0; i < 10; ) {
                try{
                    lock.lock();
                    while(state % 3 == 1){
                        System.out.println("B");
                        state++;
                        i++;
                    }
                }
                finally{
                    lock.unlock();
                }
            }
        }
    }

    static class ThreadC extends Thread{
        @Override
        public void run(){
            for (int i = 0; i < 10; ) {
                try{
                    lock.lock();
                    while(state % 3 == 2){
                        System.out.println("C");
                        state++;
                        i++;
                    }
                }
                finally{
                    lock.unlock();
                }
            }
        }
    }

  public static void main(String[] args) {
        new ThreadA().start();
        new ThreadB().start();
        new ThreadC().start();
        //
  }
}
```

## 3. 使用Lock->ReentrantLock和Condition

使用ReentrantLock一般搭配Condition使用

Condition是被绑定到Lock上的，必须使用lock.newCondition()才能创建一个Condition。从上面的代码可以看出，Synchronized能实现的通信方式，Condition都可以实现.

```java
public class Lock_RentrantLock_ABC {
    private static Lock lock = new ReentrantLock();
    private static Condition A = lock.newCondition();
    private static Condition B = lock.newCondition();
    private static Condition C = lock.newCondition();

    private static int count = 0;

    static class ThreadA extends Thread{
        @Override
        public void run() {
            try{
                lock.lock();
                for(int i = 0; i < 10; i++) {
                    while(count % 3 != 0){
                        A.await();
                    }
                    System.out.println("A");
                    count++;
                    B.signal();

                }
            }
            catch (InterruptedException e){
                e.printStackTrace();
            }
            finally{
                lock.unlock();
            }
        }
    }

    static class ThreadB extends Thread{
        @Override
        public void run() {
            try{
                lock.lock();
                for(int i = 0; i < 10; i++) {
                    while(count % 3 != 1){
                        B.await();
                    }
                    System.out.println("B");
                    count++;
                    C.signal();

                }
            }
            catch (InterruptedException e){
                e.printStackTrace();
            }
            finally{
                lock.unlock();
            }
        }
    }

    static class ThreadC extends Thread{
        @Override
        public void run() {
            try{
                lock.lock();
                for(int i = 0; i < 10; i++) {
                    while(count % 3 != 2){
                        C.await();
                    }
                    System.out.println("C");
                    count++;
                    A.signal();

                }
            }
            catch (InterruptedException e){
                e.printStackTrace();
            }
            finally{
                lock.unlock();
            }
        }
    }

  public static void main(String[] args) {
    //
      new ThreadA().start();
      new ThreadB().start();
      new ThreadC().start();
  }
}
```

