# Aparapi 示例

> **用于展示 Aparapi 功能的示例和代码片段**

下面的文章将为你解答一些使用 Aparapi 时常见的问题.

我们欢迎您提交任何新问题或对此文章的建议.

**如果我不能写内核字段的话如何返回数据?**

在内核中将数据写入到一个长度为1的简易数组即可.

例如, 下面的内核代码会检测 `buffer[]` 中是否包含 `1234` 并将结果存于 `found[0]` 中.

```java
final int buffer[] = new int[HUGE];
final boolean found[] = new boolean[]{false};
// fill buffer somehow
kernel kernel = new kernel(){
    @Override public void run(){
        if (buffer[getGlobald()]==1234){
            found[0]=true;
        }
    }
};
kernel.execute(buffer.length);
```

上面的代码确实包含了一个竞争条件, `buffer[]` 中可能含有多个 `1234` 使内核对 `found[0]` 进行多次写入. 这在这里不是问题, 我们不关心是否有多个内核匹配到指定值, 只要有一个内核执行过写入即可.

**我如何才能既使用 Aparapi 还能继续像面向对象那样操作我的数据?**

请查阅 **新特性** 页面. Aparapi 现在可以处理简单对象数组, 这将使用 Aparapi 对原代码的影响降到最低. 但是使用值类型数组确实会有更高的性能, 为了在获得更高性能的同时还减少直接操作值类型数组, 我们可以通过一点修改来使两种形式的数据共存. 现在让我们重新思考一下N体问题.

一个 Java 开发者为了求解N体问题可能会创建一个如下类:

```java
class Body{
  float x,y,z;
  float getX(){return x;}
  void setX(float _x){ x = _x;}
  float getY(){return y;}
  void setY(float _y){ y = _y;}
  float getZ(){return z;}
  void setZ(float _z){ z = _z;}

  // other data related to Body unused by positioning calculations
}
```

开发者可能还会创建一个包含多个 `Body` 实例的容器类:

```java
class NBodyUniverse{
    final Body[] bodies = null;
    NBodyUniverse(final Bodies _bodies[]){
        bodies = _bodies;
        for (int i=0; i<bodies.length; i++){
            bodies[i].setX(Math.random()*100);
            bodies[i].setY(Math.random()*100);
            bodies[i].setZ(Math.random()*100);
        }
    }
    void adjustPositions(){
    // can use new array of object Aparapi features, but is not performant
    }
}
Body bodies = new Body[BODIES];
for (int i=0; i<bodies; i++){
    bodies[i] = new Body();
}
NBodyUniverse universe = new NBodyUniverse(bodies);
while (true){
   universe.adjustPositions();
   // display NBodyUniverse
}
```

`NBodyUniverse.adjustPostions()` 方法包含一个嵌套循环来根据所有 `Body` 的位置计算受力情况从而调整 `Body` 的位置, 使其成为一个理想的 Aparapi 示例.

虽然你可以通过 getter 和 setter 读写 `Body[]` 中实例的坐标数据, 但最好的实现是直接操作值类型数组中的数组. 如对 `Body[10]` 的读写是建立在 `x[10]`, `y[10]` 和 `z[10]` 上的.

出于性能考量, 你可能会像这样做:

```java
class Body{
    int idx;
    NBodyUniverse universe;
    void setUniverseAndIndex(NBodyUniverse _universe, int _idx){
        universe = _universe;
        idx = _idx;
    }

    // other fields not used by layout

    void setX(float _x){ layout.x[idx]=_x;}
    void setY(float _y){ layout.y[idx]=_y;}
    void setZ(float _z){ layout.z[idx]=_z;}
    float getX(){ return layout.x[idx];}
    float getY(){ return layout.y[idx];}
    float getZ(){ return layout.z[idx];}
}
class NBodyUniverse {
     final Body[] bodies;
     final int[] x, y, z;
     NBodyUniverse(Body[] _bodies){
        bodies = _bodies;
        for (int i=0; i<bodies.length; i++){
           bodies[i].setUniverseAndIndex(this, i);
           bodies[i].setX(Math.random()*100);
           bodies[i].setY(Math.random()*100);
           bodies[i].setZ(Math.random()*100);
        }
     }
     void adjustPositions(){
         // can now more efficiently use Aparapi
     }
}

Body bodies = new Body[BODIES];
for (int i=0; i<bodies; i++){
    bodies[i] = new Body();
}
NBodyUniverse universe = new NBodyUniverse(bodies);
while (true){
   universe.adjustPositions();
   // display NBodyUniverse
}
```

上面的例子展示了如何在 Java™ 代码中使用面向对象风格代码处理N体问题, 且在 Aparapi 中还能使用并行值类型数组形式以读写 `Body` 的位置.
