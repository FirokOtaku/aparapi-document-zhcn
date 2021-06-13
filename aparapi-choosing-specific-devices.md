# 指定设备

> **用新的 **设备 API** 将内核的运行在指定设备上**

早先版本的 Aparapi 会将内核运行到找到的第一块 GPU 上, 这限制了用户对代码运行做更精细的控制, 特别是有时候找到的第一块 GPU 可能不适合运行指定代码. 我们提供了新的 API 允许您指定内核应运行在什么设备上.

您可以使用新引入的 `Device` 类实例来选择将代码运行在特定设备上; 或者您可以调用工具方法 `Device.firstGPU()` 或 `Device.best()`. 用户可以遍历所有设备, 根据一些标准 (功能? 供应商名称?) 选择设备使用.

下面是如何选择 "最好" (性能最强的) 设备:

```java
Device device = Device.best();
```

下面是如何使用第一块 AMD GPU 设备:

```java
Device chosen=null;
for (Device device: devices.getAll()){
    if (device.getVendor().contains("AMD") && device.isGPU()){
        chosen = device;
        break;
    }
}
```

您可以调用 `Device` 实例的属性方法 (`isGPU()`, `isOpenCL()`, `isGroup()`, `isJava()`, `getOpenCLPlatform()`, `getMaxMemory()`, `getLocalSizes()`) 来辨别此设备相关特性.

想要指定内核运行在哪个设备上, 您需要通过设备实例创建 `Range`.

```java
Range range = device.createRange2D(width, height);
```

在您对设备底层架构有一定了解的情况下, 您可以创建自定义 `Range`. 比如 `device.createRange3D(1024, 1024, 1024, 16, 16, 16)` 的调用在设备不支持指定大小 (16 × 16 × 16) 本地容量时会失败.

可以由设备实例创建 `Range` 或为手动创建的 `Range` 实例指定设备.

假设有如下代码:

```java
Range range = Range.create(width, height);
range.setDevice(device);
```

上面的 `Range` 将会强制代码运行在指定设备上.

假设有一个内核:

```java
Kernel kernel = new Kernel(){
    @Override public void run(){
      ...
    }
}
```

然后使用由设备创建的 `Range`:

```java
Device device = Device.firstGPU();
Kernel kernel = new Kernel(){
    @Override public void run(){
      // uses input[];
    }
};
range = device.createRange2D(1024, 1024);
kernel.execute(range);
```

现在我们强制让内核运行在第一块 GPU 上了.
