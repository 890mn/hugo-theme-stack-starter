---
title: HinarUI / 原生PCB部署配置
description: HinarUI 原生 PCB 的部署与烧录配置
date: 2025-01-09 00:00:00+0000
categories: ["Embedded Develop"]
tags: ["HinarUI", "PCB"]
weight: 20
---

本篇内容将对 **HinarUI 原生 PCB** 的部署与烧录流程进行说明。  
适用于使用官方 PCB 的用户；若你使用自定义 MCU，请阅读：[HinarUI / 自定义MCU部署配置](https://link2hinar.fun/p/hinarui-/-%E8%87%AA%E5%AE%9A%E4%B9%89mcu%E9%83%A8%E7%BD%B2%E9%85%8D%E7%BD%AE/)

## 准备事项

在开始之前，请确保完成以下准备：

1. 已安装并可正常使用 **VS Code + PlatformIO**
2. 已安装 **CH340/CH9102** 等 USB 转串口驱动（视板载芯片而定）
3. 一根可靠的 **Type-C 数据线**（注意不是只充电线）

如尚未完成环境配置，请先阅读：[HinarUI / 环境配置及烧录](http://link2hinar.fun/p/hinarui_environment/)

## 获取并打开工程

1. 从 Releases 下载源码（或本地已有源码）
2. 使用 VS Code 打开 `example` 目录作为工程根目录
3. 首次打开会提示安装/配置 PlatformIO，等待其完成初始化

> 若打开后未出现 PlatformIO 状态栏，可在左侧 **扩展** 中重新安装 PlatformIO。

## 连接硬件

1. 将 HinarUI 原生 PCB 通过 Type-C 连接到电脑
2. 确认系统识别到串口设备（Windows 设备管理器中可看到 COM 口）
3. 若无法识别串口，请检查驱动是否安装

## 编译与烧录

1. 点击 VS Code 状态栏中的 **✔️** 进行编译
2. 编译成功后，点击 **➡️** 进行烧录
3. 如需监视串口输出，可打开 PlatformIO 的 **Serial Monitor**

> 若烧录失败，请优先检查串口占用、线材与供电稳定性。

## 常见问题排查

- **无法识别串口**：更换数据线或重新安装驱动
- **烧录失败**：关闭占用串口的软件（串口助手/监视器等）
- **显示异常**：确认供电稳定与屏幕排线连接牢固

完成烧录后，设备应进入 HinarUI 主界面并正常显示。

[🔙 点击此处可以返回主目录浏览](https://link2hinar.fun/p/hinarui/)