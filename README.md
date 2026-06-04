
 # STM32 0.96-inch OLED Display Experiment with ChatGPT

## 项目简介

本项目基于 STM32F103C8T6 和 HAL 库，实现 0.96 英寸 OLED 屏幕点亮与字符显示。OLED 使用 I2C 通信方式，本实验使用 GPIO 模拟 I2C，其中：

* OLED SDA 接 STM32 的 PA9
* OLED SCL 接 STM32 的 PA8

本项目适合 STM32 初学者学习 GPIO、模拟 I2C、OLED 驱动文件导入和 Keil 工程编译下载。

---



## 需要的硬件实物

* 0.96 英寸 OLED 屏幕，I2C 接口


---

## OLED 驱动文件说明

本实验需要使用以下 3 个文件：已经打包在仓库请自行下载

```text
OLED.c
OLED.h
OLED_Font.h
```

文件作用如下：

```text
OLED.c        OLED 屏幕驱动源文件，包含 OLED 初始化、写命令、写数据、显示字符等函数
OLED.h        OLED 驱动头文件，声明 OLED 相关函数
OLED_Font.h   OLED 字库文件，保存 ASCII 字符点阵数据
```

---

## 硬件接线

0.96 英寸 OLED 一般有 4 个引脚：

```text
VCC
GND
SCL
SDA
```

按照下面方式连接：

| OLED 引脚 | STM32F103C8T6 引脚 | 说明        |
| ------- | ---------------- | --------- |
| VCC     | 3.3V             | OLED 电源正极 |
| GND     | GND              | OLED 电源负极 |
| SCL     | PA8              | I2C 时钟线   |
| SDA     | PA9              | I2C 数据线   |

注意：

* 建议 OLED 使用 3.3V 供电。
* 如果你的 OLED 模块支持 5V，也仍然建议优先接 3.3V。
* PA8 和 PA9 在本实验中作为普通 GPIO 使用，不作为硬件 I2C 外设使用。
* 如果你启用了 USART1，PA9 默认可能是 USART1_TX，需要关闭 USART1 或更换引脚。

---

## STM32CubeMX 配置步骤




## Keli中导入 OLED 驱动文件

将文件放到工程目录中：

 

目录结构大致如下：

```text
Core/
├── Inc/
│   ├── main.h
│   ├── gpio.h
│   ├── OLED.h
│   └── OLED_Font.h
│
└── Src/
    ├── main.c
    ├── gpio.c
    └── OLED.c
```

---

## 修改 OLED.c 的引脚定义

打开 `OLED.c`，找到 SCL 和 SDA 的引脚定义位置。

如果原来的代码不是 PA8 和 PA9，需要改成下面这种形式：

```c
#define OLED_SCL_GPIO_Port GPIOA
#define OLED_SCL_Pin       GPIO_PIN_8

#define OLED_SDA_GPIO_Port GPIOA
#define OLED_SDA_Pin       GPIO_PIN_9

#define OLED_W_SCL(x) HAL_GPIO_WritePin(OLED_SCL_GPIO_Port, OLED_SCL_Pin, (x) ? GPIO_PIN_SET : GPIO_PIN_RESET)
#define OLED_W_SDA(x) HAL_GPIO_WritePin(OLED_SDA_GPIO_Port, OLED_SDA_Pin, (x) ? GPIO_PIN_SET : GPIO_PIN_RESET)
```

如果你的 `OLED.c` 里面原来写的是类似下面这种：

```c
GPIOB
GPIO_PIN_8
GPIO_PIN_9
```

就需要改成：

```c
GPIOA
GPIO_PIN_8
GPIO_PIN_9
```

也就是：

```text
SCL → PA8
SDA → PA9
```

---

## 在 Keil 中添加 OLED.c 文件

如果 `OLED.c` 没有自动加入 Keil 工程，需要手动添加。

操作步骤：

1. 打开 Keil 工程
2. 找到左侧工程目录
3. 右键点击 `Application/User/Core`
4. 选择 `Add Existing Files to Group`
5. 找到：

