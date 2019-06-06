---
published: true
title: 全志H3移植Linux4.20.17的RTL8189FTV驱动
---
## 设备与软件版本
- OrangePi Lite
- RTL8189FTV
- Linux 4.20.17 mainline

## 1.开启MMC控制器
### 1.1 开启MMC1
- 根据原理图RTL8189FTV挂在在MMC1
- 编辑设备树文件sunxi-h3-h5.dtsi，开启mmc1
```
		mmc1: mmc@1c10000 {
			/* compatible and clocks are in per SoC .dtsi file */
			reg = <0x01c10000 0x1000>;
			pinctrl-names = "default";
			pinctrl-0 = <&mmc1_pins>;
			resets = <&ccu RST_BUS_MMC1>;
			reset-names = "ahb";
			interrupts = <GIC_SPI 61 IRQ_TYPE_LEVEL_HIGH>;
			status = "okay";
			#address-cells = <1>;
			#size-cells = <0>;
		};
```
