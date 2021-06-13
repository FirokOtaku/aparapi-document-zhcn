# Aparapi - FAQ

> **最常被问到的问题**

**这个项目为什么叫做 Aparapi, 应该怎么念?**

Aparapi 是 **A PAR{allel} API** 的缩写, 念作 *ap-per-rap-ee*.

**Aparapi 仅能在 AMD 显卡上使用吗?**

不. Aparapi 已经在 Windows, Linux 和 Mac OSX 平台上使用 AMD 的 OpenCL 驱动和设备, 以及若干 NVidia 驱动和设备进行了测试. 运行时的最低要求是 OpenCL 1.1. 如果你有一个兼容 OpenCL 1.1的设备就能使用 Aparapi.

目前的构建是为 AMD APP SDK 配置的. 但 OpenCL 是一个开放标准, 我们期待能有更多贡献, 允许 Aparapi 能面向其它 OpenCL SDK 构建.

使用 AMD APP SDK 构建的 dll 可以运行在其它的平台上. 所以我们的二进制构建文件可以运行在所有 OpenCL 1.1 平台上.

Witold Bolt 提供了补丁以便支持 Mac OS. Mac OS 版可以运行在 OpenCL 1.1 和 1.0 上, 但我们不会修复任何 OpenCL 1.0 环境下出现的问题. 你得代码可能能用, 也可能不能.

Aparapi 可以在 Oracle® 的 JDK 支持的所有平台上以 JTP (Java 线程池) 模式使用.


** Aparapi 仅能运行在 AMD CPU 上吗? **

不, Aparapi 不限制使用任何 CPU. 我们使用的 JNI 代码可以在任何 X86/X64 机器上运行, 只要您的平台有一个兼容的 Java Virtual Machine® JVM实现.

**.NET 平台上会有像 Aparapi 这样的翻译器吗?**

目前这仍旧是一项早期技术. Aparapi 目前专注于 Java® 环境. 也有针对 .NET 的类似项目. (参见 www.tidepowerd.com)

**我如何对 Aparapi 生成的 OpenCL 核心进行性能分析？ 我能否得知我的核心请求的延迟? 我如何优化我的核心?**

AMD 提供了 **AMD APP 性能分析器** 可以用于分析. 使用 Aparapi 的情况下, 我们推荐使用此分析器的命令行模式. 使用 AMD APP 性能分析器, 你可以看到每个核心执行花了多少时间, 以及缓冲区交换量. 并且你可以获得每个核心的详细信息, 如内存读写和其它数据.

**我可以使用多个线程同时使用 GPU 运算吗?**

是的. 仅当设备本身是瓶颈时可能会对性能造成影响. 但 OpenCL 和你的 GPU 驱动都被设计为协调运行各种线程.

**我能在 `run` 方法中调用其它方法吗?**

一般来说你只能调用与 `run()` 方法处于同一个类中声明的其它方法. Aparapi 将会追踪相关调用链来确定能否创建 OpenCL. 例如, 如果 Aparapi 遇到 `System.out.println("Hello World")` 这种不在内核类中的方法, 就会拒绝将指定代码创建为 OpenCL.

此规则的例外之一是, 一个内核可以通过它们的 getter 或 setter 来改变数组中的对象状态. 打比方说, 一个内核可能会进行如下操作:

```java
out[i].setValue(in[i].getValue()*5);
```

**Aparapi 支持向量类型吗?**

由于 Java 中没有向量类型, Aparapi 不能直接使用它们. 另外由于 Java 没有运算符重载, 在 Java 中模拟向量类型将会非常复杂, 用起来非常不方便.

**有没有办法能查看生成的 OpenCL?**

是的, 在启动 JVM 时添加 `-Dcom.aparapi.enableShowGeneratedOpenCL=true` 参数即可.

**Aparapi 是否支持与 JOGL 共享缓冲区? 我是否可以使用 JOGAMP/glugen 的特性?**

我们不是只支持渲染相关的运算, 而是追求通用的数据并行计算. 因此我们决定不把 Aparapi 与 JOGL 结合得太紧密.

**在性能上比起手写 OpenCL 有什么差异?**

This depends heavily on the application. Although we can currently show 20x performance improvement on some compute intensive Java applications compared with the same algorithm using a Java Thread Pool a developer who is prepared to handcraft and hand-tune OpenCL and write custom host code in C/C++ is likely to see better performance than Aparapi may achieve.

We understand that some user may use Aparapi as a gateway technology to test their Java code before porting to hand-crafted/tuned OpenCL.

Are you working with Project Lambda for offloading/parallelizing suitable work?

We are following the progress of Project Lambda (currently scheduled for inclusion in Java 8) and would like to be able to leverage Lambda expression format in Aparapi, but none exists now.

**我能在系统中存在多个 GPU 时指定使用某个 GPU 吗?**

我们正在考虑. 当下, Aparapi 仅使用找到的第一块 AMD GPU (或 APU). 如果社区需要这项特性, 请让我们知道.

**我能获取在 JavaOne 或 ADFS 上的演示/示例吗?**

