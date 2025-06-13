---
title: 有一点厉害的Usafe.class
date: 2024-01-07 02:19:28
tags: tech
---

第一次知道Unsafe概念是在使用ReentrntLock和[CountDownLatch](https://zhida.zhihu.com/search?content_id=241461290&content_type=Article&match_order=1&q=CountDownLatch&zhida_source=entity)。这两个都实现了[AQS](https://zhida.zhihu.com/search?content_id=241461290&content_type=Article&match_order=1&q=AQS&zhida_source=entity)，AQS能够做到线程安全同步，关键在于AQS中通过查询state变量的值来知道当前资源有没有被锁，改变state变量的值来表示当前资源被锁，改变state是要保证[原子性](https://zhida.zhihu.com/search?content_id=241461290&content_type=Article&match_order=1&q=原子性&zhida_source=entity)的，原子性保障由Unsafe中的CompareAntSwap支持。

从ReentrantLock的非公平锁开始看：

```java
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

lock方法主要是在compareAndSetState(0,1)成功后再让本线程进来执行任务。

```java
protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

```java
// stateOffset是state的偏移量。 
stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
```

所以，ReentrantLock在上锁时一直在维护一个state变量, 想获得锁必须做compareAndSwap比较与交换的操作。

这个"比较与交换"是用Unsafe实现的：

```java
   protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }

    // 比较与交换的native方法 保证比较与交换的原子性
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

还想说一下，什么是原子性，compareAndSwap比较与交换有多个指令组成，这些指令中途不能被打断，整个一起执行完，这就是原子性。

上面是结合ReentrantLock的场景。



Unsafe类非常有意思:

1. 它是根据变量在对象中的偏移量来改变值的，非常底层，非常c语言指针的感觉，但由保证了原子性。
2. 构造方法是private, 程序员不能new过来用, 只能由最底层的system级别的类加载器加载的类才可以通过getUnsafe()方法获取用。(加载器由三种：引导类加载器，ExtClassLoader，AppClassLoader。除了jdk自带的那些底层类，其他由开发者编写的类都被AppClassLoader加载，所以想调用getUnsafe获取unsafe做操作，那你必须是jdk自带的那些底层类) 以下是关键代码:

```java
private Unsafe() {
    }

    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class var0 = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }
```

3. unsafe意味着危险，因为它操作内存的权限比较大，给底层开发jdk的人去用，实现这些concurrent里面的那些原子操作。

当然我们也可以拿来做做实验体会一下，因为还可以通过反射机制来获取unsafe使用：

demo:

```java
/**
 * unsafe是一个强大的类，但是危险 他像c语言一样能访问内存
 * 但是构造方法是private，不给new的，他是一个饿汉式单例模式，static静态代码块里把自己初始化好了
 * Unsafe.getUnsafe()也不能拿到，调用这个方法的雷必须是有引导类加载器加载才能用这个方法拿到unsafe对象
 * 想用可以通过反射机制获取。
 */
public class UnsafeFactory {

    //获取unsafe对象
    public static Unsafe getUnsafe() throws NoSuchFieldException, IllegalAccessException {

        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        return  (Unsafe)field.get(null);
    }

    //获取变量的内存偏移量
    public static long getFieldOffset(Unsafe unsafe, Field field) {
        long offset = unsafe.objectFieldOffset(field);
        return offset;
    }
  
  public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {

        Unsafe unsafe = getUnsafe();

        Field num = ThreadPlusPlusProblem.class.getDeclaredField("num");


        long fieldOffset = getFieldOffset(unsafe, num);

        System.out.println("num在ThreadPlusPlusProblem中的偏移量:"+fieldOffset);

        //put置
        ThreadPlusPlusProblem threadPlusPlusProblem = new ThreadPlusPlusProblem();
        num.setAccessible(true);
        num.set(threadPlusPlusProblem,1);
        System.out.println(JSON.toJSONString(threadPlusPlusProblem));

    }
}

```

建议从ReentrantLock.lock()点进去看一下怎么通过unsafe来原子性的维护state变量来实现锁状态的查询和改变，一定要读AQS这里的源码，想一下AQS中的state用volatile修饰，这里的用意是很底层和比较cool。

希望你是非常喜欢自己的工作, 从中能找到意义和成就^^

