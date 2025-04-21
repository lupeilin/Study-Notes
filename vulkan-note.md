## 🧠 `triangle.cpp` 整体逻辑思维导图（Markdown）

```
markdownVulkan Triangle Example
├── 初始化 Vulkan
│   ├── 创建 VkInstance（实例）
│   ├── 选择物理设备（vkEnumeratePhysicalDevices）
│   ├── 创建逻辑设备（VkDevice）与队列（VkQueue）
│   ├── 创建窗口系统表面（VkSurfaceKHR）
│   ├── 创建交换链（VkSwapchainKHR）
│   └── 创建 Command Pool / Command Buffers
│
├── 准备资源
│   ├── 顶点数据（位置 + 颜色）
│   │   ├── 创建 Staging Buffer（Host Visible）
│   │   ├── 创建 Device Buffer（GPU Local）
│   │   └── 使用 vkCmdCopyBuffer 拷贝数据
│   ├── 索引数据
│   ├── Uniform Buffer（每帧一个）
│   │   └── 保存 MVP 矩阵
│   └── 着色器模块（SPIR-V）
│       ├── triangle.vert.spv
│       └── triangle.frag.spv
│
├── 设置描述符（Descriptor）
│   ├── Descriptor Set Layout
│   │   └── layout(binding=0) uniform buffer
│   ├── Descriptor Pool
│   └── 分配 Descriptor Sets（每帧一套）
│       └── 绑定 UBO 到 DescriptorSet
│
├── 创建 RenderPass
│   ├── 定义 Color Attachment（清除 + 储存）
│   ├── 定义 Depth Attachment
│   ├── 设置 Subpass（颜色 + 深度）
│   └── 设置 Subpass 依赖
│
├── 创建 Framebuffer（每个 Swapchain Image）
│   └── 将 Color + Depth image view 与 RenderPass 绑定
│
├── 创建图形管线（Graphics Pipeline）
│   ├── 顶点输入布局（VertexInputState）
│   ├── 着色器阶段（vertex / fragment）
│   ├── 设置视口/裁剪区域（viewport/scissor）
│   ├── 光栅化状态
│   ├── 多重采样、深度测试
│   └── pipelineLayout（包含 descriptor set layout）
│
├── 命令缓冲构建
│   ├── 每帧构建一次 CommandBuffer
│   │   ├── vkBeginRenderPass（绑定 Framebuffer）
│   │   ├── vkCmdBindPipeline
│   │   ├── vkCmdBindDescriptorSets
│   │   ├── vkCmdBindVertexBuffers / IndexBuffers
│   │   ├── vkCmdDrawIndexed
│   │   └── vkEndRenderPass
│
├── 渲染主循环（renderLoop）
│   ├── 处理窗口事件（Win32 / SDL / GLFW）
│   ├── 等待上帧 GPU 信号（Fence）
│   ├── 获取交换链图像（vkAcquireNextImageKHR）
│   ├── 更新当前帧 UBO 数据
│   ├── 提交 CommandBuffer（vkQueueSubmit）
│   └── 呈现图像（vkQueuePresentKHR）
│
└── 清理资源
    ├── 销毁 Buffers / Framebuffers
    ├── 销毁 Pipeline / RenderPass
    ├── 销毁 Device / Instance
    └── 等待 GPU 空闲
```

------

## 📌 核心模块归类参考



| 模块           | 包含内容                                                 |
| -------------- | -------------------------------------------------------- |
| **初始化模块** | VkInstance、VkDevice、Swapchain、CommandPool             |
| **资源模块**   | 顶点/索引缓冲、UBO、着色器                               |
| **同步模块**   | Semaphores、Fences                                       |
| **渲染模块**   | RenderPass、Framebuffer、GraphicsPipeline、CommandBuffer |
| **渲染循环**   | Acquire → Update → Record → Submit → Present             |



## 🧠 `triangle.cpp` 函数调用链逻辑图（按实际函数划分）

```
markdownmain() / WinMain()
└── VulkanExample::VulkanExample()
    └── VulkanExampleBase() 构造 → 初始化平台窗口 + Vulkan 实例
        └── initVulkan()
            ├── createInstance()                → 创建 VkInstance
            ├── setupDebugMessenger()           → 设置验证层回调（可选）
            ├── createSurface()                 → 创建窗口表面 VkSurfaceKHR
            ├── pickPhysicalDevice()            → 选择 GPU
            ├── createLogicalDevice()           → 创建逻辑设备 VkDevice
            ├── createSwapChain()               → 创建交换链 VkSwapchainKHR
            ├── createImageViews()              → 为交换链图像创建 ImageView
            ├── createDepthResources()          → 创建深度图像与 ImageView
            ├── createRenderPass()              → 创建 RenderPass（颜色+深度）
            ├── createDescriptorSetLayout()     → 创建描述符布局
            ├── createGraphicsPipeline()        → 创建图形管线（绑定 renderpass）
            ├── createFramebuffers()            → 为每个图像创建 Framebuffer
            ├── createCommandPool()             → 创建 Command Pool
            ├── createVertexBuffer()            → 顶点数据缓冲区（Staging 拷贝）
            ├── createIndexBuffer()             → 索引缓冲区（同上）
            ├── createUniformBuffers()          → 每帧一个 Uniform Buffer
            ├── createDescriptorPool()          → 分配描述符池
            ├── createDescriptorSets()          → 为每帧分配 descriptor sets
            └── createCommandBuffers()          → 分配 command buffer
        └── setupWindow(hInstance, WndProc)     → 创建 Win32 窗口
        └── prepare()                            → 预加载资源（通常为空/补充工作）
        └── renderLoop()                         → 开始渲染主循环
            └── render()
                ├── wait for Fence
                ├── vkAcquireNextImageKHR()     → 获取图像
                ├── updateUniformBuffer()       → 更新 UBO 数据
                ├── recordCommandBuffer()       → 记录 command buffer
                    └── vkBeginRenderPass()
                        └── vkCmdBindPipeline()
                        └── vkCmdBindVertexBuffers()
                        └── vkCmdBindIndexBuffer()
                        └── vkCmdBindDescriptorSets()
                        └── vkCmdDrawIndexed()
                    └── vkEndRenderPass()
                ├── vkQueueSubmit()             → 提交 GPU 执行
                └── vkQueuePresentKHR()         → 呈现图像
```

------

## 🗂️ 按功能划分的函数清单

### 🔧 初始化部分（一次性）



| 函数名                        | 说明                              |
| ----------------------------- | --------------------------------- |
| `initVulkan()`                | 初始化所有 Vulkan 资源            |
| `createInstance()`            | 创建 VkInstance                   |
| `setupDebugMessenger()`       | 设置验证层调试回调                |
| `createSurface()`             | 和平台窗口系统结合                |
| `pickPhysicalDevice()`        | 选择合适的 GPU                    |
| `createLogicalDevice()`       | 创建 VkDevice                     |
| `createSwapChain()`           | 创建交换链                        |
| `createImageViews()`          | 创建图像视图                      |
| `createDepthResources()`      | 创建深度缓冲                      |
| `createRenderPass()`          | 创建渲染过程蓝图                  |
| `createDescriptorSetLayout()` | 设置描述符结构                    |
| `createGraphicsPipeline()`    | 设置渲染状态（shader、raster 等） |
| `createFramebuffers()`        | 为每张交换链图像配 Framebuffer    |
| `createCommandPool()`         | 创建命令池                        |
| `createCommandBuffers()`      | 分配命令缓冲区                    |

