# OpenCL 绑定

> **如何使用新的 OpenCL 绑定功能**

现在向扩展机制迈出一步, 我们需要一种方式将 OpenCL 绑定到一个接口上.

下面我们将用 "正方形" 演示.

首先需要定义一个带有 OpenCL 注解的接口:

```java
interface Squarer extends OpenCL<Squarer>{
@Kernel("{\n"//
     + "  const size_t id = get_global_id(0);\n"//
     + "  out[id] = in[id]*in[id];\n"//
     + "}\n")//
public Squarer square(//
     Range _range,//
     @GlobalReadOnly("in") float[] in,//
     @GlobalWriteOnly("out") float[] out);
}
```

这描述了一个希望绑定到一组内核上的 API (示例中只绑定了一个, 实际上可以绑定任意个). 然后我们需要命令一个设备 "实现" 这个接口. `Device` 是 Aparapi 提供的新类, 用来描述一块 GPU 或 CPU OpenCL 设备. 下面我们要求默认情况下找到的第一个设备实现刚才的接口:

```java
Squarer squarer = Device.firstGPU(Squarer.class);
```

现在你可以直接由一个 `Range` 示例调用刚才的接口:

```java
squarer.square(Range.create(in.length), in, out);
```

我认为我们将有最简单的 OpenCL 绑定.

一些 [在线讨论和建议](http://a-hackers-craic.blogspot.com/2012/03/aparapi.html) 希望我们提供直接从文件/URL读取 OpenCL 源码的功能.

所以我们提供了下面的接口:

```java
@OpenCL.Resource("squarer.cl");
interface Squarer extends OpenCL<Squarer>{
     public Squarer square(//
       Range _range,//
       @GlobalReadOnly("in") float[] in,//
       @GlobalWriteOnly("out") float[] out);
}
```
或者也可以在编译的时候就把本文放进字面值常量里:

```java
@OpenCL.Source("... opencl text here");
interface Squarer extends OpenCL<Squarer>{
     public Squarer square(//
       Range _range,//
       @GlobalReadOnly("in") float[] in,//
       @GlobalWriteOnly("out") float[] out);
}
```

最后, 为了允许动态创建 OpenCL (方便计算各种半径的 FFT):

```java
String openclSource = ...;
Squarer squarer = Device.firstGPU(Squarer.class, openclSource);
```
