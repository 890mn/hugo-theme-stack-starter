---
title: HinarUI / 自定义MCU部署配置
description: 参考 GitHub 中现有定义整理的自定义 MCU 适配说明
date: 2025-01-09 00:00:00+0000
categories: ["Embedded Develop"]
tags: ["HinarUI", "MCU"]
weight: 20
---

在 HinarUI 的移植实践里，最核心的事情不是“写更多代码”，而是把现有定义吃透并保持结构一致。下面的流程是我在替换 MCU 时整理出的可复用经验，强调可验证、可回滚，方便你在项目里真正落地。

## 适配原则（先稳后快）

1. **先对齐结构**：优先复用 GitHub 仓库里的 MCU 定义目录结构，不要一上来大改。
2. **先点亮再优化**：先完成显示驱动，再逐步打开 UI 功能。
3. **一次只动一件事**：时钟、SPI、引脚不要同时改，便于定位问题。

## 示例：STM32F103C8 + PlatformIO

### 1) 工程配置先能编译

```ini
[env:bluepill]
platform = ststm32
board = bluepill_f103c8
framework = stm32cube
upload_protocol = stlink
monitor_speed = 115200
build_flags =
  -DHINARUI_BOARD=STM32F103
  -DHINARUI_OLED_SSD1306
```

这一阶段只看两件事：**能编译**、**能烧录**。如果这两点都不稳，后续所有改动都会被“噪声”淹没。

### 2) 时钟配置要明确，不要“看着差不多”

UI 的刷新、动画节奏都依赖系统时钟，时钟不稳就会出现跳帧或闪屏。确认主频、SysTick、总线分频一致：

```c
// board_clock.c
void SystemClock_Config(void) {
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
    RCC_OscInitStruct.HSEState = RCC_HSE_ON;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
    RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9; // 72MHz
    HAL_RCC_OscConfig(&RCC_OscInitStruct);

    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK
                                 | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
    HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2);
}
```

### 3) SPI/I2C 封装要统一，保证上层调用稳定

驱动层不要“花哨”，而是稳定、可复用。统一接口，后续换 MCU 时替换成本最低：

```c
// hal_spi.c
void hinarui_spi_write(const uint8_t *data, size_t len) {
    HAL_SPI_Transmit(&hspi1, (uint8_t *)data, len, 100);
}

void hinarui_gpio_set(uint16_t pin, GPIO_PinState state) {
    HAL_GPIO_WritePin(GPIOA, pin, state);
}
```

### 4) 引脚映射明确到位，避免“跑着跑着不对”

把关键引脚写清楚：RST、DC、CS、BL。哪怕短期能点亮，长期维护也离不开清晰映射：

```c
// board_pins.h
#define OLED_RST_PIN    GPIO_PIN_0
#define OLED_DC_PIN     GPIO_PIN_1
#define OLED_CS_PIN     GPIO_PIN_4
#define OLED_PORT       GPIOA
```

### 5) 显示参数与 UI 资源保持一致

资源尺寸、显示方向、刷新速率必须对齐；否则你会看到“局部对齐但整体偏移”的怪异问题。

## 验证顺序建议（可操作）

1. **纯色 + 清屏**：确认驱动稳定。
2. **文字/图标**：确认字库与位图方向。
3. **完整 UI**：最后再启用动画与交互。

## 经验提示

如果一次调整后变得不可控，最有效的方法就是回到仓库里“最接近的定义”，对比差异逐项排查。真正能用的移植方案，往往不是写得多，而是改得稳、改得准。
