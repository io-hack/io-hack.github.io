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
- 编辑设备树文件linux-4.20.17/arch/arm/boot/dts/sunxi-h3-h5.dtsi，开启mmc1
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
### 1.2 绑定MMC1与rtl8189ftv
- 编辑设备树文件linux-4.20.17/arch/arm/boot/dts/sun8i-h3-orangepi-lite.dts
```
&mmc1 {
	vmmc-supply = <&reg_vcc3v3>;
	bus-width = <4>;
	non-removable;
	status = "okay";

	/*
	 * Explicitly define the sdio device, so that we can add an ethernet
	 * alias for it (which e.g. makes u-boot set a mac-address).
	 */
	rtl8189ftv: sdio_wifi@1 {
		reg = <1>;
	};
};
```

## 2.移植RTL8189FTV驱动
### 2.1替换新的设备树文件，启动后查看SDIO_ID确定设备驱动
```
[root@OrangePi_Lite /]# cat /sys/bus/sdio/devices/mmc1\:0001\:1/uevent                              
DRIVER=rtl8189fs                                                                                    
OF_NAME=sdio_wifi                                                                                   
OF_FULLNAME=/soc/mmc@1c10000/sdio_wifi@1                                                            
OF_COMPATIBLE_N=0                                                                                   
OF_ALIAS_0=ethernet0                                                                                
SDIO_CLASS=07                                                                                       
SDIO_ID=024C:F179                                                                                   
MODALIAS=sdio:c07v024CdF179  
```
### 2.2下载驱动代码
- git clone https://github.com/jwrdegoede/rtl8189ES_linux.git
- cd tl8189ES_linux
- git checkout -b rtl8189fs origin/rtl8189fs
- git pull
- 根据os_dep/linux/sdio_intf.c与SDIO_ID确定匹配设备驱动为CONFIG_RTL8188F

### 2.3配置驱动代码Makefile
- CONFIG_RTL8188F = y
- CONFIG_SDIO_HCI = y
- CONFIG_PLATFORM_ARM_SUNxI = y
```
ifeq ($(CONFIG_PLATFORM_ARM_SUNxI), y)
EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
EXTRA_CFLAGS += -DCONFIG_PLATFORM_ARM_SUNxI
\# default setting for Android 4.1, 4.2
EXTRA_CFLAGS += -DCONFIG_CONCURRENT_MODE
EXTRA_CFLAGS += -DCONFIG_IOCTL_CFG80211 -DRTW_USE_CFG80211_STA_EVENT
\ 
EXTRA_CFLAGS += -DCONFIG_PLATFORM_OPS
ifeq ($(CONFIG_USB_HCI), y)
EXTRA_CFLAGS += -DCONFIG_USE_USB_BUFFER_ALLOC_TX
_PLATFORM_FILES += platform/platform_ARM_SUNxI_usb.o
endif
ifeq ($(CONFIG_SDIO_HCI), y)
\# default setting for A10-EVB mmc0
\#EXTRA_CFLAGS += -DCONFIG_WITS_EVB_V13
_PLATFORM_FILES += platform/platform_ARM_SUNxI_sdio.o
endif
\ 
ARCH := arm
\#CROSS_COMPILE := arm-none-linux-gnueabi-
\#CROSS_COMPILE=/home/android_sdk/Allwinner/a10/android-jb42/lichee-jb42/buildroot/output/external-toolchain/bin/arm-none-linux-gnueabi-
\#KVER  := 3.0.8
\#KSRC:= ../lichee/linux-3.0/
\#KSRC=/home/android_sdk/Allwinner/a10/android-jb42/lichee-jb42/linux-3.0
KVER := 4.20.17
CROSS_COMPILE := /opt/gcc-linaro-7.4.1-2019.02-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
KSRC=/home/jin/linux-4.20.17
endif
```

### 2.4针对Linux4.20.17作出修改
- LINUX_VERSION_CODE似乎并没有起作用，强制修改os_dep/linux/os_intfs.c
```
static u16 rtw_select_queue(struct net_device *dev, struct sk_buff *skb
				, struct net_device *sb_dev
				, select_queue_fallback_t fallback
)
{
	_adapter	*padapter = rtw_netdev_priv(dev);
	struct mlme_priv *pmlmepriv = &padapter->mlmepriv;

	skb->priority = rtw_classify8021d(skb);

	if(pmlmepriv->acm_mask != 0)
	{
		skb->priority = qos_acm(pmlmepriv->acm_mask, skb->priority);
	}

	return rtw_1d_to_queue[skb->priority];
}
```

### 2.5编译驱动
- 编译得到8189fs.ko
- 复制到目标系统的文件

## 3.配置内核cfg80211
```
		Symbol: CFG80211 [=y]                                                   │  
  │ Type  : tristate                                                        │  
  │ Prompt: cfg80211 - wireless configuration API                           │  
  │   Location:                                                             │  
  │     -> Networking support (NET [=y])                                    │  
  │ (1)   -> Wireless (WIRELESS [=y])                                       │  
  │   Defined at net/wireless/Kconfig:19                                    │  
  │   Depends on: NET [=y] && WIRELESS [=y] && (RFKILL [=n] || !RFKILL [=n] │  
  │   Selects: FW_LOADER [=y] && CRYPTO_SHA256 [=y]
```
```
  │ │    --- Wireless                                                     │ │  
  │ │    <*>   cfg80211 - wireless configuration API                      │ │  
  │ │    [ ]     nl80211 testmode command                                 │ │  
  │ │    [ ]     enable developer warnings                                │ │  
  │ │    [*]     enable powersave by default                              │ │  
  │ │    [ ]     cfg80211 DebugFS entries                                 │ │  
  │ │    [*]     cfg80211 wireless extensions compatibility               │ │  
  │ │    <*>   Generic IEEE 802.11 Networking Stack (mac80211)            │ │  
  │ │          Default rate control algorithm (Minstrel)  --->            │ │  
  │ │    [*]   Enable mac80211 mesh networking (pre-802.11s) support      │ │  
  │ │    [ ]   Enable LED triggers                                        │ │  
  │ │    [ ]   Export mac80211 internals in DebugFS                       │ │  
  │ │    [ ]   Trace all mac80211 debug messages                          │ │  
  │ │    [ ]   Select mac80211 debugging features  ----     
```
- 编译内核

## 4.启动RTL8189FTV
- insmod 8189fs.ko
- 查看设备 iwconfig

## 5.启动与扫描无线网络
- ifconfig wlan0 up
- iwlist wlan0 scanning