------

### 🧾 数据资源准备



| 函数名                   | 说明                                   |
| ------------------------ | -------------------------------------- |
| `createVertexBuffer()`   | 创建顶点缓冲（用 staging buffer 拷贝） |
| `createIndexBuffer()`    | 创建索引缓冲                           |
| `createUniformBuffers()` | 每帧一个 UBO，保存 MVP 矩阵            |
| `createDescriptorPool()` | 分配 Descriptor Sets 所需的池          |
| `createDescriptorSets()` | 每帧一套绑定资源的 descriptor set      |

------

### 🌀 每帧运行时调用（render loop）



| 函数名                            | 说明                              |
| --------------------------------- | --------------------------------- |
| `render()`                        | 每帧调用                          |
| `updateUniformBuffer(frameIndex)` | 每帧写入新的 MVP 矩阵             |
| `recordCommandBuffer(frameIndex)` | 重建 command buffer，记录绘图命令 |
| `vkAcquireNextImageKHR()`         | 获取可用图像                      |
| `vkQueueSubmit()`                 | 提交 GPU 执行                     |
| `vkQueuePresentKHR()`             | 呈现图像                          |

------

### 🧹 清理阶段（未在 main 中直接出现）



| 函数名                            | 说明                           |
| --------------------------------- | ------------------------------ |
| `cleanupSwapChain()`              | 重建 swapchain 时会先清除资源  |
| 析构函数 / `~VulkanExampleBase()` | 程序退出时清除所有 Vulkan 资源 |

------

## ✅ 小结



| 阶段       | 涉及函数                             | 说明                     |
| ---------- | ------------------------------------ | ------------------------ |
| 初始化阶段 | `initVulkan()` 内的所有子函数        | 加载 Vulkan 与资源准备   |
| 资源构建   | Vertex、Index、UBO、Pipeline         | 手动构建图形流程         |
| 渲染执行   | `render()` → `recordCommandBuffer()` | 每帧运行逻辑             |
| 资源清理   | 析构/cleanupSwapChain                | 程序关闭时或 resize 触发 |

​		

## 🔍 推荐阅读顺序（如果你在读代码）

1. `WinMain()` → `VulkanExample` 构造流程
2. 阅读 `initVulkan()` 中每个函数，理解资源初始化（创建对象的时机和顺序）
3. 阅读 `createGraphicsPipeline()`，掌握着色器 + 状态配置
4. 阅读 `recordCommandBuffer()`，了解绘制流程
5. 阅读 `render()` 和 `renderLoop()`，了解每帧逻辑
6. 阅读 `updateUniformBuffer()`，理解数据如何传入着色器
7. 阅读 `cleanupXXX()` 系列函数，掌握资源生命周期









#### ✅instance

`VkInstance` 是 Vulkan 中的**顶级对象**，代表了你这个程序使用 Vulkan 的上下文，它负责：

> 💡 “连接应用程序 ↔ Vulkan 驱动 ↔ GPU 环境”

------

## 🔍 它的作用可以简单理解为：



| 你程序的角色    | Vulkan 实例（`VkInstance`）的作用                  |
| --------------- | -------------------------------------------------- |
| 初始化 Vulkan   | 建立和驱动的连接                                   |
| 加载物理设备    | 列出有哪些可用 GPU（`vkEnumeratePhysicalDevices`） |
| 创建窗口表面    | 和平台 API（如 Win32/XCB/Cocoa）结合               |
| 开启扩展/验证层 | 在 `VkInstanceCreateInfo` 中配置                   |
| 注册调试工具    | 比如 `VK_EXT_debug_utils` 回调也是和实例关联的     |









### ✅获取物理设备属性、功能和内存结构

```c++
vkGetPhysicalDeviceProperties(physicalDevice, &deviceProperties);
vkGetPhysicalDeviceFeatures(physicalDevice, &deviceFeatures);
vkGetPhysicalDeviceMemoryProperties(physicalDevice, &deviceMemoryProperties);
```

#### 含义：

- `deviceProperties`：包含硬件信息（如 GPU 名称、最大纹理尺寸、limits 等）
- `deviceFeatures`：描述该物理设备支持哪些可选功能（如 geometry shader、multiDrawIndirect 等）
- `deviceMemoryProperties`：获取 GPU 的内存类型（如是否支持 `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT`）

📌 **这三类信息是之后分配内存、启用特性的重要依据。**



Vulkan 是**显式设计**的 API，它要求你：

- 明确声明你要用哪些功能
- 明确声明你要用哪些扩展
- 明确选择和绑定资源

这段代码正体现了“使用前声明、声明前查询”的 Vulkan 哲学。









### ✅获取队列族（Queue Family）信息

```
uint32_t queueFamilyCount;
vkGetPhysicalDeviceQueueFamilyProperties(physicalDevice, &queueFamilyCount, nullptr);
assert(queueFamilyCount > 0);
queueFamilyProperties.resize(queueFamilyCount);
vkGetPhysicalDeviceQueueFamilyProperties(physicalDevice, &queueFamilyCount, queueFamilyProperties.data());
```

#### 🔍 解释：

- `vkGetPhysicalDeviceQueueFamilyProperties` 是用于获取物理设备支持的 **队列族（Queue Families）**。
- 每个 GPU 通常有多个队列族，如：
  - 图形队列（Graphics）
  - 计算队列（Compute）
  - 转换队列（Transfer）
  - 仅支持呈现（Present）的队列

#### 🚩 为什么要获取这些？

- 创建逻辑设备时，需要指定**从哪个队列族获取队列（如 graphics queue）**
- 你需要找到**支持 VK_QUEUE_GRAPHICS_BIT 和 VK_QUEUE_PRESENT_KHR 的队列族**
- 这会影响后续的命令提交和 swapchain 呈现

#### 🧠 举例队列族判断：

```
for (uint32_t i = 0; i < queueFamilyCount; i++) {
    if (queueFamilyProperties[i].queueFlags & VK_QUEUE_GRAPHICS_BIT) {
        graphicsQueueIndex = i;
    }
}
```

------

###  获取设备支持的扩展列表

```
uint32_t extCount = 0;
vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extCount, nullptr);

if (extCount > 0)
{
    std::vector<VkExtensionProperties> extensions(extCount);
    if (vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extCount, &extensions.front()) == VK_SUCCESS)
    {
        for (auto& ext : extensions)
        {
            supportedExtensions.push_back(ext.extensionName);
        }
    }
}
```

#### 🔍 解释：

