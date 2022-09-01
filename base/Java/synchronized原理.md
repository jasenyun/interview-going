# synchronized 的使用

synchronized 关键字的作用就是只允许同一时刻只有一个线程操作资源。主要用以解决多个线程同时访问时可能出现的问题。synchronized 关键字用于三个位置：

- 用于修饰代码块中

- 用于修饰实例方法

- 用于修饰静态方法

```java
synchronized(锁对象){}
synchronized void method(){}
synchronized static void method(){}
```

## 修饰代码块

```java
public class Learn1 implements Runnable{
    static Learn1 ll = new Learn1();
    @Override
    public void run() {
        synchronized (this){
            System.out.println("我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "结束");
        }
    }
}
```

synchronized 修饰代码块可以指定加锁对象，可以是this、对象、类加锁。从上面代码块可以看到使用线程时，线程的锁都是同一个，所以每个线程都需要等待其他线程释放锁后才能执行。

`synchronized(this|object)` 表示进入同步代码库前要获得给定对象的锁。`synchronized(类.class)` 表示进入同步代码前要获得 当前 `class` 的锁。

```java
//以下使用对象也是一样，只有释放锁后其他线程才能执行
public class Learn1 implements Runnable{
   Object lock1 = new Object()
   @Override
    public void run() {
        synchronized (lock1){
        ... ...
      }
  }
}
   
// 所有线程需要的锁也是同一把   
public class Learn1 implements Runnable{
   @Override
    public void run() {
        synchronized (Learn1.class){
        ... ...
      }
  }
}

```

## 修饰实例方法

相当于是对当前实例加锁，默认锁是this。

```java
public class Learn1 implements Runnable{
    static Learn1 ll = new Learn1();
    @Override
    public void run() {
       sayHi();
    }

    public synchronized void sayHi(){
        System.out.println("我是线程" + Thread.currentThread().getName());
    }
}
```

## 修饰静态方法

是给当前类加锁，会作用于类的所有对象实例，所以进入同步代码前要获得当前 class 的锁。静态成员知识类的一种成员，不属于任何实例对象。当线程A 调用实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象。

```java
public class Learn1 implements Runnable{
    static Learn1 lockObj1 = new Learn1();
    static Learn1 lockObj2 = new Learn1();

    public static synchronized void sayHi(){
        System.out.println("我是线程" + Thread.currentThread().getName());
    }
    public static void main(String[] args) {
     Thread t1 = new Thread(instence1);
     Thread t2 = new Thread(instence2);
     t1.start();
     t2.start()
   }
}
```

以上的源码执行后的结果可以看出，无论是哪个线程访问它，需要的锁都只有一个。因为用的是类锁。

注意：类锁和实例对象锁（其中 this 就是当前实例对象）是不一样的。修饰代码块时是可以自己指定锁对象的；修饰方法时，静态方法是类锁，非静态方法是对象锁

## 使用 Synchronized 保证线程安全

加锁和释放锁的原理，可重入原理，保证可见性原理。





参考资料：

- https://pdai.tech/md/java/thread/java-thread-x-key-synchronized.html
