# 库/代理二重性

> **Aparapi 库可以加载为 JVMTI 代理**

## What are all these check-ins referring to JVMTI agents?

如果你有留意过 Aparapi 的 SVN 记录, 应该会注意到 JNI 部分代码的改变. 现在 Aparapi 库 (.dll文件等) 可以被作为 JVMTI 加载. 用传统方式启动 (假设库在 `${APARAPI_DIR}`):

```bash
java –Djava.library.path=${APARAPI_DIR} –classpath ${APARAPI_DIR}/aparapi.jar;my.jar mypackage.MyClass
```

或:

```bash
java –agentpath=${APARAPI_DIR}/aparapi_x86_64.dll –classpath ${APARAPI_DIR}/aparapi.jar;my.jar mypackage.MyClass
```

这样就可以将 dll 文件同时作为库和 JVMTI 代理使用.

## 我什么时候需要代理?

Prevously Aparapi loaded classes that it needed to convert to OpenCL using java.lang.Class.getResourceAsStream(). This only works if we have a jar, or if the classes are on the filesystem somewhere. This approach will not work for ‘synthetically generated classes’.

There are applications/frameworks which create synthetic classes (at runtime) which would not normally be useable by Aparapi.

Specifically (and significantly) Java 8 uses synthetic classes to capture args (closure captures) so they can be passed to the final lambda implementation. We needed a way to allow Aparapi to access bytecode of any class, not just those in jars or on the disk.

A JVMTI agent can register an interest in loaded classes (loaded by the classloader)do this. So when we use the aparapi library in ‘agent mode’ it caches all bytes for all loaded classes (yes we could filter by name) and puts this information in a common data structure (should be a map but is a linked list at present).

By adding a new OpenCLJNI.getBytes(String) JNI method, Aparapi can now retrieve the bytes for any loaded classes, out of this cache.

So this combined with our ability to parse classes which don’t have line number information should really enable Aparapi to be used with Scala/JRuby/Groovy or other dynamic scripting languages which create classes on the fly.
