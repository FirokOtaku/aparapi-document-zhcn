# Aparapi - 关于

> **一个用于将原生 Java 代码运行至 GPU 的框架**

**此框架基于 Apache Software License v2 协议**

Aparapi 可以在运行时动态地将原生 Java 代码转换为 OpenCL 内核. Aparapi 基于 OpenCL 所以可以兼容所有支持 OpenCL 的显卡.

GPU 与 CPU 的架构非常不同. 最明显的区别之一是, 一块典型的 CPU 只有不到十几个核心, 而高端的 GPU 可能有数百个核心. 这使得 GPU 在数据并行计算任务上比一般 CPU 快数百倍. 某些程序不再需要整个数据中心级的算力, 而是一两台电脑就可以完成, 这可能节省下大量硬件投入.

Aparapi 项目最初由 AMD 公司构思和开发, 后来的几年它被 AMD 放弃并一直处理闲置状态. 社区尝试维持这个项目的活力, 但由于没有明确的社区领袖, Aparapi 项目一直停步不前. 长达5年的漫长等待后, Aparapi 终于发布了全新的版本, 社区带着焕然一新的热情继续推动项目的发展.

下面是使用 CPU 和 GPU 处理 **N体问题** 的比较. 你可以在本地运行这个实例项目. 模拟运行在一块廉价显卡上. 很明显, 使用 Aparapi 可以大幅提升模拟性能.

## 捐赠

作为一个开源项目, Aparapi 完全基于捐赠运行. 你可以在这里为我们努力工作的开发者点一杯啤酒. 所有的捐款都将用于为重要的错误和改进提供赏金. 你可以点击上述链接进入捐赠页面.

(链接已损坏)

## 支持和文档

这个项目托管于 [QOTO GitLab](https://git.qoto.org/aparapi/aparapi), 在 [GitHub](https://github.com/Syncleus/aparapi) 也维护了一份最新的镜像源.

Aparapi Javadocs: [latest](http://www.javadoc.io/doc/com.aparapi/aparapi) - [2.0.0](http://www.javadoc.io/doc/com.aparapi/aparapi/2.0.0) - [1.10.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.10.0) - [1.9.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.9.0) - [1.8.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.8.0) - [1.7.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.7.0) - [1.6.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.6.0) - [1.5.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.5.0) - [1.4.1](http://www.javadoc.io/doc/com.aparapi/aparapi/1.4.1) - [1.4.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.4.0) - [1.3.4](http://www.javadoc.io/doc/com.aparapi/aparapi/1.3.4) - [1.3.3](http://www.javadoc.io/doc/com.aparapi/aparapi/1.3.3) - [1.3.2](http://www.javadoc.io/doc/com.aparapi/aparapi/1.3.2) - [1.3.1](http://www.javadoc.io/doc/com.aparapi/aparapi/1.3.1) - [1.3.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.3.0) - [1.2.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.2.0) - [1.1.2](http://www.javadoc.io/doc/com.aparapi/aparapi/1.1.2) - [1.1.1](http://www.javadoc.io/doc/com.aparapi/aparapi/1.1.1) - [1.1.0](http://www.javadoc.io/doc/com.aparapi/aparapi/1.1.0) - [1.0.0](http://www.javadoc.io/doc/com.syncleus.aparapi/aparapi/1.0.0)

详细文档可以参考 [Aparapi.com](http://aparapi.com/) 或查阅最新的 [Javadocs](http://www.javadoc.io/doc/com.aparapi/aparapi).

如果需要支持请使用 [Gitter](https://gitter.im/Syncleus/aparapi) 或 [Aparapi 官方邮件列表和论坛](https://discourse.qoto.org/c/PROJ/APA).

请在 [QOTO GitLab](https://git.qoto.org/aparapi/aparapi/issues) 提交 bug 或特性请求. 旧问题的存档仍旧可以在 [GitHub](https://github.com/Syncleus/aparapi/issues) 找到.

Aparapi 遵循 **[格式化版本号 2.0.0 标准](http://semver.org/spec/v2.0.0.html)** , 这意味着我们的版本号不是任意的, 而是描述库接口如何变化. 你可以在 [这里](http://semver.org/spec/v2.0.0.html) 查看格式化版本号的更多描述.

## 相关项目

这个库只表示核心 Java 库, 还有几个其它的库值得一看.

**Aparapi 示例** - 一组 Java 源码示例展示和帮助开发者使用 Aparapi.
**Aparapi JNI** - 用于在运行时嵌入并加载本地组件, 这样就不用单独安装 Aparapi 本地库.
**Aparapi Native** - C/C++ 编写的本地库组件, 用于提供 Java 库和显卡之间的通信.
**Aparapi Vagrant** - 用于为 X86, X64 和 Linux 编译 Aparapi 本地库的 vagrant 环境.
**Aparapi 网站** - 本网站源码和本项目详细文档.

## 环境需求

Aparapi 可以在 CPU 上正常运行. 但是在 GPU 上运行的话, 您需要在本地系统上安装 OpenCL. 如果没有找到 OpenCL, Aparapi 就会退回到 CPU 模式. Aparapi 支持 OpenCL 1.2, OpenCL 2.0 和 OpenCL 2.1.

Aparapi 可以在所有的操作系统和平台运行, 但是 GPU 加速功能仅可在以下平台使用: Windows 64位, Windows 32位, Mac OSX 64位, Linux 64位和 Linux 32位.

注意: 现在不需要再手动安装 Aparapi JNI 本地接口, Maven 会自动安装.

## Java 依赖

想要在你的项目中使用 Aparapi 你需要声明以下 Maven 依赖.

```xml
<dependency>
    <groupId>com.aparapi</groupId>
    <artifactId>aparapi</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 获取源码

官方源码位于 Syncleus Github 中, 可以使用以下命令克隆.

```bash
git clone https://git.qoto.org/aparapi/aparapi.git
```

## 起步

下面是一个典型的串行循环, 将 `inA` 和 `inB` 中的所有元素依次求和存放到 `result` 中.

```java
final float inA[] = .... // get a float array of data from somewhere
final float inB[] = .... // get a float array of data from somewhere
assert (inA.length == inB.length);
final float result = new float[inA.length];

for (int i = 0; i < array.length; i++) {
    result[i] = inA[i] + inB[i];
}
```

上述代码可以重构为如下形式:

```java
Kernel kernel = new Kernel() {
    @Override
    public void run() {
        int i = getGlobalId();
        result[i] = inA[i] + inB[i];
    }
};

Range range = Range.create(result.length);
kernel.execute(range);
```
