# 嵌入式部门 Tutorial

## 嵌入式开发环境配置

请根据所使用的操作系统选择对应的环境配置指引：

| 操作系统 | 配置入口 |
| --- | --- |
| Windows | [Windows 环境配置指引](<Environment/Windows/index.md>) |
| macOS | [macOS 环境配置指引](<Environment/macOS/index.md>) |
| Linux | [Linux 环境配置指引](<Environment/Linux/index.md>) |

完成环境配置后，继续阅读：

[嵌入式环境验证](<Environment/Verify/index.md>)

## 基础教程

这里整理了嵌入式开发中常用工具的入门资料，包括 STM32CubeMX 基础教程和 Ozone 使用教程：

[查看嵌入式基础教程（Google Drive）](https://drive.google.com/drive/folders/1ChaGXjBYJlJSYJRxwt5jeC3wrcTmKsQ-?usp=drive_link)

## 课程 { #courses }

| 课程 | 材料 |
| --- | --- |
| Tutorial 0：C++ Tutorial | [Google Drive](https://drive.google.com/drive/folders/1hbmdChpYoPN75QKTG9d-vQLtOlBAbcfT?usp=drive_link) |
| Tutorial 1：Introduction to Embedded System | [Google Drive](https://drive.google.com/drive/folders/12UpMeoHU_ZNJ9v3CEk3bbNuVo3BYgGuR?usp=drive_link) |
| Tutorial 2：Communication Protocols & UART | [查看讲义](<CourseMaterials/T2/Embedded Tutorial 2 - 通信协议 UART.md>)<br>[补充资料](CourseMaterials/T2/supplementary-reading.md) |
| Tutorial 3：Interrupt & DMA (UART Receive) | [查看讲义](<CourseMaterials/T3/Embedded Tutorial 3 - UART接收 中断与DMA.md>)<br>[交互式 Demo](#tutorial3-demo) |
| Tutorial 4：CAN Communication | [查看课件](<CourseMaterials/T4/Embedded Tutorial 4 - CAN Communication.pdf>) |
| Tutorial 5：Control & PID | [Google Drive](https://drive.google.com/drive/folders/1IYNPPKZhDktt1x0LAvl82ve-gzqShWJt?usp=drive_link) |
| Tutorial 6：Clock & Timer & PWM | [Google Drive](https://drive.google.com/drive/folders/10qTFk3EfvK6kUb6KxoNQUj0eGSXcNe_9?usp=drive_link) |

<details id="tutorial3-demo" open>
<summary>交互式 Demo</summary>

下面的动画可以直接在网站中播放，帮助理解 UART 接收、中断和 DMA 的工作过程。

<style>
.tutorial3-demo {
  width: 100%;
  height: 100%;
  border: 0;
  border-radius: 12px;
  background: #02040a;
}
.tutorial3-frame {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  background: #02040a;
  border-radius: 12px;
  overflow: hidden;
}
.tutorial3-frame:fullscreen {
  width: 100vw;
  height: 100vh;
  border-radius: 0;
}
.tutorial3-fullscreen {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 2;
  padding: 6px 10px;
  border: 1px solid rgba(125, 211, 252, .45);
  border-radius: 8px;
  background: rgba(2, 4, 10, .72);
  color: #dbeafe;
  cursor: pointer;
}
</style>

<h4>UART 接收过程</h4>

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="CourseMaterials/T3/interactive-demo/uart_receive.html" title="STM32 UART 接收过程" loading="lazy" allowfullscreen></iframe></div>

<h4>UART + DMA 接收</h4>

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="CourseMaterials/T3/interactive-demo/uart_receive_DMA.html" title="STM32 UART + DMA 接收" loading="lazy" allowfullscreen></iframe></div>

<h4>UART 中断接收</h4>

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="CourseMaterials/T3/interactive-demo/uart_receive_it.html" title="STM32 UART 中断接收" loading="lazy" allowfullscreen></iframe></div>

<h4>UART 接收超时</h4>

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="CourseMaterials/T3/interactive-demo/uart_receive_timeout.html" title="STM32 UART 接收超时" loading="lazy" allowfullscreen></iframe></div>

<h4>UART 按位解码</h4>

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="CourseMaterials/T3/interactive-demo/uart_decode_by_bit.html" title="UART 按位解码" loading="lazy" allowfullscreen></iframe></div>

</details>

## 作业

| 作业 | 内容 | 截止时间 |
| --- | --- | --- |
| [作业 1：单片机通讯](Assignments/Assignment1/index.md) | 接收遥控器消息：接线与配对、SBUS 解码、链路状态检测及串口回传 | 2026 年 10 月 1 日 22:00 |
| 作业 2：电机控制 |  |  |
