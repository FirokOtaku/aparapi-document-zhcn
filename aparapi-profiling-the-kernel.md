# 对内核进行性能分析

> **使用 Aparapi 内建性能分析 API**

如果你想在运行时提取 OpenCL 性能信息, 需要设定启动参数:

```bash
-Dcom.aparapi.enableProfiling=true
```

你可以在 `kernel.execute(range)` 调用成功之后调用 `kernel.getProfileInfo()` 来提取一个 `List List`.

> 译注:  
> 这个地方原文是  
> *Your application can then call kernel.getProfileInfo() after a successful call to kernel.execute(range) to extract a List List.*
> 疑似原文错误


每一个 `ProfileInfo` 都包含了缓冲区读写和执行的耗时信息.

下面的代码会将性能分析信息输出为一个简单的表格:

```java
List<ProfileInfo> profileInfo = k.getProfileInfo();
for (final ProfileInfo p : profileInfo) {
   System.out.print(" " + p.getType() + " " + p.getLabel() + " " + (p.getStart() / 1000) + " .. "
       + (p.getEnd() / 1000) + " " + ((p.getEnd() - p.getStart()) / 1000) + "us");
   System.out.println();
}
```

下面是一个简单的实现:

```java
final float result[] = new float[2048*2048];
Kernel k = new Kernel(){
   public void run(){
      final int gid=getGlobalId();
      result[gid] =0f;
   }
};
k.execute(result.length);
List<ProfileInfo> profileInfo = k.getProfileInfo();

for (final ProfileInfo p : profileInfo) {
   System.out.print(" " + p.getType() + " " + p.getLabel() + " " + (p.getStart() / 1000) + " .. "
      + (p.getEnd() / 1000) + " " + ((p.getEnd() - p.getStart()) / 1000) + "us");
   System.out.println();
}
k.dispose();
```

下面是表格式输出信息:

```bash
java
   -Djava.library.path=${APARAPI_HOME}
   -Dcom.aparapi.enableProfiling=true
   -cp ${APARAPI_HOME}:.
   MyClass

W val$result 69500 .. 72694 3194us
X exec()     72694 .. 72835  141us
R val$result 75327 .. 78225 2898us
```

这个表格表示: 从 `result` 缓冲区中向 GPU 设备转移数据 (`W`) 消耗了 3194 μs; 内核运算过程 (`X`) 消耗了 141 μs; 从设备转移回数据 (`R`)使用了 2898 μs.
