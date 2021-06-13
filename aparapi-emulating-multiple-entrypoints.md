# 模拟多入口

> **如何在现有 Aparapi API 基础上模拟多入口**

Aparapi 目前不支持多入口, 但是在目前的 API 体系下可以通过一些技巧实现类似功能.

假设我们想创建一个通用的矢量数学运算内核, 提供单数平方, 平方根和二进制加减功能. 受 Aparapi 目前的 API 限制, 我们不能直接对外提供接口. 但是我们可以传递一个单独的参数来决定内核希望执行的 "功能", 从而接近对外提供独立方法.

```java
class VectorKernel extends Kernel{
    float[] lhsOperand;
    float[] rhsOperand;
    float[] unaryOperand;
    float[] result;
    final static int FUNC_ADD =0;
    final static int FUNC_SUB =1;
    final static int FUNC_SQR =2;
    final static int FUNC_SQRT =3;
    // other functions
    int function;
    @Override public void run(){
        int gid = getGlobalId(0){
        if (function==FUNC_ADD){
           result[gid]=lhsOperand[gid]+rhsOperand[gid];
        }else if (function==FUNC_SUB){
           result[gid]=lhsOperand[gid]-rhsOperand[gid];
        }else if (function==FUNC_SQR){
           result[gid]=unaryOperand[gid]*unaryOperand[gid];
        }else if (function==FUNC_ADD){
           result[gid]=sqrt(unaryOperand[gid]);
        }else if ....
    }
}
```

用这个内核来计算两个向量和的平方, 可以像下面这样:

```java
int SIZE=1024;
Range range = Range.create(SIZE);
VectorKernel vk = new VectorKernel();
vk.lhsOperand = new float[SIZE];
vk.rhsOperand = new float[SIZE];
vk.unaryOperand = new float[SIZE];
vk.result = new float[SIZE];

// fill lhsOperand ommitted
// fill rhsOperand ommitted
vk.function = VectorKernel.FUNC_ADD;
vk.execute(range);
System.arrayCopy(vk.result, 0, vk.unaryOperand, 0, SIZE);
vk.function = VectorKernel.FUNC_SQRT;
vk.execute(range);
```

这种方法非常普通, 我也已经成功地用这种方法来执行各种管线运算, 比如 FFT. 但这并不是很好的解决方法: 首先 API 会变得很笨拙, 
This approach is fairly common and I have used it successfully to perform various pipeline stages for calculating FFT’s for example. Whilst this is functional it is not a great solution. First the API is clumsy. We have to mutate the state of the kernel instance and then re-arrange the arrays manually to chain math operations. 当然, 我们可以把这一切封装到工具类和工具方法里, 然后对外提供 `helper.add(lhs,rhs)` 和 `helper.sqrt(lhs,rhs)` 这样的接口. 

```java
class VectorKernel extends Kernel{
    float[] lhsOperand;
    float[] rhsOperand;
    float[] unaryOperand;
    float[] result;
    final static int FUNC_ADD =0;
    final static int FUNC_SUB =1;
    final static int FUNC_SQR =2;
    final static int FUNC_SQRT =3;
    // other functions
    int function;
    @Override public void run(){
        int gid = getGlobalId(0){
        if (function==FUNC_ADD){
           result[gid]=lhsOperand[gid]+rhsOperand[gid];
        }else if (function==FUNC_SUB){
           result[gid]=lhsOperand[gid]-rhsOperand[gid];
        }else if (function==FUNC_SQR){
           result[gid]=unaryOperand[gid]*unaryOperand[gid];
        }else if (function==FUNC_ADD){
           result[gid]=sqrt(unaryOperand[gid]);
        }else if ....
    }
    private void binary(int operator, float[] lhs, float[] rhs){
       lhsOperand = lhs;
       rhsOperand = rhs;
       function=operator;
       execute(lhs.length());
    }
    public void add(float[] lhs, float[] rhs){
       binary(FUNC_ADD, lhs, rhs);
    }

    public void sub(float[] lhs, float[] rhs){
       binary(FUNC_SUB, lhs, rhs);
    }

    private void binary(int operator, float[] rhs){
       System.arrayCopy(result, 0, lhsOperand, result.length);
       rhsOperand = rhs;
       function=operator;
       execute(lhsOperand.legth());
    }

    public void add(float[] rhs){
       binary(FUNC_ADD,  rhs);
    }

    public void sub( float[] rhs){
       binary(FUNC_SUB,  rhs);
    }

    private void unary(int operator, float[] unary){
       unaryOperand = unary;
       function=operator;
       execute(unaryOperand.length());
    }

    public void sqrt(float[] unary){
       unary(FUNC_SQRT, unary);
    }

    private void unary(int operator){
       System.array.copy(result, 0, unaryOperand, 0, result.length);
       function=operator;
       execute(unaryOperand.length());
    }

    public void sqrt(){
       unary(FUNC_SQRT);
    }

}

VectorKernel vk = new VectorKernel(SIZE);
vk.add(copyLhs, copyRhs);  // copies args to lhs and rhs operands
                           // sets function type
                           // and executes kernel
vk.sqrt();                 // because we have no arg
                           // copies result to unary operand
                           // sets function type
                           // execute kernel
```

