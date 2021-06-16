# 多程序入口

> **如何扩展 Aparapi 才能实现多程序入口**

## 回顾单程序入口的世界

目前, Aparapi 只允许我们在单内核中分发任务. 对每个内核来说, 只有 `Kernel.run()` 方法可以用来在 GPU 上启动程序.

以 `Squarer` 内核为例, 它可以根据计算输入数组中数据的平方:

```java
Kernel squarer = new Kernel(){
   @Overide public void run(){
      int id = getGlobalId(0);
      out[id] = in[id] * in[id];
   }
};
```

如果我们需要一个计算加法的内核, 就需要创建一个全新的内核:

```java
Kernel adder = new Kernel(){
   @Overide public void run(){
      int id = getGlobalId(0);
      out[id] = in[id] * in[id];
   }
};
```

> 译注:  
> 这里原文中的代码疑似重复

那么先对数据求平方再增加一个常数就不得不调用两个内核. 或者, 也可以再创建一个 `SquarerAdder` 内核.


在 [模拟多程序入口](aparapi-emulating-multiple-entrypoints.md) 页面中我们介绍了如何提供一个额外参数来让单一程序入口模拟多程序入口.

为什么 Aparapi 不能允许将任意方法变为程序入口呢? 理想情况下, 我们希望对各种数学运算各提供一个方法, 作为更加自然易用的 API.

就像这样:

```java
class VectorKernel extends Kernel{
   public void add();
   public void sub();
   public void sqr();
   public void sqrt();
}
```

不幸的是这很难用 Aparapi 实现, 在运行时我们会遇到两个问题.

一, 当我们调用 `Kernel.execute(range)` 时, Aparapi 如何才能知道我们要执行哪些方法?
二, 第一次执行时, Aparapi 如何确定哪些方法可能是程序入口, 进而将其转换为 OpenCL?

第一个问题可以通过扩展 `Kernel.execute()` 来解决, 比如在方法里接受一个方法名作为形参:

```java
kernel.execute(SIZE, "add");
```

这种方法看似简便, 但会造成维护问题. 开发者不仅收不到编译期警告, 还更容易遇到运行时错误. 假如开发者提供了错误的方法名:

```java
kernel.execute(SIZE, "sadd"); // there is no such method
```

上面的代码可以通过编译, 但是如果不存在指定的方法, 我们只能在运行时才能发现错误.

旁白:
也许 Java 8 新增的方法引用可以帮到我们. 在下面的论文中, Brian Goetz 谈到了双冒号运算符可以直接引用方法 (`Class::method`). 这种语法在编译期就会进行检查.

所以我们可能会需要

```java
kernel.execute(SIZE, VectorKernel::add);
```

Would compile just fine, whereby

```java
kernel.execute(SIZE, VectorKernel::sadd);
```

Would yield a compile time error.

See Brian Goetz’s excellent Lambda documentation

back from Aside
The second problem (knowing which methods need to be converted to OpenCL) can probably be solved using an Annotation.

```java
class VectorKernel extends Kernel{
   @EntryPoint public void add();
   @EntryPoint public void sub();
   @EntryPoint public void sqr();
   @EntryPoint public void sqrt();
   public void nonOpenCLMethod();
}
```

Here the @EntryPoint annotation allows the Aparapi runtime to determine which methods need to be exposed.

My Extension Proposal
Here is my proposal. Not only does it allow us to reference multiple entryoints, but I think it actually improves the single entrypoint API, albeit at the cost of being more verbose.

The developer must provide an API interface
First I propose that we should ask the developer to provide an interface for all methods that we wish to execute on the GPU (or convert to OpenCL).

```java
interface VectorAPI extends AparapiAPI {
   public void add(Range range);
   public void sub(Range range);
   public void sqrt(Range range);
   public void sqr(Range range);
}
```

Note that each API takes a Range, this will make more sense in a moment.

