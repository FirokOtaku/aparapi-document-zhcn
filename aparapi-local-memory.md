# 本地内存

> **如何在内核中使用本地内存**

## 使用本地内存特性

> 译注:  
> 这一整段跟 [常量内存 # 使用常量内存特性](aparapi-constant-memory.md) 一毛一样,  
> 在这边就不重复了, 有需要的重新去看那个吧

## 如何将一个值类型数组定义为 "本地"

我们提供了两种方式来定义本地缓冲区. 一, 我们可以为变量名增加 `_$local$` 后缀 (是的这在 Java 中是合法标识符).

```java
final int[] buffer = new int[1024]; // this is global accessable to all work items.
final int[] buffer_$local$ = new int[1024]; // this is a local buffer 1024 int's shared across all work item's in a group

Kernel k = new Kernel(){
    public void run(){
         // access buffer
         // access buffer_$local$
         localBarrier(); // allows all writes to buffer_$local$ to be synchronized across all work items in this group
         // ....
    }
}
```

二, 我们可以使用 `@Local` 注解 (只能在 `Kernel` 子类中使用, 不能在前面的匿名内部类模式下使用).

```java
final int[] buffer = new int[1024]; // this is global accessable to all work items.

Kernel k = new Kernel(){
    @Local int[] localBuffer = new int[1024]; // this is a local buffer 1024 int's shared across all work item's in a group
    public void run(){
         // access buffer
         // access localBuffer
         localBarrier(); // allows all writes to localBuffer to be synchronized across all work items in this group
         // ....
    }
}
```

## 我怎么才能知道要开一个多大的本地内存?

可以使用 `Range`.

如果我们创建了一个 `Range`:

```java
Range rangeWithUndefinedGroupSize = Range.create(1024);
```

Aparapi 会选择一个合适的组大小. 一般来说这会是最大的全局因子, 最大为 256. 所以对于全局大小是 2 的幂且大于等于 256 的情况下, 组大小将会是 256.

一般来说, 本地缓冲区的大小会是组大小的某个比率.

所以如果我们每组需要 4 个 `int`, 我们可以使用这样的序列:

```java
final int[] buffer = new int[8192]; // this is global accessable to all work items.
final Range range = Range.create(buffer.length); // let the runtime pick the group size

Kernel k = new Kernel(){
    @Local int[] localBuffer = new int[range.getLocalSize(0)*4]; // this is a local buffer containing 4 ints per work item in the group
    public void run(){
         // access buffer
         // access localBuffer
         localBarrier(); // allows all writes to localBuffer to be synchronized across all work items in this group
         // ....
    }
}
```

当然, 你也可以在创建 `Range` 的时候一个组大小.

```java
final int[] buffer = new int[8192]; // this is global accessable to all work items.
final Range range = Range.create(buffer.length,16); // we requested a group size of 16

Kernel k = new Kernel(){
    @Local int[] localBuffer = new int[range.getLocalSize(0)*4]; // this is a local buffer containing 4 ints per work item in the group = 64 ints
    public void run(){
         // access buffer
         // access localBuffer
         localBarrier(); // allows all writes to localBuffer to be synchronized across all work items in this group
         // ....
    }
}
```

## 使用内存屏障

上面我们提到, 本地内存由同一个工作组内的所有内核共享. 但是, 如果要读取一个由其它内核写入的数据, 我们需要插入一个本地屏障.

一种常见的方式是, 让每个内核从全局内存中复制某个数据到本地内存:

```java
Kernel k = new Kernel(){
    @Local int[] localBuffer = new int[range.getLocalSize(0)];
    public void run(){

         localBuffer[getLocalId(0)] = globalBuffer[getGlobalId(0)];
         localBarrier(); // after this all kernels can see the data copied by other workitems in this group
         // use localBuffer[0..getLocalSize(0)]
    }
}
```

如果没有上面的内存屏障, 就不能保证某个内核能看到来自其它内核对 `localBuffer` 的改变.

## 谨慎使用内存屏障

内存屏障包含潜在的危险. 开发者有责任在每个内核的执行过程中调用等量次数的 `localBarrier()`, 特别是存在条件分支或循环的代码时需要更加小心.

下面的内核就会造成死锁:

```java
Kernel kernel = new Kernel(){
    public void run(){
         if (getGlobalId(0)>10){
            // ...
            localBarrier();
            // ...
         }
    }
}
```

需要保证同一组内的内核都调用 `localBarrier()`, 所以应该使用下面的代码才行:

```java
Kernel kernel = new Kernel(){
    public void run(){
         if (getGlobalId(0)>10){
            // ...
            localBarrier();
            // ...
         }else{
            localBarrier();
         }

    }
}
```

当然, 如果我们在 `if` 语句块中调用了多次 `localBarrier()`, 我们也需要在 `if-else` 或 `else` 语句块中调用等量次数 `localBarrier()`:

```java
Kernel kernel = new Kernel(){
    public void run(){
         if (getGlobalId(0)>10){
            // ...
            localBarrier();
            // ...
            localBarrier();
            // ...
         }else{
            localBarrier();
            localBarrier();
         }

    }
}
```

有循环时, 我们必须保证每个内核都执行等量次数的 `localBarrier()`.

下面的代码可以正常运行:

```java
Kernel kernel = new Kernel(){
    public void run(){
         for (int i=0; i< 10; i++){
            // ...
            localBarrier();
            // ...
         }
    }
}
```

下面的代码会产生死锁:

```java
Kernel kernel = new Kernel(){
    public void run(){
         for (int i=0; i< getLocalId(0); i++){
            // ...
            localBarrier();
            // ...
         }
    }
}
```

JTP 模式也会对 OpenCL 模式进行模拟, 类似的问题代码在 JTP 模式下也会产生死锁, 所以请小心!

## JTP 模式下的性能影响

当然, Java 并不支持任意形式的本地内存, 所以任何用到本地内存的代码回退到 JTP 模式时都会遇到性能的显著下降 (请试试在 JTP 模式运行N体问题示例).

我们使用 Java 提供的并行计算工具特性来模拟内存屏障. 但是由于 Java 的内存模型并不要求跨线程监控数组变化, 所以这些屏障基本上只能造成性能损失.

我建议只有在代码明确会运行到 GPU 上时才使用本地内存和内存屏障.

## 我能看看代码吗?

[这里](https://github.com/Syncleus/aparapi-examples/blob/master/src/main/java/com/aparapi/examples/nbody/Local.java) 有一个使用了本地内存特性的N体问题示例源码.