Squares 和 Mandlebrot 示例代码包含于 Aparapi 发布的二进制构建中. 由于需要 JOGL 依赖, N体问题相关源码并不包含在内. 但我们将相关源码发布于开源树 (code.google.com/p/aparapi), 并详细介绍了如何安装 JOGL 组件.

**Mersenne twister 能作为一个随机数函数移植到内核类吗?**

你可以在 Kernel 子类中实现和使用自己的 Mersenne twister.

**Aparapi 使用了 JNI 技术?**

是的, 我们实现了一个小型 JNI 中间层用于处理主机 OpenCL 调用.

**我怎么确定我的代码确实运行在 GPU 上了?**

Java 代码中可以在 `Kernel.execute(n)` 执行后得知.

```java
Kernel kernel = new Kernel(){
   @Override public void run(){
   }
} ;
kernel.execute(1024);
System.out.priintln("Execution mode = "+kernel.getExecutionMode());
```

如果内核运行于 GPU, 上面的代码将输出 `GPU`; 如果运行于 Java 线程池, 将输出 `JTP`.

另外启动 JVM 时附加 `–Dcom.aparapi.enableShowExecutionModes=true` 参数将使 Aparapi 自动在标准输出流中打印所有内核的运行模式.

**为什么 Aparapi 需要在编译我的代码时使用 `-g` 参数?**

Aparapi 从你的 `Kernel.run()` (和可及方法) 中提取创建 OpenCL 所需的大部分信息. 我们使用相关调试信息来重建原始变量名并确定局部变量作用域.

当然, 只有 Kernel 子类 (或用到 Java 对象数组功能) 的代码需要在编译时附加 `-g` 参数.

**为什么 Aparapi 的文档建议使用 Oracle 的 JDK/JRE ? 为什么不能使用其它的 JDK/JVM?**

文档中建议使用 Oracle 的 JDK/JRE 是出于覆盖率的考虑, 这不是必须的. AMD 将主要面向Oracle 的 JVM/JDK 进行测试.

这件事分为两部分.

我们的代码转换引擎在某种程度上是基于 Oracle® 提供的 javac 编译的字节码结构开发的. Aparapi 可能无法处理其它的 javac 提供的一些优化. 比如 Eclipse 就没有使用 Oracle 的 javac, 所以我们对处理 Eclipse 生成的字节码模式有一定的经验.

在运行时, 我们会使用 Oracle® 提供的 rt.jar 中的 `sun.misc.Unsafe`. 这个类非常有用, 因为它可以提供访问类字段在内存中的真实地址或原子操作相关的接口, 这能减少 JNI 调用. 为了避免这种依赖性, 所有对 `sun.misc.Unsafe` 的调用都由 Aparapi 重构的 `UnsafeWrapper` 类来处理.

**我正在使用一种动态语言 (Clojure, Scala, Groovy, Beanshell 等), 我能使用 Aparapi 吗?**

不能.

想读取一个方法的字节码的话, Aparapi 必须要获取原始字节码文件 (.class文件). Aparapi 可以使用类似下面的代码来重载字节码文件数据并解析常量池, 属性, 字段, 方法签名和方法代码.

```java
YourClass.getClassLoader().loadAsResource(YourClass.getName()+".class"))
```

动态语言往往采用某种形式的自定义类加载器来使动态生成的字节码对 JVM 可用, 上面的方法不太可能对这些动态创建的类起作用, 这些类加载器不太可能提供给我们动态生成的字节码. 但是我们鼓励贡献者进行相关的探索和研究. 除了类字节数据, 我们还希望获取到调试信息. 这不是不可能的, 因为动态语言往往允许使用 JDB 兼容的调试器对代码进行调试.

Aparapi 的代码分析器可以识别由 Oracle® 提供的 javac 编译器生成的字节码, 但是不一定能兼容其它动态语言生成的字节码.

综上, 目前 Aparapi 不太可能支持动态语言. 但是如果有人愿意提供帮助, 这将会是出色的贡献. 能看到 Aparapi 支持其它基于 JVM 的动态语言将是极好的.

**为什么看起来 Aparapi 不停地在主机和 GPU 之间不必要地拷贝数据? 能不这样吗?**

Aparapi 确保所需的数据在内核执行之前被转移至 GPU, 并且在后续 Java 代码执行之前将数据转移至内存, 一般来说这也是 Java 用户所希望的. 然而对于一些连续进行多个 `Kernel.execute()` (或是在一些紧密的循环中时) Aparapi 的方式可能不是最优解.

在 **新特性** 页面我们介绍了数个 Aparapi 提供的增强功能, 使开发者可以进行干预, 减少不必要的数据转移.

**我必须把自己的代码重构为基于值类型数组吗? 为什么 Aparapi 不能直接操作 Java 对象?**

Aparapi 读取字节码生成 OpenCL. 一般来说, 是 OpenCL 限制我们使用并行的值类型数组 (OpenCL 确实允许结构类型数据, 但是 Java 和 OpenCL 对相关结构的内存布局不具有可比性), 因此你需要重构代码. 在最初本版的 Aparapi 中我们对简易 Java 对象数组提供了有限支持, 你可以在 **新特性** 页面查看如何使用这项功能.
