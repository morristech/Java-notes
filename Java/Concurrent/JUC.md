# JUC

## 1、原子类AtomicInteger等

### 1.1 AtomicInteger

下面是该类的部分方法和字段，至于其他的原子类与该类基本相同。

    public class AtomicInteger extends Number implements java.io.Serializable {
        private static final long serialVersionUID = 6214790243416807050L;

        private static final Unsafe unsafe = Unsafe.getUnsafe();
        private static final long valueOffset;

        static {
            try {
                valueOffset = unsafe.objectFieldOffset
                    (AtomicInteger.class.getDeclaredField("value"));
            } catch (Exception ex) { throw new Error(ex); }
        }

        private volatile int value;

        public AtomicInteger() {}
        public AtomicInteger(int initialValue) { value = initialValue; }

        public final int get() { return value; }

        public final void set(int newValue) { value = newValue; }

        public final int getAndSet(int newValue) {
            return unsafe.getAndSetInt(this, valueOffset, newValue);
        }
    }

从上面可以看出，除了赋值和读取的操作之外，我们所需要的原子操作getAndSet()等，都是借助Unsafe来实现的。

### 1.2 Unsafe

这里的Unsafe.getAndSetInt()方法的定义是：

    public final int getAndSetInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var4));

        return var5;
    }

这里的compareAndSwapInt是一个本地方法，它的各个参数的含义是：

	/**
	* 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
	* 
	* @param obj 需要更新的对象
	* @param offset obj中整型field的偏移量
	* @param expect 希望field中存在的值
	* @param update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
	* @return 如果field的值被更改返回true */
	public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);

上面的方法叫做CAS操作。CAS操作有3个操作数，内存值M，预期值E，新值U，如果M==E，则将内存值修改为B，否则啥都不做。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。” java.util.concurrent包全完建立在CAS之上，没有CAS也就没有此包。

### 1.3 CAS

如果不使用compareAndSwapInt，代码大致是：

    public int i = 1;
    public boolean compareAndSwapInt(int j) {
        if (i == 1) {
            i = j;
            return true;
        }
        return false;
    }

当然这段代码在并发下是肯定有问题的，有可能线程1运行到了第5行正准备运行第7行，线程2运行了，把i修改为10，线程切换回去，线程1由于先前已经满足第5行的if了，所以导致两个线程同时修改了变量i。

解决办法也很简单，给compareAndSwapInt方法加锁同步就行了，这样，compareAndSwapInt方法就变成了一个原子操作。CAS也是一样的道理，比较、交换也是一组原子操作，不会被外部打断，先根据var1,var2获取到内存当中当前的内存值var5，在将内存值var5和原值expect作比较，要是相等就修改为要修改的值update，由于CAS都是硬件级别的操作，因此效率会高一些。

### 1.4 AtomInteger原理

所以，当我们调用AtomInteger.getAndSet()方法的时候，它通过CAS实现线程安全的方式是：

1. 假设AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程1和线程2各自持有一份value的副本，值为3；
2. 线程1运行到getIntVolatile()方法时获取到当前的value为3，线程切换；
3. 线程2开始运行，获取到value为3，利用CAS对比内存中的值也为3，比较成功，修改内存，此时内存中的value改变比方说是4，线程切换；
4. 线程1恢复运行，利用CAS比较发现自己的value为3，内存中的value为4，得到一个重要的结论-->此时value正在被另外一个线程修改，所以我不能去修改它；
5. 线程1的compareAndSet失败，循环判断，因为value是volatile修饰的，所以它具备可见性的特性，线程2对于value的改变能被线程1看到，只要线程1发现当前获取的value是4，内存中的value也是4，说明线程2对于value的修改已经完毕并且线程1可以尝试去修改它；
6. 最后说一点，比如说此时线程3也准备修改value了，没关系，因为比较-交换是一个原子操作不可被打断，线程3修改了value，线程1进行compareAndSet的时候必然返回的false，这样线程1会继续循环去获取最新的value并进行compareAndSet，直至获取的value和内存中的value一致为止。

整个过程中，利用CAS机制保证了对于value的修改的线程安全性。

所以，理解起来就是，对于每个线程，先使用getIntVolatile()方法读取内存中的值，然后用compareAndSwapInt()方法试着去更新内存中的值。如果我（当前线程）持有的值和内存中的值不一样，那么我就不更新，compareAndSwapInt返回false。 然后，等到下次用getIntVolatile()方法到值的时候，内存中的值已经被更新完毕了。内存中的值和我持有的值一样，所以我可以对其进行更新。

### 1.5 CAS的问题

CAS看起来很美，但这种操作显然无法涵盖并发下的所有场景，并且CAS从语义上来说也不是完美的，存在这样一个逻辑漏洞：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？如果在这段期间它的值曾经被改成了B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个漏洞称为CAS操作的"ABA"问题。java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类"AtomicStampedReference"，它可以通过控制变量值的版本来保证CAS的正确性。不过目前来说这个类比较"鸡肋"，大部分情况下ABA问题并不会影响程序并发的正确性，如果需要解决ABA问题，使用传统的互斥同步可能回避原子类更加高效。

[原文参考：https://www.cnblogs.com/xrq730/p/4976007.html](https://www.cnblogs.com/xrq730/p/4976007.html)