```text
Core/Src/OLED.c
```

6. 添加进去

如果不添加 `OLED.c`，编译时可能会出现类似错误：

```text
undefined symbol OLED_Init
undefined symbol OLED_ShowString
```

---

## main.c 示例代码

打开 `main.c`，先在文件顶部加入头文件：

```c
#include "OLED.h"
```

然后在 `main()` 函数中，初始化 GPIO 之后，添加 OLED 初始化代码。

示例代码如下：

```c
#include "main.h"
#include "gpio.h"
#include "OLED.h"

int main(void)
{
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();

    OLED_Init();
    OLED_Clear();

    OLED_ShowString(1, 1, "STM32 OLED");
    OLED_ShowString(2, 1, "SDA: PA9");
    OLED_ShowString(3, 1, "SCL: PA8");
    OLED_ShowString(4, 1, "Hello!");

    while (1)
    {
    }
}
```

注意：

不同 OLED 驱动文件的函数名可能略有不同。常见函数包括：

```c
OLED_Init();
OLED_Clear();
OLED_ShowChar();
OLED_ShowString();
OLED_ShowNum();
```

如果编译时报函数名不存在，需要根据你的 `OLED.h` 文件里面声明的函数名来修改。

---

## 编译和下载

### 1. 编译工程

在 Keil 中点击：

```text
Build
```

或者按快捷键：

```text
F7
```

如果没有报错，说明编译成功。

### 2. 设置下载器

进入：

```text
Options for Target → Debug
```

如果使用 DAP-Link，选择：

```text
CMSIS-DAP Debugger
```

如果使用 ST-LINK，选择：

```text
ST-Link Debugger
```

然后点击：

```text
Settings
```

确认接口选择：

```text
SWD
```

### 3. 下载程序

点击：

```text
Download
```

或者按快捷键：

```text
F8
```

下载完成后，OLED 屏幕应该显示：

```text
STM32 OLED
SDA: PA9
SCL: PA8
Hello!
```

---

## 常见问题

### 1. OLED 完全不亮

检查以下内容：

* VCC 是否接到 3.3V
* GND 是否接对
* SDA 是否接 PA9
* SCL 是否接 PA8
* PA8、PA9 是否配置成 GPIO_Output
* `OLED.c` 里面的引脚定义是否改成 PA8、PA9
* OLED 地址是否正确

常见 OLED I2C 地址有：

```text
0x78
0x7A
```

如果 `0x78` 不亮，可以尝试改成 `0x7A`。

---

### 2. 编译报错：找不到 OLED.h

错误原因：

```text
OLED.h 没有放到 Core/Inc
```

解决方法：

```text
把 OLED.h 和 OLED_Font.h 放到 Core/Inc 文件夹
```

---

### 3. 编译报错：undefined symbol OLED_Init

错误原因：

```text
OLED.c 没有加入 Keil 工程
```

解决方法：

```text
在 Keil 中手动添加 Core/Src/OLED.c
```

---

### 4. 屏幕显示乱码

可能原因：

* 字库文件 `OLED_Font.h` 不匹配
* 显示函数只支持英文和数字，不支持中文
* 显示坐标参数写错

普通 ASCII 字符可以直接显示：

```c
OLED_ShowString(1, 1, "Hello");
```

如果要显示中文，需要额外添加中文字库。

---

### 5. PA9 和串口冲突

PA9 默认常用于 USART1_TX。如果你同时启用了 USART1，OLED 的 SDA 接 PA9 可能会冲突。

解决方法：

* 关闭 USART1
* 或者把 OLED SDA 改到其他 GPIO 引脚
* 或者改用硬件 I2C 引脚，例如 PB6/PB7

---

## 实验现象

程序下载成功后，0.96 英寸 OLED 屏幕显示：

```text
STM32 OLED
SDA: PA9
SCL: PA8
Hello!
```

说明 OLED 驱动文件导入成功，GPIO 模拟 I2C 通信正常，STM32 已经成功点亮 0.96 英寸 OLED 屏幕。

 