- 通过 `vkEnumerateDeviceExtensionProperties()` 获取物理设备支持的 **设备级扩展（如 VK_KHR_swapchain）**
- 查询结果是 `VkExtensionProperties` 数组
- 最终将支持的扩展名（字符串）收集进 `supportedExtensions`

#### 📌 这一步的用途是：

- 后续调用 `getEnabledExtensions()` 时，可以检查某个扩展是否支持，再决定是否启用

#### 🧠 示例用途：

```
if (std::find(supportedExtensions.begin(), supportedExtensions.end(), VK_KHR_SWAPCHAIN_EXTENSION_NAME) != supportedExtensions.end()) {
    enabledExtensions.push_back(VK_KHR_SWAPCHAIN_EXTENSION_NAME);
}
```

------

## ✅ 这些信息如何影响设备创建？

- 创建逻辑设备时需要指定：
  - 哪个队列族？几个队列？
  - 启用了哪些扩展？
  - 是否启用了 validation layer？









## 🧩 `createLogicalDevice`主要工作流程

1. **选择队列类型**（`requestedQueueTypes`）：
   - 根据需要的队列类型（如图形、传输等）选择合适的队列族。
   - 通过 `vkGetPhysicalDeviceQueueFamilyProperties()` 获取设备支持的队列族。
2. **创建队列信息（`VkDeviceQueueCreateInfo`）**：
   - 每个队列族的队列数量、优先级、支持的队列类型会被封装进 `VkDeviceQueueCreateInfo` 结构体。
3. **设置设备特性**（`VkPhysicalDeviceFeatures`）：
   - 通过 `enabledFeatures` 启用设备支持的特性。
4. **设置设备扩展**：
   - 配置 `enabledExtensions` 以启用设备支持的扩展（例如：Swapchain、RayTracing）。
5. **创建逻辑设备**（`VkDeviceCreateInfo`）：
   - 在 `VkDeviceCreateInfo` 结构体中指定队列信息、扩展、特性等，调用 `vkCreateDevice()` 来创建逻辑设备。
6. **返回设备创建结果**：
   - 函数会返回 `VkResult`，指示设备创建是否成功。











##  背景知识：什么是 `Queue Family`？

在 Vulkan 中，GPU 的命令执行被分为不同种类的“队列”（`Queue`），而这些队列被划分在一个或多个“队列族”（`QueueFamily`）中。每个 Queue Family 支持某些特定类型的操作（通过 `VkQueueFlags` 标记）：



| 队列标志                | 说明                                  |
| ----------------------- | ------------------------------------- |
| `VK_QUEUE_GRAPHICS_BIT` | 支持图形绘制命令                      |
| `VK_QUEUE_COMPUTE_BIT`  | 支持计算着色器命令                    |
| `VK_QUEUE_TRANSFER_BIT` | 支持数据传输命令（buffer/image 拷贝） |

------

## 🧩 该函数的逻辑分为 3 步

### 🥇 Step 1：优先寻找**专用的计算队列**

```
cpp



if ((queueFlags & VK_QUEUE_COMPUTE_BIT) == queueFlags)
```

表示：如果用户**只请求 `VK_QUEUE_COMPUTE_BIT`**，就去寻找“**只支持计算、不支持图形**”的队列（这样可以避免计算和图形抢资源）：

```
cpp



if ((queueFlags[i] & VK_QUEUE_COMPUTE_BIT) && !(queueFlags[i] & VK_QUEUE_GRAPHICS_BIT))
```

------

### 🥈 Step 2：优先寻找**专用的传输队列**

类似逻辑：

```
cpp



if ((queueFlags & VK_QUEUE_TRANSFER_BIT) == queueFlags)
```

去找“**只支持传输、不支持图形和计算**”的队列，减少主队列负载：

```
cpp



if (transfer-only && !compute && !graphics) return index;
```

------

### 🥉 Step 3：兜底——只要能满足需求就行

```
for (...) {
    if ((queueFlags[i] & queueFlags) == queueFlags)
        return i;
}
```

只要这个队列族支持你请求的全部标志，就可以返回了，即使它也支持其他类型（如图形+计算+传输）。

------

### ❌ Step 4：如果都找不到，就抛出错误

```
cpp



throw std::runtime_error("Could not find a matching queue family index");
```

------

## 📌 示例：查找图形队列

```
cpp



uint32_t graphicsQueueIndex = getQueueFamilyIndex(VK_QUEUE_GRAPHICS_BIT);
```

会走到第 3 步，返回第一个支持图形队列的 family index。

------

## 🧠 总结成流程图：

```
textgetQueueFamilyIndex(queueFlags):
├─ 如果只需要 compute：
│   └─ 找只支持 compute 的队列族
├─ 如果只需要 transfer：
│   └─ 找只支持 transfer 的队列族
├─ 否则：
│   └─ 找第一个满足所有请求标志的队列族
└─ 如果都没有 → 抛异常
```













## 📌 `VkDeviceQueueCreateInfo` 是什么？

> 它告诉 Vulkan：
>
> > “我要从物理设备的哪个 **队列族（Queue Family）** 中创建几个 **队列（Queue）**，优先级是多少”。

------

## 🧱 结构体字段详细解释：

```
typedef struct VkDeviceQueueCreateInfo {
    VkStructureType             sType;               // 必填，必须是 VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO
    const void*                 pNext;               // 指向扩展结构的指针（一般为 nullptr）
    VkDeviceQueueCreateFlags    flags;               // 保留字段，通常为 0
    uint32_t                    queueFamilyIndex;    // 队列族索引（来自 vkGetPhysicalDeviceQueueFamilyProperties）
    uint32_t                    queueCount;          // 想从该队列族中创建多少个队列
    const float*                pQueuePriorities;    // 每个队列的优先级（范围 0.0~1.0）
} VkDeviceQueueCreateInfo;
```

## 🧠 `pQueuePriorities` 是干嘛用的？

- 它是一个 **`float\*` 指针**，指向一个数组，数组长度是 `queueCount`

- 每个元素是一个 **队列的优先级值**

- 优先级范围是：

  ```
  cpp
  
  
  
  0.0f（最低优先级） ～ 1.0f（最高优先级）
  ```

> ✅ 优先级影响同族队列之间的调度“顺序”，**但不会影响不同 Queue Family 之间的调度**。

------

## 🔍 举个实际例子：

你要从 **同一个队列族** 中创建两个队列，一个用来干活（图形），一个做后台上传（传输），你可以这么写：

```
float queuePriorities[] = { 1.0f, 0.5f };  // 第一个高优先级，第二个低

VkDeviceQueueCreateInfo queueCreateInfo{};
queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
queueCreateInfo.queueFamilyIndex = myQueueFamilyIndex;
queueCreateInfo.queueCount = 2;
queueCreateInfo.pQueuePriorities = queuePriorities;
```

这样：

- `queue[0]` 会比 `queue[1]` 更早被 GPU 调度
- 如果两个都满负荷运行，会更优先处理 `queue[0]` 提交的命令

------

## 🚩 注意事项：



