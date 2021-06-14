# 手动管理缓冲区

> **如何最优化缓冲区数据交换**

Aparapi 设计初衷是避免使 Java 开发者手动进行 OpenCL 主机和 GPU 设备之间数据交换的实现细节. Aparapi 可以分析 `Kernel.run()` 和其可及方法, 在执行代码之前确定需要将哪些数据复制到 GPU, 以及在 GPU 运算完成之后需要将哪些数据复制回主机.

一般来说这种实现方式整洁且高效, Aparapi 会很好地完成给定的任务.

但是, 偶尔也会看到下面这种代码模式:

```java
final int[] hugeArray = new int[HUGE];
Kernel kernel= new Kernel(){
    ... // reads/writes hugeArray
};
for (int loop=0; loop <MAXLOOP; loop++){
    kernel.execute(HUGE);
}
```

这种模式非常常见, 很不幸的是 Aparapi 在处理这种模式时会碰到一个问题.

虽然 Aparapi 可以分析 `Kernel.run()` 方法和其可及方法的字节码, Aparapi 不能看到上层调用者如何使用此接口. 在上面的代码里, Aparapi 不能检测到在 `hugeArray` 在 `for` 循环体中有没有被修改; 出于安全考虑, Aparapi 必须将 `hugeArray` 内的数据在主机和 GPU 之间来回进行复制.

下面我们添加注释来指出在哪些地方会出现不必要的数据交换:

```java
final int[] hugeArray = new int[HUGE];
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray
};
for (int loop=0; loop <MAXLOOP; loop++){
   // copy hugeArray to GPU
   kernel.execute(HUGE);
   // copy hugeArray back from the GPU
}
```

实际上, 我们只需要在循环体开始之前和之后各将 `hugeArray` 复制到 GPU 中一次.

下面我们标注一下数据交换最好应怎样进行:

```java
final int[] hugeArray = new int[HUGE];
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray
};
// Ideally transfer hugeArray to GPU here
for (int loop=0; loop <MAXLOOP; loop++){
   kernel.execute(HUGE);
}
// Ideally transfer hugeArray back from GPU here
```

让我们看一下另一种常见的代码模式:

```java
final int[] hugeArray = new int[HUGE];
final int[] done = new int[]{0};
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray and writes to done[0] when complete
};
done[0]=0;
while (done[0] ==0)){
   kernel.execute(HUGE);
}
```

这种模式在 map-reduce ([map-reduce - Wikipedia][map-reduce-en],[map-reduce - Wikipedia(中文)][map-reduce-zhcn]) 编程模型中的 reduce 阶段很常见. 这本质上是开发者希望内核在满足某些条件时能停止执行, 比如在 *双调排序* ([bitonic-sorter - Wikipedia][bitonic sorter-en]) 和各类金融运算时. 

从上面的代码能看到, 内核从 `hugeArray[]` 读取数据然后把结果存到单元素数组 `done[]` 中.

默认情况下, Aparapi 会在每次 `Kernel.execute(HUGE)` 之前将 `done[]` 和 `hugeArray[]` 传输到 GPU 上.

在下面的代码中用注释来指明传输了哪些缓冲区.

```java
final int[] hugeArray = new int[HUGE];
final int[] done = new int[]{0};
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray and writes to done[0] when complete
};
done[0]=0;
while (done[0] ==0)){
   // Send done[] to GPU
   // Send hugeArray[] to GPU
   kernel.execute(HUGE);
   // Fetch done[] from GPU
   // Fetch hugeArray[] from GPU
}
```

对代码进一步分析表明, `hugeArray[]` 没有被内核内部循环使用, 所以这个数组被不必要地向 GPU 传输了999次, 又被不必要地传输回主机999次. 但是只有2次是有效的: 一次是在循环开始之前将数据传输到 GPU, 另一次是在循环结束之后将数据传输回主机.

`done[]` 在每次迭代时都会被调用 (虽然循环中从来没有对其进行过写入); 它确实需要在每次 `Kernel.execute()` 执行结束后传输回来, 但是1次足矣.

显然, 应该避免不必要的数据传输, 特别是类似于 `hugeArray[]` 这样的大缓冲区数据.

Aparapi 提供了一个功能, 允许开发者对上述情况进行优化, 手动控制数据传输.

要使用这个功能, 开发者首先需要调用 `kernel.setExplicit(true)` 启用手动数据传输, 然后使用 `kernel.put()` 或 `kernel.get()` 在主机和 GPU 之间进行数据传输.

下面的代码演示应如何使用上述接口:

