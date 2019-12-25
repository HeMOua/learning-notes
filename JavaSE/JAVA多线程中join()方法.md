# JAVA多线程中join()方法

> 结论：t.join()方法只会使主线程(或者说调用t.join()的线程)进入等待池并等待t线程执行完毕后才会被唤醒。并不影响同一时刻处在运行状态的其他线程。

下面则是分析过程。

## 两个线程

### 不使用join

```Java
public class TestJoin {
 
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		ThreadTest t1=new ThreadTest("A");
		ThreadTest t2=new ThreadTest("B");
		t1.start();
		t2.start();
	}
 
 
}
class ThreadTest extends Thread {
	private String name;
	public ThreadTest(String name){
		this.name=name;
	}
	public void run(){
		for(int i=1;i<=5;i++){
				System.out.println(name+"-"+i);
		}		
	}
}
```

运行结果：

```
A-1
B-1
B-2
B-3
A-2
B-4
A-3
B-5
A-4
A-5
```

可以看出A线程和B线程是交替执行的。

### 使用join

而在其中加入join()方法后(后面的代码都略去了ThreadTest类的定义)

```java
public class TestJoin {
 
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		ThreadTest t1=new ThreadTest("A");
		ThreadTest t2=new ThreadTest("B");
		t1.start();
		t1.join();
		t2.start();
	}
}
```

运行结果：

```
A-1
A-2
A-3
A-4
A-5
B-1
B-2
B-3
B-4
B-5
```

显然，使用t1.join()之后，B线程需要等A线程执行完毕之后才能执行。需要注意的是，t1.join()需要等t1.start()执行之后执行才有效果，此外，如果t1.join()放在t2.start()之后的话，仍然会是交替执行。

为了深入理解，我们先看一下join()的源码。

## 源码

```java
/**
     * Waits for this thread to die.
     *
     * <p> An invocation of this method behaves in exactly the same
     * way as the invocation
     *
     * <blockquote>
     * {@linkplain #join(long) join}{@code (0)}
     * </blockquote>
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
public final void join() throws InterruptedException {
    join(0);            //join()等同于join(0)
}
/**
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);           //join(0)等同于wait(0)，即wait无限时间直到被notify
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

可以看出，join()方法的底层是利用wait()方法实现的。可以看出，join方法是一个同步方法，当主线程调用t1.join()方法时，主线程先获得了t1对象的锁，随后进入方法，调用了t1对象的wait()方法，使主线程进入了t1对象的等待池，此时，A线程则还在执行，并且随后的t2.start()还没被执行，因此，B线程也还没开始。等到A线程执行完毕之后，主线程继续执行，走到了t2.start()，B线程才会开始执行。

## 三个线程

### 不使用join

```java
public class TestJoin {
 
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		System.out.println(Thread.currentThread().getName()+" start");
		ThreadTest t1=new ThreadTest("A");
		ThreadTest t2=new ThreadTest("B");
		ThreadTest t3=new ThreadTest("C");
		System.out.println("t1start");
		t1.start();
		System.out.println("t2start");
		t2.start();
		System.out.println("t3start");
		t3.start();
		System.out.println(Thread.currentThread().getName()+" end");
	}
 
}
```

运行结果为

```
main start
t1start
t1end
t2start
t2end
t3start
t3end
A-1
A-2
main end
C-1
C-2
C-3
C-4
C-5
A-3
B-1
B-2
B-3
B-4
B-5
A-4
A-5
```

### 使用join

A、B、C和主线程交替运行。加入join()方法后

```java
package CSDN;
 
public class TestJoin {
 
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		System.out.println(Thread.currentThread().getName()+" start");
		ThreadTest t1=new ThreadTest("A");
		ThreadTest t2=new ThreadTest("B");
		ThreadTest t3=new ThreadTest("C");
		System.out.println("t1start");
		t1.start();
		System.out.println("t1end");
		System.out.println("t2start");
		t2.start();
		System.out.println("t2end");
		t1.join();
		System.out.println("t3start");
		t3.start();
		System.out.println("t3end");
		System.out.println(Thread.currentThread().getName()+" end");
	}
 
}
```

运行结果

```
main start
t1start
t1end
t2start
t2end
A-1
B-1
A-2
A-3
A-4
A-5
B-2
t3start
t3end
B-3
main end
B-4
B-5
C-1
C-2
C-3
C-4
C-5
```

多次实验可以看出，主线程在t1.join()方法处停止，并需要等待A线程执行完毕后才会执行t3.start()，然而，并不影响B线程的执行。因此，可以得出结论，t.join()方法只会使主线程进入等待池并等待t线程执行完毕后才会被唤醒。并不影响同一时刻处在运行状态的其他线程。

PS:join源码中，只会调用wait方法，并没有在结束时调用notify，这是因为线程在die的时候会自动调用自身的notifyAll方法，来释放所有的资源和锁。





原文：[JAVA多线程中join()方法的详细分析](https://blog.csdn.net/u013425438/article/details/80205693)