| 项目       | 说明                                                       |
| ---------- | ---------------------------------------------------------- |
| 必须设置   | 每个创建的队列必须设置优先级，即使只有 1 个                |
| 范围限制   | Vulkan 要求值在 `[0.0f, 1.0f]` 之间（一般都设置为 `1.0f`） |
| 不影响跨族 | 不同 Queue Family 之间的调度独立，优先级不相关             |







## 🧩 分析逐行代码含义：

```
VkBool32 validFormat{ false };
```

初始化一个标志位，表示是否找到合适的格式。

```
if (requiresStencil) {
    validFormat = vks::tools::getSupportedDepthStencilFormat(physicalDevice, &depthFormat);
} else {
    validFormat = vks::tools::getSupportedDepthFormat(physicalDevice, &depthFormat);
}
```

这段代码是 Vulkan 中用于选择一个 **合适的深度格式（depth format）或深度 + 模板格式（depth-stencil format）** 的逻辑，在设置 `RenderPass`、`Framebuffer` 或创建 `Depth Attachment` 时非常关键。

------

## 🧠 整体功能概括：

```
// 查找一个当前物理设备（physicalDevice）支持的深度格式
// 如果用到了 stencil（模板缓冲），就查找支持 depth + stencil 的格式
```

- 根据是否需要 stencil（模板缓冲）调用不同函数：
  - `getSupportedDepthFormat()`：查找**只包含深度**的格式（如 `VK_FORMAT_D32_SFLOAT`）
  - `getSupportedDepthStencilFormat()`：查找**同时支持深度+模板**的格式（如 `VK_FORMAT_D24_UNORM_S8_UINT`）

这些函数会：

- 遍历常见格式列表
- 用 `vkGetPhysicalDeviceFormatProperties()` 检查哪些格式支持 `VK_IMAGE_TILING_OPTIMAL` 和 `VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT`
- 成功时，将 `depthFormat` 设置为合适格式，并返回 `true`

```
assert(validFormat);
```

断言判断：如果没找到支持的格式，直接报错终止（开发时很实用）。

------

## ✅ 常见可选格式列表（Vulkan 常用）：



| 格式                           | 类型      | 位数说明           | 备注             |
| ------------------------------ | --------- | ------------------ | ---------------- |
| `VK_FORMAT_D32_SFLOAT`         | 深度      | 32位浮点深度       | 最常用           |
| `VK_FORMAT_D24_UNORM_S8_UINT`  | 深度+模板 | 24位深度 + 8位模板 | 用于 stencil     |
| `VK_FORMAT_D16_UNORM`          | 深度      | 16位               | 性能优先但精度低 |
| `VK_FORMAT_D32_SFLOAT_S8_UINT` | 深度+模板 | 32位深度 + 8位模板 | 高精度场景       |

------

## 📌 为什么不能硬编码？

不同的 GPU 支持的格式不完全一样：



| GPU 品牌 | 支持格式可能不同      |
| -------- | --------------------- |
| NVIDIA   | 一般支持所有          |
| AMD      | 通常支持 D32、D24S8   |
| 移动 GPU | 有的只支持 D16 或 D24 |

👉 所以必须用 `vkGetPhysicalDeviceFormatProperties()` 来**动态判断**当前设备支持哪些格式。









## ✅ 结构体：

```
typedef struct VkFormatProperties {
    VkFormatFeatureFlags    linearTilingFeatures;
    VkFormatFeatureFlags    optimalTilingFeatures;
    VkFormatFeatureFlags    bufferFeatures;
} VkFormatProperties;
```

------

## 🧠 它是干嘛的？

`VkFormatProperties` 是 Vulkan 中用来描述一个像素格式（`VkFormat`）在当前物理设备上支持哪些特性（feature）的结构体。

你通过调用：

```
vkGetPhysicalDeviceFormatProperties(VkPhysicalDevice physicalDevice, VkFormat format, VkFormatProperties* pProperties);
```

来获取 GPU 是否支持某个 `VkFormat` 在不同使用场景下的功能。

------

## 🧩 成员字段详解



| 字段名                  | 类型                   | 含义                                                         |
| ----------------------- | ---------------------- | ------------------------------------------------------------ |
| `linearTilingFeatures`  | `VkFormatFeatureFlags` | 当前 format 在 `VK_IMAGE_TILING_LINEAR` 下支持的功能（如采样、颜色附件、blit） |
| `optimalTilingFeatures` | `VkFormatFeatureFlags` | 当前 format 在 `VK_IMAGE_TILING_OPTIMAL` 下支持的功能（最常用） |
| `bufferFeatures`        | `VkFormatFeatureFlags` | 当前 format 在 `buffer` 使用场景（如 `texel buffer`）下的支持特性 |

------

`VkFormatProperties` 结构体用于描述特定图像格式（`VkFormat`）的特性，具体来说，这三个成员变量的含义如下：

1. **`linearTilingFeatures`**:
    这是一个 `VkFormatFeatureFlags` 类型的标志，表示该格式在 "线性"（`VK_IMAGE_TILING_LINEAR`）排布方式下的支持特性。线性排列的图像内存通常用于直接访问（如 CPU 访问），因此它会列出该格式是否支持的功能，如是否支持纹理采样、颜色存储等。
2. **`optimalTilingFeatures`**:
    这是一个 `VkFormatFeatureFlags` 类型的标志，表示该格式在 "最优"（`VK_IMAGE_TILING_OPTIMAL`）排布方式下的支持特性。最优排列方式通常用于 GPU 访问，并且是 Vulkan 中用于纹理、渲染目标等的默认排布。这个成员列出了该格式在最优排列下支持的特性。
3. **`bufferFeatures`**:
    这是一个 `VkFormatFeatureFlags` 类型的标志，表示该格式在缓冲区（`VK_IMAGE_TYPE_1D` 或 `VK_IMAGE_TYPE_2D`）中使用时的支持特性。与图像相关的格式在缓冲区内使用时，可能会有一些特定的功能支持，比如数据拷贝、纹理采样等。

总结来说，这些成员变量描述了特定图像格式在不同内存排布方式下（线性与最优）以及在缓冲区内的特性支持情况。它们通常用于在创建图像时帮助选择适当的格式和布局。

## ✅ 常见用途：判断 format 是否能作为附件或采样

### 判断格式是否支持作为深度附件：

```
VkFormatProperties props;
vkGetPhysicalDeviceFormatProperties(physicalDevice, VK_FORMAT_D32_SFLOAT, &props);

if (props.optimalTilingFeatures & VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT) {
    // 这个 format 可以作为 depth/stencil 用
}
```

------

### 判断格式是否可以被采样：

```
if (props.optimalTilingFeatures & VK_FORMAT_FEATURE_SAMPLED_IMAGE_BIT) {
    // 可以作为采样图像使用
}
```

------

## 🧾 常见 `VkFormatFeatureFlagBits` 值（部分）