The developer provides a bound implementation
Aparapi should provide a mechanism for mapping the proposed implementation of the API to it’s implementation.

Note the weasel words here, this is not a conventional implementation of an interface. We will use an annotation (@Implements(Class class)) to provide the binding.

```java
@Implements(VectorAPI.class) class Vector extends Kernel {
   public void add(RangeId rangeId){/*implementation here */}
   public void sub(RangeId rangeId){/*implementation here */}
   public void sqrt(RangeId rangeId){/*implementation here */}
   public void sqr(RangeId rangeId){/*implementation here */}
   public void  public void nonOpenCLMethod();
}
```

Why we can’t the implementation just implement the interface?
This would be ideal. Sadly we need to intercept a call to say VectorAPI.add(Range) and dispatch to the resulting Vector.add(RangeId) instances. If you look at the signatures, the interface accepts a Range as it’s arg (the range over which we intend to execute) whereas the implementation (either called by JTP threads or GPU OpenCL dispatch) receives a RangeId (containing the unique globalId, localId, etc fields). At the very end of this page I show a strawman implementation of a sequential loop implementation.

So how do we get an implementation of VectorAPI
We instantiate our Kernel by creating an instance using new. We then ask this instance to create an API instance. Some presumably java.util.Proxy trickery will create an implementation of the actual instance, backed by the Java implementation.

So execution would look something like.

```java
Vector kernel = new Vector();
VectorAPI kernelApi = kernel.api();
Range range = Range.create(SIZE);
kernalApi.add(range);
```

So the Vector instance is a pure Java implementation. The extracted API is the bridge to the GPU.

Of course then we can also execute using an inline call through api()

```java
Vector kernel = new Vector();
Range range = Range.create(SIZE);
kernel.api().add(range);
kernel.api().sqrt(range);
```

or even expose api as public final fields

```java
Vector kernel = new Vector();
Range range = Range.create(SIZE);
kernel.api.add(range);
kernel.api.sqrt(range);
```

How would our canonical Squarer example look

```java
interface SquarerAPI extends AparapiAPI{
   square(Range range);
}

@Implement(SquarerAPI) class Squarer extends Kernel{
   int in[];
   int square[];
   public void square(RangeId rangeId){
      square[rangeId.gid] = in[rangeId.gid]*in[rangeId.gid];
   }
}
```

Then we execute using

```java
Squarer squarer = new Squarer();
// fill squarer.in[SIZE]
// create squarer.values[SIZE];


squarer.api().square(Range.create(SIZE));
```

Extending this proposal to allow argument passing
Note that we have effectively replaced the use of the ‘abstract’ squarer.execute(range) with the more concrete squarer.api().add(range).

Now I would like to propose that we take one more step by allowing us to pass arguments to our methods.

Normally Aparapi captures buffer and field accesses to create the args that it passes to the generated OpenCL code. In our canonical squarer example the in[] and square[] buffers are captured from the bytecode and passed (behind the scenes) to the OpenCL.

However, by exposing the actual method we want to execute, we could also allow the API to accept parameters.

So our squarer example would go from

```java
interface SquarerAPI extends AparapiAPI{
   square(Range range);
}

@Implement(SquarerAPI) class Squarer extends Kernel{
   int in[];
   int square[];
   public void square(RangeId rangeId){
      square[rangeId.gid] = in[rangeId.gid]*in[rangeId.gid];
   }
}


Squarer squarer = new Squarer();
// fill squarer.in[SIZE]
// create squarer.values[SIZE];

squarer.api().square(Range.create(SIZE));
```

to

