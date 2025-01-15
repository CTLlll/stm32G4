如果你的文件结构是 **CubeMX 自动生成的结构**，那确实会与标准的 Keil 工程结构有所不同。以下是如何在 **CubeMX 自动生成的 Keil 项目结构** 中，从头添加 `delay.c` 和 `delay.h` 文件的详细步骤。

---

### **1. 确认 CubeMX 生成的项目文件结构**

通常，CubeMX 生成的 Keil 项目会有一个类似以下的文件结构：

```
Project_Folder/
├── Core/
│   ├── Inc/
│   │   ├── main.h
│   │   ├── stm32xxxx_hal.h
│   │   ├── other HAL headers...
│   ├── Src/
│   │   ├── main.c
│   │   ├── stm32xxxx_hal_msp.c
│   │   ├── other HAL source files...
├── Drivers/
│   ├── CMSIS/
│   │   ├── Device/
│   │   ├── Include/
│   ├── STM32xxxx_HAL_Driver/
│   │   ├── Inc/
│   │   ├── Src/
```

- **`Core/Inc/`**：放置用户头文件。
- **`Core/Src/`**：放置用户源文件。
- **`Drivers/`**：放置 HAL 和 CMSIS 驱动代码。

---

### **2. 添加 `delay.c` 和 `delay.h` 文件**

#### **2.1 创建文件**
在 CubeMX 生成的项目中：

1. 在 `Core/Inc/` 文件夹中创建一个新的头文件，命名为 `delay.h`。
2. 在 `Core/Src/` 文件夹中创建一个新的源文件，命名为 `delay.c`。

#### **2.2 文件结构示例**
你的文件结构现在应该是这样的：

```
Project_Folder/
├── Core/
│   ├── Inc/
│   │   ├── delay.h   // 新建的头文件
│   │   ├── main.h
│   ├── Src/
│   │   ├── delay.c   // 新建的源文件
│   │   ├── main.c
```

---

### **3. 编写 `delay.h` 和 `delay.c` 文件的内容**

#### **3.1 `delay.h` 文件内容**
在 `Core/Inc/delay.h` 中写入以下内容：

```c
#ifndef __DELAY_H
#define __DELAY_H

#include "stm32g4xx_hal.h" // 替换为你的 HAL 头文件

// 函数声明
void DWT_Init(void);        // 初始化 DWT 计数器
void Delay_us(uint32_t us); // 微秒级延时函数

#endif // __DELAY_H
```

#### **3.2 `delay.c` 文件内容**
在 `Core/Src/delay.c` 中写入以下内容：

```c
#include "delay.h"

void DWT_Init(void) {
    // 启用 DWT 寄存器
    CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
    // 启用 CYCCNT 计数器
    DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
    // 清零计数器
    DWT->CYCCNT = 0;
}

void Delay_us(uint32_t us) {
    // 获取当前计数器值
    uint32_t start = DWT->CYCCNT;
    // 计算所需的时钟周期数
    uint32_t ticks = us * (SystemCoreClock / 1000000);

    // 等待计数器达到目标值
    while ((DWT->CYCCNT - start) < ticks);
}
```

---

### **4. 将 `delay.c` 文件加入 Keil 工程**

1. **打开 Keil 工程**：
   打开 Keil uVision 并加载 CubeMX 生成的项目。

2. **右键添加文件**：
   - 在左侧的 **Project Explorer** 中，右键点击 `Core/Src` 分组。
   - 选择 **Add Existing Files to Group 'Core/Src'**。

3. **选择文件**：
   - 在弹出的文件浏览器中，找到 `Core/Src/delay.c` 文件。
   - 点击 **Add**，然后点击 **Close**。

4. **确认文件已加入**：
   - 在 **Project Explorer** 中，你应该能看到 `delay.c` 文件已经被添加到 `Core/Src` 分组中。

---

### **5. 配置头文件路径**

为了让 Keil 编译器找到 `delay.h` 文件，需要在工程中添加头文件路径：

1. **打开项目设置**：
   - 在 Keil 菜单栏中，点击 **Project** -> **Options for Target**。

2. **设置 Include Paths**：
   - 在弹出的窗口中，选择 **C/C++** 选项卡。
   - 在 **Include Paths** 输入框中，添加以下路径：
     ```
     .\Core\Inc
     ```
   - 点击右侧的文件夹图标，浏览到 `Core/Inc` 文件夹，确认添加后点击 **OK**。

---

### **6. 在 `main.c` 中使用延时功能**

#### **6.1 引入头文件**
在 `Core/Src/main.c` 中包含 `delay.h`：

```c
#include "delay.h"
```

#### **6.2 调用延时函数**
在主函数中初始化 DWT 并调用 `Delay_us()`：

```c
int main(void) {
    HAL_Init();           // 初始化 HAL 库
    SystemClock_Config(); // 配置系统时钟

    DWT_Init();           // 初始化 DWT 计数器

    while (1) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5); // 翻转 GPIO 引脚
        Delay_us(500);                         // 延时 500 微秒
    }
}
```

---

### **7. 编译和调试**

1. 点击 **Rebuild** 按钮（快捷键 `F7`）重新生成工程。
2. 如果没有错误，烧录程序到 STM32 开发板并运行。
3. 使用示波器或逻辑分析仪观察 GPIO 引脚信号，验证 `Delay_us()` 函数的效果。

---

### **8. 常见问题及解决**

#### **问题 1：`No such file or directory 'delay.h'`**
- 确认 `delay.h` 文件位于 `Core/Inc` 文件夹中。
- 确认 Keil 的 Include Paths 中包含 `.\Core\Inc`。

#### **问题 2：`undefined reference to 'DWT_Init'`**
- 确认 `delay.c` 文件已添加到 Keil 工程中。
- 确认 `delay.c` 定义了 `DWT_Init()` 和 `Delay_us()` 函数。

#### **问题 3：`SystemCoreClock` 未定义**
- 确认在 `delay.h` 中包含了 `stm32g4xx_hal.h`（或对应的 HAL 头文件）。
- 确保在 `main.c` 中调用了 `HAL_Init()` 和 `SystemClock_Config()`。

---

### **总结**

1. 在 `Core/Inc` 和 `Core/Src` 中分别创建 `delay.h` 和 `delay.c` 文件。
2. 将 `delay.c` 添加到 Keil 工程，并配置头文件路径。
3. 在 `main.c` 中调用延时函数，并验证功能。

通过这些步骤，你可以在 CubeMX 生成的 Keil 项目中，成功集成和使用 `delay.c` 和 `delay.h` 文件。如果仍有问题，请随时提问！