| 宏名                                             | 意义                           |
| ------------------------------------------------ | ------------------------------ |
| `VK_FORMAT_FEATURE_SAMPLED_IMAGE_BIT`            | 可以被着色器采样               |
| `VK_FORMAT_FEATURE_COLOR_ATTACHMENT_BIT`         | 可以作为 color attachment 使用 |
| `VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT` | 可作为 depth/stencil           |
| `VK_FORMAT_FEATURE_STORAGE_IMAGE_BIT`            | 可作为 image store（写入）使用 |
| `VK_FORMAT_FEATURE_BLIT_SRC_BIT`                 | 可以作为 blit 源               |
| `VK_FORMAT_FEATURE_BLIT_DST_BIT`                 | 可以作为 blit 目标             |

👉 这些值用于 `linearTilingFeatures` 和 `optimalTilingFeatures`。

------

## ✅ 用途总结



| 你想干嘛              | 应该查哪个字段                                               |
| --------------------- | ------------------------------------------------------------ |
| 图片用作贴图          | `optimalTilingFeatures` 是否包含 `SAMPLED_IMAGE_BIT`         |
| 图片作为颜色/深度输出 | 是否有 `COLOR_ATTACHMENT_BIT` 或 `DEPTH_STENCIL_ATTACHMENT_BIT` |
| 用 buffer 加载数据    | 查 `bufferFeatures` 是否支持 `UNIFORM_TEXEL_BUFFER_BIT` 等   |





## 创建信号量

```
vkCreateSemaphore(...) → 创建信号量
```

代码逻辑创建了两个信号量：



| 信号量名                     | 用途                               |
| ---------------------------- | ---------------------------------- |
| `semaphores.presentComplete` | **等待图像准备完成**，才能开始渲染 |
| `semaphores.renderComplete`  | **等待渲染完成**，才能开始呈现图像 |

------

## 🧩 分析每一行

### ✅ 创建信号量的结构体初始化

```

VkSemaphoreCreateInfo semaphoreCreateInfo = vks::initializers::semaphoreCreateInfo();
```

这使用了 Sascha Willems 的工具函数封装，实际上是：

```
cppVkSemaphoreCreateInfo semaphoreCreateInfo{};
semaphoreCreateInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;
semaphoreCreateInfo.pNext = nullptr;
semaphoreCreateInfo.flags = 0;
```

用于初始化一个空的标准信号量。

------

### ✅ 创建用于图像获取同步的信号量

```
vkCreateSemaphore(device, &semaphoreCreateInfo, nullptr, &semaphores.presentComplete);
```

- 用于在 `vkAcquireNextImageKHR()` 后发出信号
- 让后续的 `vkQueueSubmit()` 等待它触发再开始工作
- 确保我们拿到了可以渲染的图像

------

### ✅ 创建用于渲染完成 → 呈现图像的信号量

```

vkCreateSemaphore(device, &semaphoreCreateInfo, nullptr, &semaphores.renderComplete);
```

- 当 command buffer 渲染完成后，触发这个信号量
- 作为 `vkQueuePresentKHR()` 的等待条件
- 确保在 GPU 渲染完成后才进行图像显示

------

## 🔁 典型的信号量使用流程（每帧）

```
cpp// 1. 获取图像
vkAcquireNextImageKHR(swapchain, ..., semaphores.presentComplete, ...);

// 2. 提交渲染命令（等待图像可用）
vkQueueSubmit(queue, ..., waitSemaphore = presentComplete, signalSemaphore = renderComplete);

// 3. 呈现图像（等待渲染完成）
vkQueuePresentKHR(queue, waitSemaphore = renderComplete);
```

------

## 🎯 信号量 vs 栅栏（Fence）



| 对象        | 作用      | 阻塞行为             | 用于谁等待谁               |
| ----------- | --------- | -------------------- | -------------------------- |
| `Semaphore` | GPU ↔ GPU | 非阻塞（GPU 内部）   | 渲染等待图像、显示等待渲染 |
| `Fence`     | CPU ↔ GPU | 可阻塞（CPU 等 GPU） | 程序等待 GPU 任务完成      |

------

## ✅ 总结



| 变量名            | 类型                  | 用途                     |
| ----------------- | --------------------- | ------------------------ |
| `presentComplete` | `VkSemaphore`         | 等待“图像可用”再开始渲染 |
| `renderComplete`  | `VkSemaphore`         | 等待“渲染完成”再进行呈现 |
| 创建函数          | `vkCreateSemaphore()` | 初始化同步信号量         |

🔗 它们在每一帧的渲染流程中被重复使用，确保 GPU 指令按顺序执行、图像不会被覆盖或提前显示。









## 总体作用概括

> 设置一次提交命令的同步依赖，包括：
>
> ✅ 需要等待的信号量（图像准备完）
>  ✅ GPU 的执行阶段（比如顶点着色器阶段）
>  ✅ 渲染完成后要发出的信号量（通知可以开始呈现）
>  ✅ 要提交的命令缓冲（稍后会设置）

------

## 🧩 分析逐行代码含义

------

### ✅ 初始化 `VkSubmitInfo`

```

submitInfo = vks::initializers::submitInfo();
```

调用封装的初始化函数，实际效果等同于：

```
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
```

------

### ✅ 设置等待阶段（同步点）

```

submitInfo.pWaitDstStageMask = &submitPipelineStages;
```

- `submitPipelineStages` 通常是：

```

VkPipelineStageFlags submitPipelineStages = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
```

- 表示：当我们等到了 `presentComplete` 信号量，**从 color attachment 输出阶段**开始执行这次渲染提交。
- 它描述了 **信号量触发后，要在哪个阶段真正开始执行命令**

------

### ✅ 设置等待的信号量

```
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = &semaphores.presentComplete;
```

- 表示这次提交的命令缓冲需要 **等待 `presentComplete` 这个信号量被触发**
- 也就是说，**只有图像从交换链中获取成功，我们才开始执行命令缓冲**

------

### ✅ 设置要发出的信号量

```
submitInfo.signalSemaphoreCount = 1;
submitInfo.pSignalSemaphores = &semaphores.renderComplete;
```

- 表示：当这次提交的所有命令执行完成，**GPU 会触发 `renderComplete` 信号量**
- 通常在 `vkQueuePresentKHR()` 之前等待这个信号量，确保图像已经渲染完了

------

## ✅ 后续配合命令缓冲提交使用：

```
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffer;

vkQueueSubmit(graphicsQueue, 1, &submitInfo, inFlightFence);
```

------

## 🎯 时序关系图（简化）：

```
text[ acquire image ] --> presentComplete(semaphore) --> 
[ submit render cmd ] --> renderComplete(semaphore) -->
[ present image ]
```

------

## 🔍 总结字段含义对照表



| 字段名              | 类型           | 说明                                              |
| ------------------- | -------------- | ------------------------------------------------- |
| `pWaitSemaphores`   | 信号量指针数组 | 提交前必须等待的信号（例如图像准备完成）          |
| `pWaitDstStageMask` | 执行阶段数组   | 上面信号等待在哪个阶段解除（通常是 color output） |
| `pCommandBuffers`   | 命令缓冲数组   | 要提交给 GPU 执行的工作                           |
| `pSignalSemaphores` | 信号量指针数组 | GPU 执行完后要触发的信号（通常是通知可呈现）      |

