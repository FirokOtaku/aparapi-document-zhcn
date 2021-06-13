# 显式处理缓冲区

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

This is a common pattern in reduce stages of map-reduce type problems. Essentially the developer wants to keep executing a kernel until some condition is met. For example, this may be seen in bitonic sort implementations and various financial applications.

From the code it can be seen that the kernel reads and writes hugeArray[] array and uses the single item done[] array to indicate some form of convergence or completion.

As we demonstrated above, by default Aparapi will transfer done[] and hugeArray[] to and from the GPU device each time Kernel.execute(HUGE) is executed.

To demonstrate which buffers are being transfered, these copies are shown as comments in the following version of the code.

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

Further analysis of the code reveals that hugeArray[] is not accessed by the loop containing the kernel execution, so Aparapi is performing 999 unnecessary transfers to the device and 999 unnecessary transfers back. Only two transfers of hugeArray[] are needed; one to move the initial data to the GPU and one to move it back after the loop terminates.

The done[] array is accessed during each iteration (although never written to within the loop), so it does need to be transferred back for each return from Kernel.execute(), however, it only needs to be sent once.

Clearly it is better to avoid unnecessary transfers, especially of large buffers like hugeArray[].

Aparapi exposes a feature which allows the developer to control these situations and explicitly manage transfers.

To use this feature first the developer needs to ‘turn on’ explicit mode, using the kernel.setExplicit(true) method. Then the developer can request buffer/array transfers using either kernel.put() or kernel.get(). Kernel.put() forces a transfer to the GPU device and Kernel.get() transfers data back.

The following code illustrates the use of these new explicit buffer management APIs.

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

Note that marking a kernel as explicit and failing to request the appropriate transfer is a programmer error.

We deliberately made Kernel.put(...), Kernel.get(...) and Kernel.execute(range) return an instance of the executing kernel to allow these calls be chained. Some may find this fluent style API more expressive.

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

An alternate approach for loops containing a single kernel.execute(range) call. One variant of code which would normally suggest the use of Explicit Buffer Management can be handled differently. For cases where Kernel.execute(range) is the sole statement inside a loop and where the iteration count is known prior to the first iteration we offer an alternate (hopefully more elegant) way of minimizing buffer transfers.

So for cases like:-

```java
final int[] hugeArray = new int[HUGE];
Kernel kernel= new Kernel(){
    ... // reads/writes hugeArray
};

for (int pass=0; pass<1000; pass++){
   kernel.execute(HUGE);
}
```

The developer can request that Aparapi perform the outer loop rather than coding the loop. This is achieved explicitly by passing the iteration count as the second argument to Kernel.execute(range, iterations).

Now any form of code that looks like :-

```java
int range = 1024;
int loopCount = 64;
for (int passId = 0; passId < loopCount; passId++){
   kernel.execute(range);
}
```

Can be replaced with

```java
int range = 1024;
int loopCount = 64;

kernel.execute(range, loopCount);
```

Not only does this make the code more compact and avoids the use of explicit buffer management APIs, it allows Aparapi visibility to the complete loop so that Aparapi can minimize the number of transfers. Aparapi will only transfer buffers to the GPU once and transfer them back once, resulting in improved performance.

Sometimes kernel code using this loop-pattern needs to track the current iteration number as the code passed through the outer loop. Previously we would be forced to use explicit buffer management to allow the kernel to do this.

The code for this would have looked something like

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

In the current version of Aparapi we added Kernel.getPassId() to allow a Kernel to determine the current ‘pass’ through the outer loop without having to use explicit buffer management.

So the previous code can now be written without any explicit buffer management APIs:-

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

One common use for Kernel.getPassId() is to avoid flipping buffers in the outer loop.

It is common for kernels to process data from one buffer to another, and in the next invocation process the data back the other way. Now these kernels can use the passId (odd or even) to determine the direction of data transfer.

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