```java
interface SquarerAPI extends AparapiAPI{
   square(Range range, int[] in, int[] square);
}

@Implement(SquarerAPI) class Squarer extends Kernel{
   public void square(RangeId rangeId, int[] in, int[] square){
      square[rangeId.gid] = in[rangeId.gid]*in[rangeId.gid];
   }
}


Squarer squarer = new Squarer();
int[] in = // create and fill squarer.in[SIZE]
int[] square = // create squarer.values[SIZE];

squarer.api().square(Range.create(SIZE), in, result);
I think that this makes Aparapi look more conventional. It also allows us to allow overloading for the first time.


interface SquarerAPI extends AparapiAPI{
   square(Range range, int[] in, int[] square);
   square(Range range, float[] in, float[] square);
}

@Implement(SquarerAPI) class Squarer extends Kernel{
   public void square(RangeId rangeId, int[] in, int[] square){
      square[rangeId.gid] = in[rangeId.gid]*in[rangeId.gid];
   }
   public void square(RangeId rangeId, float[] in, float[] square){
      square[rangeId.gid] = in[rangeId.gid]*in[rangeId.gid];
   }
}


Squarer squarer = new Squarer();
int[] in = // create and fill squarer.in[SIZE]
int[] square = // create squarer.values[SIZE];

squarer.api().square(Range.create(SIZE), in, result);
float[] inf = // create and fill squarer.in[SIZE]
float[] squaref = // create squarer.values[SIZE];

squarer.api().square(Range.create(SIZE), inf, resultf);
```

test harness

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;


public class Ideal{

   public static class OpenCLInvocationHandler<T> implements InvocationHandler {
       Object instance;
       OpenCLInvocationHandler(Object _instance){
          instance = _instance;
       }
      @Override public Object invoke(Object interfaceThis, Method interfaceMethod, Object[] interfaceArgs) throws Throwable {
         Class clazz = instance.getClass();

         Class[] argTypes =  interfaceMethod.getParameterTypes();
         argTypes[0]=RangeId.class;
         Method method = clazz.getDeclaredMethod(interfaceMethod.getName(), argTypes);


         if (method == null){
            System.out.println("can't find method");
         }else{
            RangeId rangeId = new RangeId((Range)interfaceArgs[0]);
            interfaceArgs[0]=rangeId;
            for (rangeId.wgid = 0; rangeId.wgid <rangeId.r.width; rangeId.wgid++){
                method.invoke(instance, interfaceArgs);
            }
         }

         return null;
      }
   }

   static class Range{
      int width;
      Range(int _width) {
         width = _width;
      }
   }

   static class Range2D extends Range{
      int height;

      Range2D(int _width, int _height) {
         super(_width);
         height = _height;
      }
   }

   static class Range1DId<T extends Range>{
      Range1DId(T _r){
         r = _r;
      }
      T r;

      int wgid, wlid, wgsize, wlsize, wgroup;
   }

   static class RangeId  extends Range1DId<Range>{
      RangeId(Range r){
         super(r);
      }
   }

   static class Range2DId extends Range1DId<Range2D>{
      Range2DId(Range2D r){
         super(r);
      }

      int hgid, hlid, hgsize, hlsize, hgroup;
   }





   static <T> T create(Object _instance, Class<T> _interface) {
      OpenCLInvocationHandler<T> invocationHandler = new OpenCLInvocationHandler<T>(_instance);
      T instance = (T) Proxy.newProxyInstance(Ideal.class.getClassLoader(), new Class[] {
            _interface,

      }, invocationHandler);
      return (instance);

   }



   public static class Squarer{
      interface API {
         public API foo(Range range, int[] in, int[] out);
         public Squarer dispatch();

      }

      public API foo(RangeId rangeId, int[] in, int[] out) {
         out[rangeId.wgid] = in[rangeId.wgid]*in[rangeId.wgid];
         return(null);
      }
   }

   /**
    * @param args
    */
   public static void main(String[] args) {

      Squarer.API squarer = create(new Squarer(), Squarer.API.class);
      int[] in = new int[] {
            1,
            2,
            3,
            4,
            5,
            6
      };
      int[] out = new int[in.length];
      Range range = new Range(in.length);

      squarer.foo(range, in, out);

      for (int s:out){
         System.out.println(s);
      }

   }

}
```