------

## ✅ 如果你希望做多帧并发（Frame in Flight）

通常这个结构会被多个帧资源共用，每帧绑定不同的信号量和命令缓冲。

如果你需要，我可以给你写一个完整的多帧 `submitInfo[]` 管理结构，也可以画张图表示每帧信号量的流转结构。





## 结构体是 Vulkan 的核心结构之一：

```
typedef struct VkSubmitInfo {
    VkStructureType                sType;
    const void*                    pNext;
    uint32_t                       waitSemaphoreCount;
    const VkSemaphore*             pWaitSemaphores;
    const VkPipelineStageFlags*    pWaitDstStageMask;
    uint32_t                       commandBufferCount;
    const VkCommandBuffer*         pCommandBuffers;
    uint32_t                       signalSemaphoreCount;
    const VkSemaphore*             pSignalSemaphores;
} VkSubmitInfo;
```

------

## 🧠 作用：向 GPU 提交一次指令执行请求（`vkQueueSubmit()`）

这个结构体是你在调用：

```
vkQueueSubmit(queue, 1, &submitInfo, fence);
```

时必须填写的参数，它描述了：

> “**这次 GPU 执行任务需要等谁、做什么、执行完通知谁**”

------

## 📦 字段详解



| 字段名                 | 类型                          | 说明                                                    |
| ---------------------- | ----------------------------- | ------------------------------------------------------- |
| `sType`                | `VkStructureType`             | 必须是 `VK_STRUCTURE_TYPE_SUBMIT_INFO`                  |
| `pNext`                | `const void*`                 | 扩展链，常为 `nullptr`                                  |
| `waitSemaphoreCount`   | `uint32_t`                    | 需要等待的信号量数量                                    |
| `pWaitSemaphores`      | `const VkSemaphore*`          | 指向需要等待的信号量数组（如 `presentComplete`）        |
| `pWaitDstStageMask`    | `const VkPipelineStageFlags*` | 每个等待的信号对应在哪个管线阶段解除等待                |
| `commandBufferCount`   | `uint32_t`                    | 要执行的命令缓冲数量                                    |
| `pCommandBuffers`      | `const VkCommandBuffer*`      | 要提交给 GPU 执行的命令缓冲数组                         |
| `signalSemaphoreCount` | `uint32_t`                    | 提交完成后要触发的信号量数量                            |
| `pSignalSemaphores`    | `const VkSemaphore*`          | 要在命令执行完后发出的信号量数组（如 `renderComplete`） |

------

## ✅ 图解说明（单帧渲染典型使用流程）

```
text[ CPU 调用 vkAcquireNextImageKHR ]
                ↓
         presentComplete 信号量触发
                ↓ (waitSemaphore)
         vkQueueSubmit 开始执行 commandBuffer
                ↓
        renderComplete 信号量触发
                ↓
         vkQueuePresentKHR 等待 renderComplete
                ↓
            图像显示
```

------

## ✅ 示例代码

```
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };

VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = &presentCompleteSemaphore;
submitInfo.pWaitDstStageMask = waitStages;
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &drawCommandBuffer;
submitInfo.signalSemaphoreCount = 1;
submitInfo.pSignalSemaphores = &renderCompleteSemaphore;

vkQueueSubmit(graphicsQueue, 1, &submitInfo, renderFence);
```

------

## 🎯 总结记忆小口诀：

> **“先等谁，再做什么，做完叫谁”**



| 步骤             | 对应字段            | 例子                      |
| ---------------- | ------------------- | ------------------------- |
| 等待图像准备     | `pWaitSemaphores`   | `presentComplete`         |
| 指定等待解除时机 | `pWaitDstStageMask` | `COLOR_ATTACHMENT_OUTPUT` |
| 提交命令         | `pCommandBuffers`   | 渲染指令                  |
| 触发后续信号     | `pSignalSemaphores` | `renderComplete`          |







## 1. **`WndProc`：窗口过程函数（Window Procedure）**

### 🧠 `WndProc` 是干什么的？

在 Windows API 中，窗口过程函数（`WndProc`）是一个处理 **窗口消息（如键盘输入、鼠标移动、窗口关闭等）** 的回调函数。每当窗口需要响应某个事件时，Windows 会调用 `WndProc` 来处理消息。

### 代码解析：

```
cpp复制编辑LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    if (vulkanExample != NULL)
    {
        vulkanExample->handleMessages(hWnd, uMsg, wParam, lParam);
    }
    return (DefWindowProc(hWnd, uMsg, wParam, lParam));
}
```

- `WndProc` 是**窗口消息的回调函数**，当用户与窗口交互时，Windows 系统会发送不同的消息（例如，按键、鼠标点击等），这些消息会传递给 `WndProc` 函数。
- `hWnd`：窗口的句柄，标识窗口。
- `uMsg`：消息类型，指示收到的消息（如 `WM_KEYDOWN` 表示键盘按下，`WM_DESTROY` 表示窗口销毁等）。
- `wParam` 和 `lParam`：额外的消息参数，具体内容依赖于 `uMsg` 类型。

### 主要逻辑：

- 如果 `vulkanExample` 不为空（即有效的 Vulkan 示例对象），`WndProc` 会将消息转发给 `vulkanExample->handleMessages()`，这个方法处理 Vulkan 的相关消息。
- `DefWindowProc()` 是默认的 Windows 消息处理函数，在消息没有被处理时调用，处理默认行为（如关闭窗口）。

------

## 2. **`vulkanExample->setupWindow(hInstance, WndProc)`**

### 🧠 这是干什么的？

`setupWindow` 是一个自定义的成员函数（假设属于 `VulkanExample` 类），它负责 **初始化和创建与 Vulkan 渲染相关的窗口**，并将上面提到的 `WndProc` 作为窗口消息处理函数。

### 代码解析：

```
cpp


复制编辑
vulkanExample->setupWindow(hInstance, WndProc);
```

- `hInstance`：Windows 应用程序的实例句柄。它在 `WinMain()` 中初始化，并且被传递给每个窗口相关的 API 函数。
- `WndProc`：窗口消息处理函数（如上所述）。`setupWindow` 会把它注册到 Windows 系统，作为窗口事件的处理回调。

------

### 主要步骤：

1. **注册窗口类**： `WndProc` 是作为窗口过程函数（Window Procedure）注册到 Windows 系统的。Windows 会调用 `WndProc` 来处理所有窗口事件。
2. **创建窗口**： `setupWindow` 会调用 Windows API 创建一个实际的窗口，并将 `WndProc` 作为该窗口的消息处理函数。
3. **与 Vulkan 连接**： 在 `setupWindow` 中，Vulkan 可能会进行与窗口相关的设置，例如：
   - 创建 `VkSurfaceKHR`（Vulkan 用来渲染到窗口的对象）
   - 将窗口和 Vulkan 渲染上下文关联

