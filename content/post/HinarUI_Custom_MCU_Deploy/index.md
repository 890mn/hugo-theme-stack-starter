---
title: HinarUI / 自定义MCU部署配置
description: 自定义 MCU 适配说明
date: 2025-01-09 00:00:00+0000
categories: ["Embedded Develop"]
tags: ["HinarUI", "MCU"]
weight: 20
---

本篇内容将对 **自定义MCU配置** 作详细说明  
通过阅读此内容可以逐步引入不同类型的MCU并作项目适配

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

### 2) 时钟配置要明确

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

```c
// board_pins.h
#define OLED_RST_PIN    GPIO_PIN_0
#define OLED_DC_PIN     GPIO_PIN_1
#define OLED_CS_PIN     GPIO_PIN_4
#define OLED_PORT       GPIOA
```

### 5) 显示参数与 UI 资源保持一致

资源尺寸、显示方向、刷新速率必须对齐；否则你会看到“局部对齐但整体偏移”的怪异问题。

## 验证顺序建议

1. **纯色 + 清屏**：确认驱动稳定。
2. **文字/图标**：确认字库与位图方向。
3. **完整 UI**：最后再启用动画与交互。

[🔙 点击此处可以返回主目录浏览](https://link2hinar.fun/p/hinarui/)