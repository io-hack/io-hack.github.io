---
published: true
---
## 使用设备
- 全志H3 Orangepi Lite
- 中景园 IPS 240x240 1.14寸 st7789v

## 软件版本
- Linux - mainline 4.20.17

## 1.配置SPI0
- 编辑设备树文件 linux-4.20.17/arch/arm/boot/dts/sunxi-h3-h5.dtsi
- 开启SPI0 节点spi0: spi@1c68000 添加/修改属性 status = "okay";
- 配置SPI0引脚上拉，防止电平干扰(这是个坑，调了很久发现是干扰) 节点spi0_pins: spi0 添加 bias-pull-up;

## 2.配置内核
Symbol: STAGING [=y]                                                    │  
  │ Type  : bool                                                            │  
  │ Prompt: Staging drivers                                                 │  
  │   Location:                                                             │  
  │ (1) -> Device Drivers                                                   │  
  │   Defined at drivers/staging/Kconfig:1 