------

## 🧠 总结：

### `WndProc`：

- **作用**：处理 Windows 窗口的消息（输入、关闭、调整大小等）。
- **回调函数**：由 Windows 操作系统调用，当窗口接收到事件时，调用 `WndProc` 来处理。

### `vulkanExample->setupWindow(hInstance, WndProc)`：

- **作用**：初始化窗口并将 `WndProc` 绑定为窗口事件的处理函数，同时为 Vulkan 渲染准备相关设置（如 `VkSurfaceKHR`）。
- **功能**：这一步是 Vulkan 渲染应用在 Windows 上的基础，它确保窗口事件与 Vulkan 渲染同步。















## 🧠 setupWindow` 函数目的

`setupWindow` 函数的主要目标是：

- **注册窗口类**：通过 Windows API `RegisterClassEx` 注册一个窗口类。
- **创建窗口**：使用 `CreateWindowEx` 创建一个窗口，并根据设置选择是否为全屏模式。
- **设置窗口样式和位置**：根据是否启用全屏模式来调整窗口的大小和显示。

------

## 🧩 逐行代码解析

### 1. **设置 `WNDCLASSEX` 结构体**

```
cpp复制编辑WNDCLASSEX wndClass{};
wndClass.cbSize = sizeof(WNDCLASSEX);
wndClass.style = CS_HREDRAW | CS_VREDRAW;
wndClass.lpfnWndProc = wndproc;
wndClass.cbClsExtra = 0;
wndClass.cbWndExtra = 0;
wndClass.hInstance = hinstance;
wndClass.hIcon = LoadIcon(NULL, IDI_APPLICATION);
wndClass.hCursor = LoadCursor(NULL, IDC_ARROW);
wndClass.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
wndClass.lpszMenuName = NULL;
wndClass.lpszClassName = name.c_str();
wndClass.hIconSm = LoadIcon(NULL, IDI_WINLOGO);
```

- `wndClass` 是 **窗口类（Window Class）** 的配置结构。
- `style`：指定窗口的样式，`CS_HREDRAW` 和 `CS_VREDRAW` 表示窗口大小或位置改变时重绘窗口。
- `lpfnWndProc`：窗口过程函数（即 `WndProc`），用于处理窗口消息。
- `hInstance`：应用程序实例句柄。
- `hIcon`、`hCursor`、`hbrBackground` 等：分别设置窗口图标、光标、背景色等。

### 2. **注册窗口类**

```
cpp复制编辑if (!RegisterClassEx(&wndClass))
{
    std::cout << "Could not register window class!\n";
    fflush(stdout);
    exit(1);
}
```

- 使用 `RegisterClassEx` 注册窗口类。如果注册失败，程序会输出错误信息并退出。

### 3. **获取屏幕分辨率**

```
cpp复制编辑int screenWidth = GetSystemMetrics(SM_CXSCREEN);
int screenHeight = GetSystemMetrics(SM_CYSCREEN);
```

- 获取当前屏幕的宽度和高度，供后续使用。

### 4. **设置全屏模式（如果需要）**

```
cpp复制编辑if (settings.fullscreen)
{
    if ((width != (uint32_t)screenWidth) && (height != (uint32_t)screenHeight))
    {
        DEVMODE dmScreenSettings;
        memset(&dmScreenSettings, 0, sizeof(dmScreenSettings));
        dmScreenSettings.dmSize       = sizeof(dmScreenSettings);
        dmScreenSettings.dmPelsWidth  = width;
        dmScreenSettings.dmPelsHeight = height;
        dmScreenSettings.dmBitsPerPel = 32;
        dmScreenSettings.dmFields     = DM_BITSPERPEL | DM_PELSWIDTH | DM_PELSHEIGHT;
        if (ChangeDisplaySettings(&dmScreenSettings, CDS_FULLSCREEN) != DISP_CHANGE_SUCCESSFUL)
        {
            if (MessageBox(NULL, "Fullscreen Mode not supported!\n Switch to window mode?", "Error", MB_YESNO | MB_ICONEXCLAMATION) == IDYES)
            {
                settings.fullscreen = false;
            }
            else
            {
                return nullptr;
            }
        }
        screenWidth = width;
        screenHeight = height;
    }
}
```

- 如果设置了 `settings.fullscreen` 为 `true`，则尝试将显示模式切换为全屏。若切换失败，则弹出消息框询问用户是否退出全屏。
- 如果全屏模式设置成功，更新 `screenWidth` 和 `screenHeight`。

### 5. **设置窗口样式**

```
cpp复制编辑DWORD dwExStyle;
DWORD dwStyle;

if (settings.fullscreen)
{
    dwExStyle = WS_EX_APPWINDOW;
    dwStyle = WS_POPUP | WS_CLIPSIBLINGS | WS_CLIPCHILDREN;
}
else
{
    dwExStyle = WS_EX_APPWINDOW | WS_EX_WINDOWEDGE;
    dwStyle = WS_OVERLAPPEDWINDOW | WS_CLIPSIBLINGS | WS_CLIPCHILDREN;
}
```

- 根据是否启用全屏，设置不同的窗口扩展样式和常规样式。
  - **全屏**：`WS_POPUP` 样式，去掉窗口边框。
  - **窗口模式**：`WS_OVERLAPPEDWINDOW` 样式，常规窗口样式。

### 6. **调整窗口大小和位置**

```
cpp复制编辑RECT windowRect = {
    0L,
    0L,
    settings.fullscreen ? (long)screenWidth : (long)width,
    settings.fullscreen ? (long)screenHeight : (long)height
};

AdjustWindowRectEx(&windowRect, dwStyle, FALSE, dwExStyle);
```

- 根据设置的 `width` 和 `height`（或全屏尺寸）调整窗口大小。
- 使用 `AdjustWindowRectEx` 确保窗口大小和样式匹配，计算最终的窗口尺寸。

### 7. **创建窗口**

```
cpp复制编辑window = CreateWindowEx(0,
    name.c_str(),
    windowTitle.c_str(),
    dwStyle | WS_CLIPSIBLINGS | WS_CLIPCHILDREN,
    0,
    0,
    windowRect.right - windowRect.left,
    windowRect.bottom - windowRect.top,
    NULL,
    NULL,
    hinstance,
    NULL);
```

- 使用 `CreateWindowEx` 创建窗口，指定窗口样式、标题、大小、位置等参数。
- `name.c_str()` 是窗口类名，`windowTitle.c_str()` 是窗口标题。

### 8. **窗口创建失败时处理**

```
cpp复制编辑if (!window)
{
    std::cerr << "Could not create window!\n";
    fflush(stdout);
    return nullptr;
}
```

- 如果创建窗口失败，输出错误信息并返回 `nullptr`。

### 9. **窗口显示**

```
cpp复制编辑if (!settings.fullscreen)
{
    uint32_t x = (GetSystemMetrics(SM_CXSCREEN) - windowRect.right) / 2;
    uint32_t y = (GetSystemMetrics(SM_CYSCREEN) - windowRect.bottom) / 2;
    SetWindowPos(window, 0, x, y, 0, 0, SWP_NOZORDER | SWP_NOSIZE);
}

ShowWindow(window, SW_SHOW);
SetForegroundWindow(window);
SetFocus(window);
```

