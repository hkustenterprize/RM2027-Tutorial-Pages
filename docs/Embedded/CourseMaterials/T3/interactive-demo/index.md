# Tutorial 3 交互式 Demo

下面的动画用于演示 UART 接收、中断和 DMA 的工作过程。点击每个动画右上角的“全屏”按钮，可以铺满浏览器窗口观看。

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

## UART 接收过程

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="uart_receive.html" title="STM32 UART 接收过程" loading="lazy" allowfullscreen></iframe></div>

## UART 接收超时

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="uart_receive_timeout.html" title="STM32 UART 接收超时" loading="lazy" allowfullscreen></iframe></div>

## UART 中断接收

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="uart_receive_it.html" title="STM32 UART 中断接收" loading="lazy" allowfullscreen></iframe></div>

## UART + DMA 接收

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="uart_receive_DMA.html" title="STM32 UART + DMA 接收" loading="lazy" allowfullscreen></iframe></div>

## UART 按位解码

<div class="tutorial3-frame"><button class="tutorial3-fullscreen" type="button" onclick="this.parentElement.requestFullscreen()">⛶ 全屏</button><iframe class="tutorial3-demo" src="uart_decode_by_bit.html" title="UART 按位解码" loading="lazy" allowfullscreen></iframe></div>
