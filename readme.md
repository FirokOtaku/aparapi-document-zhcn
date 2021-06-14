# Aparapi 文档翻译

本项目是对 [Aparapi](https://aparapi.com/) 项目官方文档的 **自用** 简中翻译, 不保证翻译质量.

* 本项目使用 ![](https://www.deepl.com/img/logo/deepl-logo-blue.svg) [DeepL](https://www.deepl.com/translator) 辅助翻译
* 本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可  
  [ ![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png) ](http://creativecommons.org/licenses/by-nc-sa/4.0/)  
  如果您需要在您的项目中使用本项目内容, 请遵循上述协议

> [知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议 (中文)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)  

* **不允许** 对本作品的全文或部分 **直接复制粘贴再发布**  
  如需再发布本作品请进行 **有意义的** 增删改.  
  本作品很垃圾, 也请不要让垃圾一点不变味就到处传播

> 中文开发社区已经很拉了, 求求别让它更拉了  
> 每次想找点资料都跟进了废品回收站一样

翻译状态|说明
-|-
🕒|未开始
▶|进行中
⏯|暂停
✅|完成

原文|译文|翻译状态
-|-|-
**Introduction**|**介绍**
[Aparapi - About](https://aparapi.com/introduction/about.html)|[Aparapi - 关于](aparapi-about.md)|✅
[Aparapi - Getting Started](https://aparapi.com/introduction/getting-started.html)|[Aparapi - 起步](aparapi-getting-started.md)|✅
[Aparapi - FAQ](https://aparapi.com/introduction/faq.html)|[Aparapi - FAQ](aparapi-faq.md)|✅
**Documentation**|**文档**
[Aparapi Patterns](https://aparapi.com/documentation/aparapi-patterns.html)|[Aparapi 示例](aparapi-patterns.md)|✅
[Choosing Specific Devices](https://aparapi.com/documentation/choosing-specific-devices.html)|[指定运行设备](aparapi-choosing-specific-devices.md)|✅
[Converting Java to OpenCL](https://aparapi.com/documentation/converting-java-to-opencl.html)|[Java 和 OpenCL 之间的转换](aparapi-converting-java-to-opencl.md)|⏯
[Emulating Multiple Entrypoints](https://aparapi.com/documentation/emulating-multiple-entrypoints.html)|[模拟多入口](aparapi-emulating-multiple-entrypoints.md)|✅
[Explicit Buffer Handling](https://aparapi.com/documentation/explicit-buffer-handling.html)|[手动管理缓冲区](aparapi-explicit-buffer-handling.md)|✅
[HSA Enabled Lambda](https://aparapi.com/documentation/hsa-enabled-lambda.html)||🕒
[Kernel Guidelines](https://aparapi.com/documentation/kernel-guidelines.html)|[内核编程指南](aparapi-kernel-guidelines.md)|✅
[Library Agent Duality](https://aparapi.com/documentation/library-agent-duality.html)||🕒
[New Features](https://aparapi.com/documentation/new-features.html)|新特性|🕒
[OpenCL Bindings](https://aparapi.com/documentation/opencl-bindings.html)||🕒
[Private Memory Space](https://aparapi.com/documentation/private-memory-space.html)|私有内存空间|🕒
[Profiling the Kernel](https://aparapi.com/documentation/profiling-the-kernel.html)||🕒
[Setting Up HSA](https://aparapi.com/documentation/setting-up-hsa.html)||🕒
[Unit Tests](https://aparapi.com/documentation/unit-tests.html)|单元测试|🕒
[Using HSA Simulator](https://aparapi.com/documentation/using-hsa-simulator.html)||🕒
[Constant Memory](https://aparapi.com/documentation/constant-memory.html)||🕒
[Local Memory](https://aparapi.com/documentation/local-memory.html)||🕒
[Multiple Dim Ranges](https://aparapi.com/documentation/multiple-dim-ranges.html)|多维执行域|🕒
**Proposals**|**建议**
[Multiple Dim ND Range](https://aparapi.com/proposals/multiple-dim-nd-range.html)|多维执行域|🕒
[Lambdas](https://aparapi.com/proposals/lambdas.html)||🕒
[Address Space with Buffers]()||🕒
[Extensions](https://aparapi.com/proposals/extensions.html)||🕒
[Device](https://aparapi.com/proposals/device.html)||🕒
[Multiple Entry Points](https://aparapi.com/proposals/multiple-entry-points.html)||🕒
[Lambda Syntax](https://aparapi.com/proposals/lambda-syntax.html)||🕒

## 译例

### range - 执行域

一层循环就是一维执行域, 二层循环就是二维执行域, 以此类推.

后面有个 "multiple-dim range" 其实就是多层循环.

> 为什么是 "执行域"?  
> 不知道, 脑内词汇随机排列组合抖机灵
