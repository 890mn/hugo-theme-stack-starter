---
title: HinarUI / 自定义MCU部署配置
description: 参考 GitHub 中现有定义整理的自定义 MCU 适配说明
date: 2026-01-09 10:00:00+0000
categories: ["Embedded Develop"]
tags: ["HinarUI", "MCU"]
---

我自己给 HinarUI 换过 MCU，踩过的坑不止一个。下面这份流程不是“模板化步骤”，而是我照着 GitHub 里现有定义一点点抠出来的体感总结：照着它做，出问题时也更容易定位。

## 我习惯的起步思路

1. 先挑一个和你最接近的 MCU/板卡定义当“底板”。不求完美，关键是结构相同。
2. 先让屏幕点亮，再谈 UI；先跑最小 demo，再谈功能完整。
3. 每次只动一个点（时钟 / SPI / 引脚），这样错误更好抓。

## 以 STM32F103C8 为例（PlatformIO）

### 1) platformio.ini 先能编译过

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

这一步的目标很单纯：工程先跑起来，编译器路径、链接脚本、烧录方式都通了，后面再去啃 UI。

### 2) MCU 配置别贪快，最常见的坑是时钟

如果 UI 动画忽快忽慢、刷屏会“抽动”，十有八九是时钟不稳。确保主频和 SysTick 都按你选的 MCU 来：

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

### 3) SPI/I2C 接口统一接口风格，方便 UI 上层调用

别把驱动写“花”，保持接口统一，后面替换 MCU 才不崩。举个 SPI 的例子：

```c
// hal_spi.c
void hinarui_spi_write(const uint8_t *data, size_t len) {
    HAL_SPI_Transmit(&hspi1, (uint8_t *)data, len, 100);
}

void hinarui_gpio_set(uint16_t pin, GPIO_PinState state) {
    HAL_GPIO_WritePin(GPIOA, pin, state);
}
```

### 4) 引脚映射一定要写清楚

我当时最容易错的是 DC/CS/RST 引脚顺序，写清楚能救命：

```c
// board_pins.h
#define OLED_RST_PIN    GPIO_PIN_0
#define OLED_DC_PIN     GPIO_PIN_1
#define OLED_CS_PIN     GPIO_PIN_4
#define OLED_PORT       GPIOA
```

### 5) 显示参数和 UI 资源要“同频”

你导出的 UI 如果是 128x64，就别在驱动里写成 128x32。屏幕方向也要对齐，别到最后全是偏移。

## 我常用的验证顺序

1. **纯色 + 清屏**：能稳定刷屏，就说明驱动基本没问题。
2. **文字/图标**：确认字体、位图方向没反。
3. **完整 UI**：最后再上动画和交互逻辑。

## 最后一点真话

适配 MCU 就像换底盘，最怕的是“以为自己改完了”。你只要把每一步都写清楚，后面别人接手时也能少走很多弯路。如果你愿意，直接拿 GitHub 里现成的定义对照，一项项比对，很快就能定位问题。
