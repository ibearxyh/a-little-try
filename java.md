[TOC]

# volatile

volatile是Java虚拟机提供的一种**轻量级**的同步机制

- 保证可见性
- **不保证原子性**
- 禁止指令重排

# JMM

java内存模型

主内存与工作内存

JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个**工作内存**，而Java内存模型中规定所有变量都存储在**主内存**。

- 可见性
- 原子性 （Atomic）
- 有序性

  **指令重排**

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量无法保证一致性

源代码 --->   **编译器优化重排**  ------>  **指令并行重排**  ------>  **内存系统的重排**  ------>  最终执行的指令

```java
public void mySort() {
    int x = 11;
    int y = 12;
    int x = x + 5;
    int y = x * x;
}
//1234
//1324
//2134
```

```java
public class ResortSeqDemo {
    int a = 0;
    boolean flag = false;
    
    public void method01() {
        a = 1;			//语句1
        flag = true;	//语句2
    }
    public void method02() {
        if (flag) {
            a = a + 5;	//语句3
            System.out.println("value: " + a);
        }
    }
    //由于指令重排，语句2可能比语句1先执行
    //导致执行语句3时a的值为0
    //所以结果值可能为5或6
}
```

**内存屏障**

cpu指令，保证特定操作的执行顺序，保证某些变量的内存可见性

DCL（Double Check Lock）实现单例模式的隐患

```java
instance = new SingletonDemo();
// memory = allocate(); 1、分配对象内存空间
// instance(memory); 	2、初始化对象
// instance = memory;   3、设置instance指向刚分配的内存地址
// 由于指令重排可能步骤3比步骤2先执行，导致instance为null
```

# CAS

Compare-And-Swap，比较并交换，它是一条CPU并发原语，JVM帮我们实现出CAS汇编指令。

```java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atoInt = new AtomicInteger(5);
        atoInt.compareAndSet(5, 2019);  						//true
        System.out.println("current data: " + atoInt.get());	//2019
        atoInt.compareAndSet(5, 2020);  						//false
        System.out.println("current data: " + atoInt.get());	//2019
    }
}
```

**自旋锁和Unsafe类**

rt.jar包下sum.mise包中，其内部方法操作可以像c指针一样直接操作内存

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    //变量值在内存中的偏移地址，Unsafe类根据该地址获取数据
    private static final long valueOffset;  
    private vaolatile int value;
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Excepiton ex) {
            throw new Error(ex);
        }
    }
    
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
}
```

```java
//unsafe.getAndAddInt
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);  //根据对象和内存地址获取当前值
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4)); 
    return var5;
}
//Unsafe类中的compareAndSwapInt，是一个native方法，该方法位于unsafe.cpp中
```

CAS的缺点：

- 循环时间长，开销大
- 只能保证一个共享变量的原子操作
- ABA问题