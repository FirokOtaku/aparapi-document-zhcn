# Aparapi 文档翻译

## 声明

* 本项目是对 [![](logo-aparapi.png) Aparapi](https://aparapi.com/) 项目官方文档的 **自用非官方** 简中翻译, 不保证翻译质量
* 本项目使用 [![](logo-deepl.png) DeepL](https://www.deepl.com/translator) 辅助翻译
* 本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可  
  [ ![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png) ](http://creativecommons.org/licenses/by-nc-sa/4.0/)  
  如果您需要在您的项目中使用本项目内容, 请遵循上述协议

> [知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议 (中文)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)  

* **不允许** 对本作品的全文或部分 **直接复制粘贴再发布**;  
  如需再发布本作品请进行 **有意义的** 增删改.  
  本作品很垃圾, 也请不要让垃圾一点不变味就到处传播

> 中文开发社区已经很拉了, 求求别让它更拉了  
> 每次想找点资料都跟进了废品回收站一样

## 文档列表

翻译状态|说明
-|-
🕒|未开始
▶|进行中
⏯|暂停
✅|完成

原文|译文|翻译状态
-|-|-
**Introduction**|**介绍**
[About](https://aparapi.com/introduction/about.html)|[关于](aparapi-about.md)|✅
[Getting Started](https://aparapi.com/introduction/getting-started.html)|[起步](aparapi-getting-started.md)|✅
[FAQ](https://aparapi.com/introduction/faq.html)|[FAQ](aparapi-faq.md)|✅
**Documentation**|**文档**
[Aparapi Patterns](https://aparapi.com/documentation/aparapi-patterns.html)|[Aparapi 示例](aparapi-patterns.md)|✅
[Choosing Specific Devices](https://aparapi.com/documentation/choosing-specific-devices.html)|[指定运行设备](aparapi-choosing-specific-devices.md)|✅
[Converting Java to OpenCL](https://aparapi.com/documentation/converting-java-to-opencl.html)|[Java 和 OpenCL 之间的转换](aparapi-converting-java-to-opencl.md)|⏯
[Emulating Multiple Entrypoints](https://aparapi.com/documentation/emulating-multiple-entrypoints.html)|[模拟多程序入口](aparapi-emulating-multiple-entrypoints.md)|✅
[Explicit Buffer Handling](https://aparapi.com/documentation/explicit-buffer-handling.html)|[手动管理缓冲区](aparapi-explicit-buffer-handling.md)|✅
[HSA Enabled Lambda](https://aparapi.com/documentation/hsa-enabled-lambda.html)||🕒
[Kernel Guidelines](https://aparapi.com/documentation/kernel-guidelines.html)|[内核编程指南](aparapi-kernel-guidelines.md)|✅
[Library Agent Duality](https://aparapi.com/documentation/library-agent-duality.html)|[库/代理二重性](aparapi-library-agent-duality.md)|⏯
[New Features](https://aparapi.com/documentation/new-features.html)|新特性|🕒
[OpenCL Bindings](https://aparapi.com/documentation/opencl-bindings.html)|[OpenCL 绑定](aparapi-opencl-bindings.md)|✅
[Private Memory Space](https://aparapi.com/documentation/private-memory-space.html)|[私有缓冲区](aparapi-private-memory-space.md)|✅
[Profiling the Kernel](https://aparapi.com/documentation/profiling-the-kernel.html)|[对内核进行性能分析](aparapi-profiling-the-kernel.md)|✅
[Setting Up HSA](https://aparapi.com/documentation/setting-up-hsa.html)||🕒
[Unit Tests](https://aparapi.com/documentation/unit-tests.html)|[单元测试](aparapi-unit-tests.md)|⏯
[Using HSA Simulator](https://aparapi.com/documentation/using-hsa-simulator.html)|[使用 HSA 模拟器](aparapi-using-hsa-simulator.md)|✅
[Constant Memory](https://aparapi.com/documentation/constant-memory.html)||🕒
[Local Memory](https://aparapi.com/documentation/local-memory.html)||🕒
[Multiple Dim Ranges](https://aparapi.com/documentation/multiple-dim-ranges.html)|多维执行域|🕒
**Proposals**|**建议**
[Multiple Dim ND Range](https://aparapi.com/proposals/multiple-dim-nd-range.html)|多维执行域|🕒
[Lambdas](https://aparapi.com/proposals/lambdas.html)||🕒
[Address Space with Buffers](https://aparapi.com/proposals/address-space-with-buffers.html)||🕒
[Extensions](https://aparapi.com/proposals/extensions.html)||🕒
[Device](https://aparapi.com/proposals/device.html)|[选择执行设备](aparapi-device.md)|✅
[Multiple Entry Points](https://aparapi.com/proposals/multiple-entry-points.html)|[多程序入口](aparapi-multiple-entry-points.md)|▶
[Lambda Syntax](https://aparapi.com/proposals/lambda-syntax.html)|[使用 Lambda 语法](aparapi-lambda-syntax.md)|✅

## 译例

### kernel - 内核

Aparapi 的代码都是围绕着 kernel 实例运行的.

暂时不确定有什么更好的译名.

### range - 执行域

一层循环就是一维执行域, 二层循环就是二维执行域, 以此类推.

后面有个 "multiple-dim range" 其实就是多层循环.

> 为什么是 "执行域"?  
> 不知道, 脑内词汇随机排列组合抖机灵

### method and its reachable method - 方法和其可及方法

对于下面的代码, 对 `funA()` 来说其可及方法是 `funB()` 和 `funC()`
```java
void funA()
{
    funB();
    funC();
}
void funB() {}
void funC() {}
void funD() {}
```

### entrypoint - 程序入口

```java
class Human
{
    void goA() { println("go a"); }
    void goB() { println("go b"); }
    void goC() { println("go c"); }
}
```

上面的 Human 实例对外提供3个控制入口执行不同的过程.

Aparapi 对外提供的是下面这种形式的单程序入口, 用参数来控制内部执行过程:

```java
class Human 
{
    void goWhere(int where)
    {
        switch(where)
        {
            case 1: println("go a"); break;
            case 2: println("go b"); break;
            case 3: println("go c"); break;
        }
        println("go "+where);
    }
}
```

详见 [模拟多程序入口](aparapi-emulating-multiple-entrypoints.md).

### reduction function and reduction operation - 减项函数和减项操作

reduction function 作为 reduction operator, 用于执行 reduction operation.

> [Wikipedia](https://en.wikipedia.org/wiki/Reduction_Operator) 对 `Reduction Operator` 的描述:  
> In computer science, the reduction operator is a type of operator that is commonly used in parallel programming to reduce the elements of an array into a single result.  
> Reduction operators are associative and often (but not necessarily) commutative.  
> The reduction of sets of elements is an integral part of programming models such as Map Reduce, where a reduction operator is applied (mapped) to all elements before they are reduced.  
> Other parallel algorithms use reduction operators as primary operations to solve more complex problems.  
> Many reduction operators can be used for broadcasting to distribute data to all processors.

`reduction-operation` 在 Java 中相关的接口为 `java.util.stream.Stream#reduce`.

### commutative style function - 可交换式函数

> [Wikipedia](https://zh.wikipedia.org/wiki/%E4%BA%A4%E6%8F%9B%E5%BE%8B#.E6.95.B8.E5.AD.B8.E5.AE.9A.E7.BE.A9) 对于 "可交换式运算" 的描述:  
> 1. 在集合 `S` 的一二元运算  `*` 被称之为 "可交换" 的, 若:  
>   `∀x,y∈S,x∗y=y∗x`
>   一个不满足上述性质的运算则称之为 "不可交换的".
> 2. 若称 `x` 在 `*` 下和 `y` "可交换", 即表示:  
>   `x*y=y*x`
> 3. 一二元函数 `f:A×A→B` 被称之为 "可交换" 的, 若:  
>   `∀x,y∈A,f(x,y)=f(y,x).`

对若干元素进行一个操作, 给定元素顺序不同不会影响最终结果, 这个操作就是可交换的.

比如求一堆数字里面的最大值, 无论这些数字/对数字求最大值的顺序如何, 都不会影响最终结果.