但是, 这种方法还有一个缺陷: 默认情况下会强制进行不必要的缓冲区数据拷贝.

Aparapi 分析上述 `Kernel.run()` 方法的时候会发现字节码从 `lhsOperand`, `rhsOperand` 和 `unaryOperand` 数组/缓冲区中读取数据. 显然, 在字节码分析阶段中我们无法预测会使用哪个 "函数类型", 所以每次执行 `Kernel.run()` 的时候 Aparapi 必须将所有3个缓冲区内的数据复制到 GPU 上. 这其中不仅产生了数据交换成本, 复制过去数据中的一部分也是无用的. 当然, 我们可以显式进行缓冲区管理, 减少数据交换的成本. 理想情况下我们会这样修改工具方法:

```java
class VectorKernel extends Kernel{
    float[] lhsOperand;
    float[] rhsOperand;
    float[] unaryOperand;
    float[] result;
    final static int FUNC_ADD =0;
    final static int FUNC_SUB =1;
    final static int FUNC_SQR =2;
    final static int FUNC_SQRT =3;
    // other functions
    int function;
    @Override public void run(){
        int gid = getGlobalId(0){
        if (function==FUNC_ADD){
           result[gid]=lhsOperand[gid]+rhsOperand[gid];
        }else if (function==FUNC_SUB){
           result[gid]=lhsOperand[gid]-rhsOperand[gid];
        }else if (function==FUNC_SQR){
           result[gid]=unaryOperand[gid]*unaryOperand[gid];
        }else if (function==FUNC_ADD){
           result[gid]=sqrt(unaryOperand[gid]);
        }else if ....
    }
    private void binary(int operator, float[] lhs, float[] rhs){
       lhsOperand = lhs;
       rhsOperand = rhs;
       function=operator;
       put(lhsOperand).put(rhsOperand);
       execute(lhs.length());
       get(result);
    }
    public void add(float[] lhs, float[] rhs){
       binary(FUNC_ADD, lhs, rhs);
    }

    public void sub(float[] lhs, float[] rhs){
       binary(FUNC_SUB, lhs, rhs);
    }

    private void binary(int operator, float[] rhs){
       System.arrayCopy(result, 0, lhsOperand, result.length);
       rhsOperand = rhs;
       function=operator;
       put(lhsOperand).put(rhsOperand);
       execute(lhsOperand.legth());
       get(result);
    }

    public void add(float[] rhs){
       binary(FUNC_ADD,  rhs);
    }

    public void sub( float[] rhs){
       binary(FUNC_SUB,  rhs);
    }

    private void unary(int operator, float[] unary){
       unaryOperand = unary;
       function=operator;
       put(unaryOperand);
       execute(unaryOperand.length());
       get(result);
    }

    public void sqrt(float[] unary){
       unary(FUNC_SQRT, unary);
    }

    private void unary(int operator){
       System.array.copy(result, 0, unaryOperand, 0, result.length);
       function=operator;
       put(unaryOperand);
       execute(unaryOperand.length());
       get(result);

    }

    public void sqrt(){
       unary(FUNC_SQRT);
    }

}
```
