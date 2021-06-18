# 使用 Lambda 语法

## 介绍

Java 8 已经到来, Aparapi 的 HSA 分支已经可以使用了 (尽管尚未完成), 我想我们可以用这个页面来讨论更好的 Aparapi 编程模型, 并且与 Java 8 新增的流 API 进行比较.

> 译注:  
> 2021年6月17的时候,  
> Java 18 都在路上了

## 在 Aparapi HSA 和 Aparapi Java 8 之间转换

我们的示例程序永远是 "向量和". 我们把:

```java
final float inA[] = .... // get a float array from somewhere
final float inB[] = .... // get a float from somewhere
                     // assume (inA.length==inB.length)
final float result = new float[inA.length];

for (int i=0; i<array.length; i++){
    result[i]=intA[i]+inB[i];
}
```

改写为使用 Aparapi 的形式:

```java
Kernel kernel = new Kernel(){
   @Override public void run(){
      int i= getGlobalId();
      result[i]=intA[i]+inB[i];
   }
};
Range range = Range.create(result.length);
kernel.execute(range);
```

改为 Lambda 形式:

```java
Device.hsa().forEach(result.length, i-> result[i]=intA[i]+inB[i]);
```

最接近的 Java 8 形式写法是:

```java
IntStream.range(0, result.length).parallel().forEach(i-> result[i]=intA[i]+inB[i]);
```

Aparapi 和 Java 8 都使用了 `IntConsumer` 类型的 Lambda, 所以你可以重用相关 Lambda.

```java
IntConsumer lambda = i-> result[i]=intA[i]+inB[i];

IntStream.range(0, result.length).parallel().forEach(lambda);
Device.hsa().forEach(result.length, lambda);
```

上面的代码我们有意使用 `Device` 实例执行代码, 这可以被完全隐藏掉:

```java
IntConsumer lambda = i-> result[i]=intA[i]+inB[i];

IntStream.range(0, result.length).parallel().forEach(lambda);
Aparapi.forEach(result.length, lambda);
```

我正在考虑提供更接近于 Java 8 风格的 API, 像下面这样:

```java
IntStream.range(0, result.length).parallel().forEach(lambda);
Aparapi.range(0, result.length).parallel().forEach(lambda);
```

这样方便用户在两种方案之间切换.

对于集合/数组类型, 我们提供了:

```java
T[] arr = // get an array of T from somewhere
ArrayList<T> list = // get an array backed list of T from somewhere

Aparapi.range(arr).forEach(t -> /* do something with each T */);
```

我们为特殊情况提供了支持, 比如改变图片数据:

```java
BufferedImage in, out;
Aparapi.forEachPixel(in, out, rgb[] -> rgb[0] = 0 );
```

我们可能还需要在相关的操作中做出选择:

```java
class Person{
    int age;
    String first;
    String last;
};

Aparapi.selectOne(Person[] people, (p1,p2)-> p1.age>p2.age?p1:p2 );
```

下面是一个 map-reduce 的例子.
`mapper` 可以将一种数据映射为另一种数据, 比如可以从某数据中提取子数据. 下面是一个用于将给定字符串映射成其长度的 `mapper`:

`mapper` 基类为:

```java
interface mapToInt<T>{ int map(T v); }
```

这里是实际的 `mapper`:

```java
Aparapi.range(strings).map(s->string.length())...
```

于是我们得到了一个 `int` 流, 可以使用减项函数对其作减项操作.

在这个例子中减项函数是取两 `int` 值中的最大值. 所有的减项函数必须是可交换式的.

```java
int lengthOfLongestString = Aparapi.range(strings).map(s->string.length()).reduce((k,v)-> k>v?k:v);
```

下面是一个求和减项函数:

```java
int sumOfLengths = Aparapi.range(strings).map(s ->string.length()).reduce((k,v)-> k+v);
```

对于一些常见的减项操作, 我们直接提供了接口:

```java
int sumOfLengths = Aparapi.range(strings).map(s ->string.length()).sum();
int maxOfLengths = Aparapi.range(strings).map(s ->string.length()).max();
int minOfLengths = Aparapi.range(strings).map(s ->string.length()).min();
String string = Aparapi.range(strings).map(s->string.length()).select((k,v)-> k>v);
```

最后一个需要解释一下: 我们将字符串映射成其程度, 然后取长度最大的字符串.
