# 选择执行设备

> **我们应如何选择运行设备**

目前 Aparapi 默认使用第一块 GPU 或 CPU 执行代码. 运行一些简单内核时这不会有什么问题, 但如果我们需要用到一些高级特性 (如内存屏障或私有缓冲区) 就可能会遇到问题; 或者, 我们有时候也必须手动选择具有合适缓冲区 (显存) 大小的设备.

所以我们引入了一套新的 API 用于查询设备信息, 方便在设备中分配大小合适的缓冲区, 或是将内核调度到合适的设备上运行.

总的来说, 我们调用工厂方法要求 Aparapi "给" 我们一个设备:

```java
Device device = Device.best();
```

类似的工厂方法还有 `getBestGPU()`, `getFirstCPU()`, `getJavaMultiThread()` 和 `getJavaSequential()`, 开发者可以自行选用.

请注意, 除了真实设备, 这些接口还会可能返回 "伪设备", 比如 `JavaMultiThread` 和 `Sequential`. 我们也允许对这些 "伪设备" 进行分组, 所以 `getAllGPUDevices()` 方法的返回值会包含 "伪设备".

```java
Device chosen=null;
for (Device device: devices.getAll()){
   if (device.getVendor().contains("AMD") && device.isGPU()){
      chosen = device;
      break;
   }
}
```

`Device` 提供了一些实例方法 (`isGPU()`, `isOpenCL()`, `isGroup()`, `isJava()`, `getOpenCLPlatform()`, `getMaxMemory()`, `getLocalSizes()`) 用于查询设备属性; 使用的时候可能需要将 `Device` 实例转换为具体的类型.

下面是根据设备缓存大小分配缓冲区的示例:

```java
Device device = Device.best();
if (device instanceof OpenCLDevice){
   OpenCLDevice openCLDevice  = (OpenCLDevice)device;
   char input[] = new char[openCLDevice.getMaxMemory()/4);
}
```

`Device` 实例提供了工厂方法用于创建 `Range`:

```java
Range range = device.createRange2D(width, height);
```

这让 `Range` 获得设备的底层信息. 打比方, 如果设备不允许 16×16×16 大小的执行域, `device.createRange3D(1024, 1024, 1024, 16, 16, 16)` 的调用就会失败.

从 `device.createRangeXX()` 创建执行域实例还会使相关的代码执行到指定设备, 比如:

```java
Range range = device.createRange2D(width, height);
// implied range.setDevice(device);
// This basically means that the Range locks the device that it can be used with.

// So when we have a Kernel.

Kernel kernel = new Kernel(){
    @Override public void run(){
      ...
    }
}
```

然后使用:

```java
Device device = Device.firstGPU();
final char input[] = new char[((OpenCLDevice)device).getMaxMemory()/4);
Kernel kernel = new Kernel(){
    @Override public void run(){
      // uses input[];
    }
};
range = device.createRange2D(1024, 1024);
kernel.execute(range);
```

这样我们就强制指定代码运行在第一块 GPU 上了. 当然, 还有回退到 Java 模式的可能. (也许我们应该禁用这种情况下的回退?)

```java
kernel.execute( Device.firstGPU().getRange2D(width, height));
```
