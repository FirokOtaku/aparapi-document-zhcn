# 使用 HSA 模拟器

> **在 Aparapi Lambda 分支使用 HSA 模拟器**

## 介绍

我们理解开发者有时可能不能使用 HSA 设备. HSA 基金会已经开源了一个基于 LLVM 的 HSAIL 仿真器, 我们可以基于此来测试 HSAIL 生成的代码.

此项目基于 [此](https://github.com/HSAFoundation/Okra-Interface-to-HSAIL-Simulator), 但是我们在下面为 Ubuntu 平台提供了详细的下载和构建说明信息.

Aparapi 用户/开发者可以使用这个模拟器进行测试.

## 在 Ubuntu 平台构建 HSA 模拟器

因为您能构建其它的 Aparapi 子项目, 我们设定您的环境中存在 ant, svn 和 g++.

你还需要用到 git, libelf-dev, libdwarf-dev, flex 和 cmake.

```bash
$ sudo apt-get install git libelf-dev libdwarf-dev flex cmake
```

输入密码…

```bash
$ git clone https://github.com/HSAFoundation/Okra-Interface-to-HSAIL-Simulator.git okra
$ cd okra
$ ant -f build-okra-sim.xml
```

构建过程大概需要 15 分钟.

## 如何配置并测试 Aparapi 的 Lambda/HSA 功能

> 译注:  
> 这块原文的标题和正文都乱套了

假定您已经在 `/home/gfrost/okra` 中构建了 okra.

假定您的 Java8 JDK 在 `/home/gfrost/jdk1.8.0`

假定您的 Aparapi svn 仓库为 `/home/gfrost/aparapi`

```bash
$ export JAVA_HOME=/home/gfrost/jdk1.8.0
$ export OKRA=/home/gfrost/okra
$ export PATH=${PATH}:${JAVA_HOME}/bin:${OKRA}/dist/bin
$ java -version
java version "1.8.0-ea"
Java(TM) SE Runtime Environment (build 1.8.0-ea-b94)
Java HotSpot(TM) 64-Bit Server VM (build 25.0-b36, mixed mode)
$ cd /home/gfrost/aparapi/branches/lambda
$ ant
$ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${OKRA}/dist/bin
$ java -agentpath:com.aparapi.jni/dist/libaparapi_x86_64.so -cp com.aparapi/dist/aparapi.jar:${OKRA}/dist/okra.jar hsailtest.Squares
```