- 如果窗口不是全屏，**居中显示窗口**。
- 调用 `ShowWindow` 来显示窗口，`SetForegroundWindow` 将窗口设置为前景窗口，`SetFocus` 将焦点设置到窗口。

### 10. **返回窗口句柄**

```
cpp


复制编辑
return window;
```

- 最后返回创建的窗口句柄 `window`。

------

## 🧠 总结

`setupWindow` 函数完成了以下工作：

1. **注册窗口类**：通过 `RegisterClassEx` 注册窗口类，设置窗口的样式、图标、光标等。
2. **创建窗口**：使用 `CreateWindowEx` 创建窗口，设置窗口的样式和大小。
3. **全屏支持**：根据 `settings.fullscreen` 设置窗口为全屏或窗口模式，调整显示模式（如果需要）。
4. **窗口显示和焦点管理**：调用 `ShowWindow` 等函数显示窗口，并使其获得焦点。
5. **返回窗口句柄**：返回创建的窗口句柄 `HWND`。









### 🧩 `VulkanExampleBase::prepare()` 和 `prepare()` 函数的工作原理

#### 1. **`VulkanExampleBase::prepare()`** — 基础资源初始化

```
cpp复制编辑void VulkanExampleBase::prepare()
{
    createSurface();                  // 创建 Vulkan 窗口表面（VkSurfaceKHR）
    createCommandPool();              // 创建命令池（Command Pool）
    createSwapChain();                // 创建交换链（Swapchain）
    createCommandBuffers();           // 创建命令缓冲区（Command Buffers）
    createSynchronizationPrimitives(); // 创建同步对象（信号量和栅栏）
    setupDepthStencil();              // 设置深度和模板缓冲区
    setupRenderPass();                // 设置渲染过程（Render Pass）
    createPipelineCache();            // 创建管线缓存（Pipeline Cache）
    setupFrameBuffer();               // 设置帧缓冲（Framebuffer）
}
```

- `createSurface()`：在窗口系统中创建 Vulkan 表面（`VkSurfaceKHR`），用于将渲染结果呈现到屏幕上。
- `createCommandPool()`：创建命令池（`VkCommandPool`），用于存放命令缓冲区（`VkCommandBuffer`）。
- `createSwapChain()`：创建交换链（`VkSwapchainKHR`），负责管理呈现图像的交换过程。
- `createCommandBuffers()`：创建命令缓冲区（`VkCommandBuffer`），这些缓冲区包含要执行的渲染命令。
- `createSynchronizationPrimitives()`：创建同步原语，如信号量（`VkSemaphore`）和栅栏（`VkFence`），用于任务同步。
- `setupDepthStencil()`：配置深度和模板缓冲区，通常用于处理 3D 渲染中的深度测试。
- `setupRenderPass()`：设置渲染过程（`VkRenderPass`），指定渲染目标和各种渲染操作。
- `createPipelineCache()`：创建管线缓存（`VkPipelineCache`），用于存储管线创建过程中的中间状态。
- `setupFrameBuffer()`：配置帧缓冲（`VkFramebuffer`），将交换链图像与渲染过程结合。

------

#### 2. **`prepare()`（重写）** — 增加自定义资源和管线设置

```
cpp复制编辑void prepare() override
{
    VulkanExampleBase::prepare(); // 调用基类的 prepare，完成基础资源和设置
    createSynchronizationPrimitives(); // 创建同步原语
    createCommandBuffers();           // 创建命令缓冲区
    createVertexBuffer();             // 创建顶点缓冲区
    createUniformBuffers();           // 创建统一缓冲区
    createDescriptorSetLayout();      // 创建描述符集布局
    createDescriptorPool();           // 创建描述符池
    createDescriptorSets();           // 创建描述符集
    createPipelines();                // 创建渲染管线
    prepared = true;                  // 标记资源准备完成
}
```

- `VulkanExampleBase::prepare()`：首先调用基类的 `prepare()`，完成一些通用的 Vulkan 初始化操作（如交换链、命令池、渲染过程等）。
- `createSynchronizationPrimitives()`：创建同步原语（信号量、栅栏等），以确保不同任务的同步执行。
- `createCommandBuffers()`：为渲染过程创建命令缓冲区（`VkCommandBuffer`）。
- `createVertexBuffer()`：创建顶点缓冲区，存放渲染用的顶点数据。
- `createUniformBuffers()`：创建统一缓冲区，用于存放常用的渲染数据（如模型矩阵、视图矩阵、投影矩阵等）。
- `createDescriptorSetLayout()`：创建描述符集布局，指定描述符类型及其绑定方式（如 UBO、纹理等）。
- `createDescriptorPool()`：创建描述符池，管理描述符集的内存。
- `createDescriptorSets()`：创建描述符集，将资源绑定到着色器中。
- `createPipelines()`：创建渲染管线（`VkPipeline`），配置着色器、渲染状态等。
- `prepared = true;`：标记为已准备好，表示所有资源和配置已经完成，可以进入渲染循环。

------

### 🧠 `setupWindow` 与 `prepare` 之间的关系

- `setupWindow` 主要负责创建窗口和配置。这是 Vulkan 渲染过程中的基础部分。
- `prepare()` 进一步配置所有渲染所需的资源，包括命令缓冲、顶点数据、描述符集、渲染管线等。它为后续的渲染循环做好了准备。

### 🔄 `prepare()` 完整过程：从资源初始化到渲染准备

1. **资源创建**：
   - 创建 `VkCommandPool`、`VkCommandBuffer`、顶点缓冲区、统一缓冲区等。
2. **同步原语**：
   - 创建信号量和栅栏，确保任务执行顺序。
3. **渲染管线配置**：
   - 设置渲染管线、描述符集布局、描述符池、描述符集等。
4. **标记准备完成**：
   - 标记 `prepared = true`，表示所有渲染资源已经准备就绪，可以开始渲染。

------

## 🎯 总结



| 步骤           | 函数名称                                                     | 描述                                                   |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| 基础设置       | `VulkanExampleBase::prepare()`                               | 创建 Vulkan 所需基础资源（交换链、命令池、渲染过程等） |
| 同步原语       | `createSynchronizationPrimitives()`                          | 创建信号量和栅栏，确保 GPU 执行顺序                    |
| 命令缓冲       | `createCommandBuffers()`                                     | 创建渲染命令缓冲区                                     |
| 顶点和统一缓冲 | `createVertexBuffer()` / `createUniformBuffers()`            | 创建顶点数据和统一缓冲区                               |
| 描述符集       | `createDescriptorSetLayout()` / `createDescriptorPool()` / `createDescriptorSets()` | 创建描述符集布局、池和集                               |
| 渲染管线       | `createPipelines()`                                          | 创建渲染管线（着色器、光栅化状态等）                   |