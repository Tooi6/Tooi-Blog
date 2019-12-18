---
title: 线程安全问题 synchronized、ReentrantLock、volatile    
date: 2019-12-18 22:05:30  
tags:  
- 多线程
- synchronized
- ReentrantLock
- volatile
---

### 概念
#### 什么是是进程、线程？  
> **进程：** 进程是可并发执行的程序在某个数据集合上的一次计算活动，也是**操作系统进行资源分配和调度的基本单位。**    
**线程：** 进程想要执行任务就需要依赖线程。换句话说，就是**进程中的最小执行单位就是线程**，并且一个进程中至少有一个线程。  

#### 为什么要有多线程？
> **提高CPU的利用率、提高计算效率**

#### 创建多线程  
- **继承 Thread**

```
class MyThread extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("hello thread！");
    }
}

public class ThreadCreateDemo1 {
    public static void main(String[] args) {
        MyThread thread=new MyThread();
        thread.start();
    }
}
```

- **实现 Runnable 接口**

```
class MyRunnable implements Runnable{

    @Override
    public void run() {
        System.out.println("hello runnable");
    }
}

public class ThreadCreateDemo2 {
    public static void main(String[] args) {
        Runnable runnable=new MyRunnable();
        Thread thread=new Thread(runnable);
        thread.start();
    }
}
```

- **实现 Callable 接口（有返回值）**

```
class MyCallable implements Callable<String>{

    @Override
    public String call() throws Exception {
        System.out.println("hello callable");
        return "result";
    }
}

public class ThreadCreateDemo3 {
    public static void main(String[] args) {
        MyCallable myCallable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(myCallable);
        new Thread(futureTask).start();
    }
}
```

#### 解决线程安全问题
> **解决线程安全的根本方法**就是：**同一时刻有且只有一个线程在操作共享数据**，其他线程必须等到该线程处理完数据后再对共享数据进行操作。  

> 解决线程安全问题有两种方法，一是使用 synchronized 锁代码块/方法区，二是使用 ReentrantLock 加锁  
需要注意的是，synchronized锁的不是代码，而是对象

下面提供一个买票的例子  

- **线程不安全列子**
> 

```
public class ThreadUnSecurity {

    static int tickets = 10;

    class SellTickets implements Runnable {

        @Override
        public void run() {
            // 未加同步时产生脏数据
            while (tickets > 0) {
                System.out.println(Thread.currentThread().getName() + "--->售出第：  " + tickets + " 票");
                tickets--;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            if (tickets <= 0) {
                System.out.println(Thread.currentThread().getName() + "--->售票结束！");
            }
        }
    }

    public static void main(String[] args) {
        SellTickets sell = new ThreadUnSecurity().new SellTickets();
        Thread thread1 = new Thread(sell, "1号窗口");
        Thread thread2 = new Thread(sell, "2号窗口");
        Thread thread3 = new Thread(sell, "3号窗口");
        Thread thread4 = new Thread(sell, "4号窗口");
        Thread thread5 = new Thread(sell, "5号窗口");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
    }
}
```
- **运行结果**  

![image](https://note.youdao.com/yws/api/personal/file/6E1037C2D0944380833E3B55A1E1F534?method=download&shareKey=4311fc0d8262148e18fdbd564cff3072)

> 同一张票被出售多次。  

- **使用 synchronized 同步代码块**  

```
public class ThreadSynchronizedSecurity {
    static int tickets = 5;
    class SellTickets implements Runnable {
        @Override
        public void run() {
            while (tickets > 0) {
                // 加锁
                synchronized (this) {
                    if(tickets<=0) return;
                    System.out.println(Thread.currentThread().getName() + "--->售出第：  " + tickets + " 票");
                    tickets--;
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            if (tickets <= 0) {
                System.out.println(Thread.currentThread().getName() + "--->售票结束！");
            }
        }
    }
}
```

- **运行结果**  

![image](https://note.youdao.com/yws/api/personal/file/A35EC5F0AD9E464488B8999ECBFFA045?method=download&shareKey=a483019add9f48eba5070161a63db429)

- **使用 synchronized 同步方法**  

```
public class ThreadSynchronizedMethodSecurity {

    static int tickets = 5;

    class SellTickets implements Runnable {

        @Override
        public void run() {
            // 同步方法
            while (tickets > 0) {
                synMethod();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            if (tickets <= 0) {
                System.out.println(Thread.currentThread().getName() + "--->售票结束！");
            }
        }
    }
    // 使用synchronized修饰，同步方法
    synchronized void synMethod() {
        synchronized (this){
            if (tickets <=0) {
                return;
            }
            System.out.println(Thread.currentThread().getName()+"---->售出第 "+tickets+" 票 ");
            tickets-- ;
        }
    }
}
```

- **使用Lock加锁**  

```
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ThreadLockSecurity {
    static int tickets = 5;

    class SellTickets implements Runnable {
        Lock lock = new ReentrantLock();

        @Override
        public void run() {
            // 未加同步时产生脏数据
            while (tickets > 0) {
                try {
                    // 加锁 
                    lock.lock();
                    if (tickets <= 0) return;
                    System.out.println(Thread.currentThread().getName() + "--->售出第：  " + tickets + " 票");
                    tickets--;
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    // 解锁
                    lock.unlock();
                }

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            if (tickets <= 0) {
                System.out.println(Thread.currentThread().getName() + "--->售票结束！");
            }
        }
    }
}
```

- **synchronized 和 ReentrantLock 的区别**  
> ①两者都是**可重入锁**（自己可以再次获取自己的内部锁）  
② synchronized 是**关键字**，依赖于 JVM 而 ReentrantLock 是**类**，依赖于 API    
③ ReentrantLock 比 synchronized 增加了一些高级功能：   
1、等待可中断；  
2、可实现公平锁；  
3、可实现选择性通知（锁可以绑定多个条件）

#### volatile 关键字  
> 一个变量声明为volatile，就意味着这个变量是随时会被其他线程修改的，因此**不能将它cache在线程memory中**  

- **volatile 与 synchronized 的区别**  
> synchronized加锁机制**既可以确保可见性，又可以确保原子性**；  
而**Volitile变量只能确保可见性**。  

- **可见性？原子性？**   
> **可见性：** 指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。  
**原子性：** 是指线程的多个操作是一个整体，不能被分割，要么就不执行，要么就全部执行完，中间不能被打断