```java
final int[] hugeArray = new int[HUGE];
final int[] done = new int[]{0};
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray and writes to done[0] when complete
};
kernel.setExplicit(true);
done[0]=0;
kernel.put(done);
kernel.put(hugeArray);
while (done[0] ==0)){
   kernel.execute(HUGE);
   kernel.get(done);
}
kernel.get(hugeArray);
```

需要注意的是, 开启一个内核的手动数据传输控制功能却没有进行适当的数据传输是开发者的责任.

我们故意让 `Kernel.put(...)`, `Kernel.get(...)` 和 `Kernel.execute(range)` 方法返回内核实例方便进行链式调用. 链式编程模式代码在某些情况下更具表现力.

```java
final int[] hugeArray = new int[HUGE];
final int[] done = new int[]{0};
Kernel kernel= new Kernel(){
   ... // reads/writes hugeArray and writes to done[0] when complete
};
kernel.setExplicit(true);
done[0]=0;
kernel.put(done).put(hugeArray);    // chained puts
while (done[0] ==0)){
   kernel.execute(HUGE).get(done);  // chained execute and put
}
kernel.get(hugeArray);
```

另外, 对于 `Kernel.execute(range)` 是循环体内的唯一语句且迭代次数是已知的情况, 我们提供了另一种 (理论上更优雅) 方式来优化缓冲区传输.

所以对于下面这种情况:

```java
final int[] hugeArray = new int[HUGE];
Kernel kernel= new Kernel(){
    ... // reads/writes hugeArray
};

for (int pass=0; pass<1000; pass++){
   kernel.execute(HUGE);
}
```

开发者可以将迭代次数作为第二形参调用 `Kernel.execute(range, iterations)` 来要求 Aparapi 进行外层循环, 而不是手动编写循环.

所以像下面这样的代码:

```java
int range = 1024;
int loopCount = 64;
for (int passId = 0; passId < loopCount; passId++){
   kernel.execute(range);
}
```

可以改为下面这样:

```java
int range = 1024;
int loopCount = 64;

kernel.execute(range, loopCount);
```

这不仅让代码更紧凑, 避免了手动管理缓冲区, 还让 Aparapi 对外部循环本身有可见性, 从而最优化数据传输. 现在 Aparapi 只会将缓冲区传输到 GPU 1次, 再传输回1次, 从而提升性能.

有时这种模式的代码需要在外部循环中追踪当前正在进行迭代的迭代次数号, 在以前我们只能用手动缓冲区管理来实现:

```java
int range = 1024;
int loopCount = 64;
final int[] hugeArray = new int[HUGE];
final int[] passId = new int[0];
Kernel kernel = new Kernel(){
   @Override public void run(){
      int id=getGlobalId();
      if (passId[0] == 0){
          // perform some initialization!
      }
      ... // reads/writes hugeArray
   }
};
Kernel.setExplicit(true);
kernel.put(hugeArray);
for (passId[0]=0; passId[0]<loopCount; passId[0]++){

   kernel.put(passId).execute(range);
}
```

当前 Aparapi 提供了 `Kernel.getPassId()` 接口, 允许内核不需要使用手动缓冲区管理就能获取当前是第几次外部循环执行过程.

所以之前的代码可以像下面这样写:

```java
final int[] hugeArray = new int[HUGE];
final int[] pass[] = new int[]{0};
Kernel kernel = new Kernel(){
   @Override public void run(){
      int id = getGlobalId();
      int pass = getPassId();
      if (pass == 0){
          // perform some initialization!
      }
      ... // reads/writes both hugeArray
   }
};

kernel.execute(HUGE, 1000);
```

`Kernel.getPassId()` 的一个常见用途是避免在外部循环中翻转缓冲区.

我们经常会需要使内核将一个缓冲区内的数据转移到另一个缓冲区, 下一次调用时再用另一种形式返回. 现在我们可以用 `passId` 的奇偶性来控制数据传输的方向.

> 译注:  
> 其实就是 `passId` 也能当作一个参数呗,  
> 毕竟一个整型能存32位数据

```java
final int[] arr1 = new int[HUGE];
final int[] arr2 = new int[HUGE];
Kernel kernel = new Kernel(){
   int f(int v){ … }

   @Override public void run(){
      int id = getGlobalId();
      int pass = getPassId();
      if (pass % 2 == 0){
          arr1[id] = f(arr2[id]);
      }else{
          arr2[id] = f(arr1[id]);

      }
   }
};

kernel.execute(HUGE, 1000);
```

[bitonic-sorter-en]: https://en.wikipedia.org/wiki/Bitonic_sorter
[map-reduce-zhcn]: https://zh.wikipedia.org/wiki/MapReduce
[map-reduce-en]: https://en.wikipedia.org/wiki/MapReduce
