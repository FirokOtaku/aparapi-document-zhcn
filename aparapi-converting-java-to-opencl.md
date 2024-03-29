# Java 和 OpenCL 之间的转换

> **Aparapi 如何将字节码转换为 OpenCL**

## 介绍

这个页面是对最初由 AMD 编写的 《[详细介绍如何将字节码转换为 OpenCL (PDF)](https://aparapi.com/documentation/ByteCode2OpenCL.pdf)》 的总结.

Aparapi 的特有功能之一是可以将 Java 字节码自动转换为 OpenCL.

这个页面会介绍相关操作是如何进行的. 如果你对字节码还不熟悉, 请阅读 **什么是字节码**.

下面的指令

```bash
javac Source.java
```

会将 Java 源码文件编译成字节码文件.

在网上可以找到关于字节码文件的格式的详细文档和说明, 在这里就不详细介绍了. Aparapi 会提取 `Kernel.run()` 和其任何可及方法的字节码.

让我们从一个简单的内核开始说起.

```java
import com.aparapi.Kernel;

public class Squarer extends Kernel{
   int[] in;
   int[] out;
   @Override public void run(){
      int gid = getGlobalId(0);
      out[gid] = in[gid] * in[gid];
   }
}
```

我们会这样编译上面的代码:

```bash
javac -g -cp path/to/aparapi/aparapi.jar Squarer.java
```

然后我们会使用 javap 来检查生成的字节码:

```bash
javap -c -classpath path/to/aparapi/aparapi.jar;. Squarer
```

得到如下结果:

```java
public class Squarer extends com.aparapi.Kernel
  SourceFile: "Squarer.java"
  minor version: 0
  major version: 50
  Constant pool:
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V
const #2 = Method       #5.#18; //  Squarer.getGlobalId:(I)I
const #3 = Field        #5.#19; //  Squarer.out:[I
const #4 = Field        #5.#20; //  Squarer.in:[I
const #5 = class        #21;    //  Squarer
const #6 = class        #22;    //  com/amd/aparapi/Kernel
const #7 = Asciz        in;
const #8 = Asciz        [I;
const #9 = Asciz        out;
const #10 = Asciz       <init>;
const #11 = Asciz       ()V;
const #12 = Asciz       Code;
const #13 = Asciz       LineNumberTable;
const #14 = Asciz       run;
const #15 = Asciz       SourceFile;
const #16 = Asciz       Squarer.java;
const #17 = NameAndType #10:#11;//  "<init>":()V
const #18 = NameAndType #23:#24;//  getGlobalId:(I)I
const #19 = NameAndType #9:#8;//  out:[I
const #20 = NameAndType #7:#8;//  in:[I
const #21 = Asciz       Squarer;
const #22 = Asciz       com/amd/aparapi/Kernel;
const #23 = Asciz       getGlobalId;
const #24 = Asciz       (I)I;

{
int[] in;

int[] out;

public Squarer();
  Code:
   Stack=1, Locals=1, Args_size=1
   0:   aload_0
   1:   invokespecial   #1; //Method com/amd/aparapi/Kernel."<init>":()V
   4:   return


public void run();
  Code:
   Stack=5, Locals=2, Args_size=1
   0:   aload_0
   1:   iconst_0
   2:   invokevirtual   #2; //Method getGlobalId:(I)I
   5:   istore_1
   6:   aload_0
   7:   getfield        #3; //Field out:[I
   10:  iload_1
   11:  aload_0
   12:  getfield        #4; //Field in:[I
   15:  iload_1
   16:  iaload
   17:  aload_0
   18:  getfield        #4; //Field in:[I
   21:  iload_1
   22:  iaload
   23:  imul
   24:  iastore
   25:  return
}
```

现在我们能看到这个类的常量池, 默认构造方法和 `Squarer.run()` 方法.

The constant pool is a table of constant values that can be accessed from the bytecode of any methods from within this class. Some of the constants are String literals defined within the source (or literals used to name classes, fields, methods, variables or signatures), other slots represent Classes, Methods, Fields or Type signatures. These later constant pool entries cross-reference other constant pool entries to describe higher level artifact.

For example constant pool entry #1 is

```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V
```

So entry #1 defines a method. The class containing the method is defined in constant pool entry #6. So lets look at constant pool entry #6.

```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V

const #6 = class        #22;    //  com/amd/aparapi/Kernel

```

At constant pool entry #6 we find a class definition which refers to entry #22


```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V

const #6 = class        #22;    //  com/amd/aparapi/Kernel

const #22 = Asciz       com/amd/aparapi/Kernel;
```

Which just contains the String (Ascii) name of the class.

Looking back at entry #1 again, we note that the Method also references entry #17 which contains a NameAndType entry for determining the method name and the signature.


```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V

const #6 = class        #22;    //  com/amd/aparapi/Kernel


const #17 = NameAndType #10:#11;//  "<init>":()V

const #22 = Asciz       com/amd/aparapi/Kernel;
```

Entry #17’s “NameAndType” references #10 for the method name.


```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V

const #6 = class        #22;    //  com/amd/aparapi/Kernel

const #10 = Asciz       <init>;

const #17 = NameAndType #10:#11;//  "<init>":()V

const #22 = Asciz       com/amd/aparapi/Kernel;
```

And then references #11 to get the signature.

```java
const #1 = Method       #6.#17; //  com/amd/aparapi/Kernel."<init>":()V

const #6 = class        #22;    //  com/amd/aparapi/Kernel

const #10 = Asciz       <init>;

const #11 = Asciz       ()V;

const #17 = NameAndType #10:#11;//  "<init>":()V

const #22 = Asciz       com/amd/aparapi/Kernel;
```

So from constant pool #1 we ended up using slots 1,6,10,11,17 and 22 to fully resolve the method.

This looks like a lot of work, however by breaking method and field references up like this, allows the various slots to be reused by other field/method descriptions.

So when we see disassembled bytecode which references a constantpool slot the actual slot # (2 in the example below) will appear after the bytecode for invokevirtual.

```java
2:   invokevirtual   #2; Method getGlobalId:(I)I
```

Bytecode is basically able to access three things

Constant pool entries
Variable slots
Stack operands
Instructions are able to pop operands from the stack, push operands to the stack, load values from variable slots (to the stack), store values (from the stack) to variable slots, store values from accessed fields (to the stack) and call methods (popping args from the stack).

Some instructions can only handle specific types (int, float, double, and object instances - arrays are special forms of objects) and usually the first character of the instruction helps determine which type the instruction acts upon. So imul would be a multiply instruction that operates on integers, fmul would multiply two floats, dmul for doubles. Instructions that begin with ‘a’ operate on object instances.

So lets look at the first instruction.

```java
0:   aload_0
```

This instruction loads an object (a is the first character) from variable slot 0 (we’ll come back to the variable slots in a moment) and pushes it on the stack.

Variables are held in ‘slots’ that are reserved at compiled time.

Consider this static method.

```java
static int squareMe(int value){
  value += value;
  return(value);
}
```

This method requires one variable slot. At any one time there is only one variable that is live, it just happens to be an argument to the method.

The following method also contains one slot.

```java
static int squareMe(){
  int value=4;
  value += value;
  return(value);
}
```

Here we need two slots

```java
static int squareMe(int arg){
  int value=arg*arg;
  return(value);
}
```

Suprisingly the following also only requires two slots.

```java
static int squareMe(int arg){
  {
    int temp = arg*arg;
  }
  int value=arg*arg;
  return(value);
}
```

Note that in the above example the temp variable loses scope before the local variable value is used. So only two slots are required. Both temp and value can share a slot.

If we have an instance method we always require one extra slot (always slot 0) for the this reference.

So

```java
int squareMe(int arg){
  int value=arg*arg;
  return(value);
}
```

Requires three slots.

Anyway back to our bytecode

```java
0:   aload_0
```

This loads the object instance in slot 0 (this) and pushes it on the stack.

Next we have

```java
1:   iconst_0
```

Which pushes the int constant 0 on the stack. So the stack contains {this,0}

Next we have

```java
2:   invokevirtual   #2; //Method getGlobalId:(I)I
```

This is the bytecode for calling a method. Basically the instruction itself references the constant pool (we’ll come back to this ;) ) and pulls the method description in constantPool2 which happens to be the description for a method called getGlobalId() which takes an integer and returns an int.

So the VM will pop the top value (int - const 0) as the method arg, and then will pop an object reference (this!) and will call the method this.getGlobalId(0) and will push the result (an int) back on the stack.

So our stack which contains {this,0} now contains the result of this.getGlobalId(0), lets assume it is {0}. We describe this invoke instruction as consuming two operands from the stack and producing one.

Before we start executing our stack is empty {}, the slots are initialized with ‘this’ (if an instance method) and any arguments passed to the method.

```java
                                                            0   1
                                                   slots=[this, ?  ]    stack={}

                                                            0   1
0:   aload_0                                        slots=[this, ?  ]    stack={this}
                                                            0   1
1:   iconst_0                                       slots=[this, ?  ]    stack={this, 0}
                                                            0   1
2:   invokevirtual   #2; Method getGlobalId:(I)I    slots=[this, ?  ]  stack={result of this.getGlobalId(0) lets say 0}

5:   istore_1                                       slots=[this, 0  ]    stack={}

6:   aload_0                                        slots=[this, 0  ]    stack={this}

7:   getfield        #3; //Field out:[I
```
