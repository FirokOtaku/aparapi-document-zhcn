# 常量内存

> **如何在内存中使用常量内存**

# 使用常量内存特性

默认情况下, Aparapi 内核访问到的所有值类型数组都被当作全局内存.如果我们使用 `-Dcom.aparapi.enableShowGeneratedOpenCL=true` 查看生成的代码, 我们会发现值类型数组 (比如 `int buf[]`) 在 OpenCL 中被映射为 `__global` 指针 (如 `__global int *buf`).

虽然这让 Aparapi 更易使用 (特别是对于不熟悉分层内存层次的 Java 开发者来说), 它确实限制了 "强大开发者" 使用 Aparapi 从 GPU 获得更多性能.

来自 AMD 的 [这个页面](http://www.amd.com/us/products/technologies/stream-technology/opencl/pages/opencl-intro.aspx?cmpid=cp_article_2_2010) 介绍了 OpenCL 开发者可以使用哪些不同类型的内存.

在 Aparapi 中的全局内存缓冲区 (Java 值类型数组) 被储存到主机内存和 GPU 全局内存中 (GPU 的显存里).

本地内存距离运算设备更近, 不会 (在用到时才) 被从主机内存复制. 在 OpenCL 中使用本地内存可以大大提升代码性能, 因为从本地内存获取数据的成本要低得多.

本地内存由所有相同组内的内核实例共享. 这就是为什么本地内存的被推迟使用, 直到我们实现了一套合适的机制来指定所需的组容量.

我们最近也为只需要写入 GPU 设备一次而再不会更改的常量内存提供了支持.

Aparapi 仅支持定长数组.

# 如何将一个值类型数组定义为 "常量"

我们提供了两种方式来定义常量缓冲区. 一, 我们可以为变量名增加 `_$constant$` 后缀 (是的这在 Java 中是合法标识符).

```java
final int[] buffer = new int[1024]; // this is global accessable to all work items.
final int[] buffer_$constant$ = new int[]{1,2,3,4,5,6,7,8,9} // this is a constant buffer

Kernel k = new Kernel(){
    public void run(){
         // access buffer
         // access buffer_$constant$
         // ....
    }
}
```

二, 我们可以使用 `@Constant` 注解 (只能在 `Kernel` 子类中使用, 不能在前面的匿名内部类模式下使用).

```java
final int[] buffer = new int[1024]; // this is global accessable to all work items.

Kernel k = new Kernel(){
    @Constant int[] constantBuffer = new int[]{1,2,3,4,5,6,7,8,9} // this is a constant buffer
    public void run(){
         // access buffer
         // access constantBuffers
         // ....
    }
}
```

## 我能看看代码吗?

我提交了*曼德博*示例, RGB 托盘上的值就代表常量内存. 代码可以在 [这里](http://code.google.com/p/aparapi/source/browse/trunk/samples/mandel/src/com/amd/aparapi/sample/mandel/Main.java) 找到, 请看第 95 行. 顺便一说在这个例子里大概提升了 5-7% 的性能.

> 译注:  
> Mandelbrot - 曼德博  
> 没仔细查, 应该是生成分形图形的例子
