# Dell OptiPlex 9020M 黑苹果（Hackintosh）安装指南

![DELL OptiPlex 9020m](screenshots/9020m.jpg)

## 概述

[Dell OptiPlex 9020m](https://www.dell.com/support/home/ae/en/aebsdt1/product-support/product/optiplex-9020m-desktop/diagnose) 是款 Q87 芯片组的小型个人 PC，目前（2019年初）二手市场的准系统价格大概在 400-500 上下而且保有量巨大，由于使用了 4 代的 Intel CPU，可以安装魔改的移动 i7 处理器，所以具有很高的性价比。

原来已经有一台 Hackintosh 了，来自[联想的 ThinkCenter M93P](https://github.com/mingcheng/lenovo-thinkcentre-m93p-hackintosh) 机子，观察到 9020m 和它的芯片组都是为 Q87 芯片组，同时相比可以多搭载块硬盘（分别是 SATA 和 M2 8020 接口），同时还能使用 ngff 接口的无线网卡，因此又考虑多黑一台机子。

![geekbench](screenshots/geekbench.png)

简单的说，这台机子的优势是：

1. 可以使用四代魔改移动的 CPU，比较低的价格就可以上 i7 八核；
2. 安装双硬盘，支持时间胶囊；
3. 网卡使用 ngff 接口，可以搭配转接口使用 Apple 的原装无线和蓝牙模块；
4. 硬件保有量比较大，维修和替换比较方便。


## 硬件介绍

硬件方面从淘宝购买了准系统以及 4870HQ 的 CPU、两根 8g 的 DDR3 1600 三星内存条、固态硬盘为来自京东的三星 860 EVO 、蓝牙和无线网卡使用 MacBook Air 拆机的 BCM943224，搭配了 ngff 转接卡，同时 SATA 硬盘位安装了拆机的 500g 日立机械硬盘用作时间胶囊。

2019-01-23 更新：已经平滑升级到 10.14.3，没有发现任何的问题。

![about2](screenshots/about-2.png)

总体模拟为 iMac14.1 ，根据目前运行的情况完美的部分为：

1. 完美睡眠（休眠）唤醒，同时开启 HiDPI 支持 2k 显示器；
2. USB 端口、有线网卡、声卡均可以正常工作；
3. 通过注入 SSDT 搭配 CPUFriend 能够实现变频；
4. WIFI 和蓝牙能够正常使用，同时支持蓝牙键盘唤醒（还有部分不完美，需要观察）；
5. AirDrop 能够正常使用，iMessage 还为测试；
6. 可以读取风扇转速、CPU 温度、硬盘温度等。

还有不足的地方：

1. 开机 USB 鼠标会有卡顿，大概 10s 以后恢复正常；
2. <del>蓝牙连接会有时会有卡顿的现象，目前已经注入 BrcmPatchRAM2 工作正常，但仍需要观察。</del> 在 `/L/E` 中注入了 `BrcmFirmwareData.kext` 和 `BrcmPatchRAM2.kext` 解决。


## 安装指南

### BIOS 设置

Dell 的机子相比联想的机子在 BIOS 上操作比较复杂（个人不是很喜欢使用鼠标操作设置 BIOS），因此请务必小心和检查 BIOS 设置是否都已经生效。

通常二手的机子使用的时间都比较长，可能从来没有更换过 CMOS 电池，同时建议收到二手的机子以后，更换 CMOS 电池（这点是比较血泪的教训），9020M 的 CMOS 电池型号是 CR2032 。

设置对应的 BIOS，英文对照：

* Boot sequence -> UEFI
* Advanced Boot Options -> Uncheck Enable Legacy Option ROMs - (only if graphics are UEFI capable)
* Serial Port -> Disabled
* Sata Operation -> AHCI
* Integrated NIC -> Enabled
* Secure Boot -> Disabled

然后重启即可。

### 显卡

本机搭配的是 [4870HQ 搭配了 Iris™ Pro Graphics 5200 的核心显卡](https://ark.intel.com/products/83504/Intel-Core-i7-4870HQ-Processor-6M-Cache-up-to-3-70-GHz-)，可以正确被 Mojave 驱动，同时通过打 FrameBuffer 补丁以后显示 2048m 的显存。目前，主要通过 WhateverGreen 驱动以及使用 FB-Patcher 打补丁。

```
<key>ig-platform-id</key>
<string>0x0d220003</string>
```

然后打上对应的补丁

```
	<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
	<dict>
		<key>framebuffer-patch-enable</key>
		<data>
		AQAAAA==
		</data>
		<key>framebuffer-unifiedmem</key>
		<data>
		AAAAgA==
		</data>
	</dict>
```

然后就可以看到运行正常了：

![fb-patcher](screenshots/fb-patcher.png)


### 声卡

9020m 的声卡型号是 ALC255，注入 id 为 27 。使用的是 AppleALC 注入的合适，没有修改对应的 DSDT。

```
	<key>PciRoot(0x0)/Pci(0x1b,0x0)</key>
	<dict>
		<key>layout-id</key>
		<data>
		GwAAAA==
		</data>
	</dict>
```

经过测试，这样子设置以后就可以完美使用，具体更多的设置方法请参见教程：

http://blog.daliansky.net/Use-AppleALC-sound-card-to-drive-the-correct-posture-of-AppleHDA.html


### 网卡和蓝牙

网卡和蓝牙这块替换了苹果提供的 `BCM943224` 然后使用转接卡转接到 ngff 插口上，硬件方面这个网卡的尺寸刚刚好可以容纳主机的空间，如下图：

![BCM943224](screenshots/BCM943224.jpg)

注意蓝牙天线以及 Wifi 天线的插头位置（我插反过，然后 Wifi 和蓝牙的信号都很差）。虽然这个网卡可以免驱动就可以使用，但是还是建议注入后使用，具体的方式参见：

https://www.tonymacx86.com/threads/broadcom-wifi-bluetooth-guide.242423/

目前的问题是有部分时候蓝牙键盘连接会有卡顿的现象。解决的方案是同时按 `Command + Option` 然后点击蓝牙图标，就可以弹出调试菜单：

![blutooth-reset](screenshots/bluetooth-reset.png)

初始化蓝牙模块以及已连接的 Apple 设备后，再重新插拔下就可以使用，但目前没有再发生卡顿的情况，还是需要观察。

更新（2019-01-23）：经过一周的测试，在 `/L/E` 中注入了 `BrcmFirmwareData.kext` 和 `BrcmPatchRAM2.kext` 没有发生卡顿的现象。


### CPU 变频

本机搭配了 4870HQ 的 CPU，变频这块可以参考 EFI 中 `ACPI/dsl/SSDT-0-CpuFriend.sdl` 这个文件，以下是效果：

![Intel-power-gadget](screenshots/intel-power-gadget.png)

待机温度能够有效控制在 50 度以内。相比 ThinkCenter 的 4720HQ 从运行温度的角度上说，这块 CPU 对温度的控制总体温度低点。

### 其他

映射正确的 SATA 方式，避免造成启动的时候磁盘顺序混乱，因此需要在 ACPI 下打个补丁

```
  <dict>
  	<key>Comment</key>
  	<string>change SAT0 to SATA</string>
  	<key>Disabled</key>
  	<false/>
  	<key>Find</key>
  	<data>
  	U0FUMA==
  	</data>
  	<key>Replace</key>
  	<data>
  	U0FUQQ==
  	</data>
  </dict>
```

来源出处参考这里： https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/

## 其他

### 安装后的常规操作

隐藏第三方启动「允许任何来源的应用」选项

```
sudo spctl --master-disable
```

强制开启第三方 SSD 的 Trim 功能

```
sudo trimforce enable
```

删除启动确认的对话框，通常通过 Brew 等渠道的安装包：

```
sudo xattr -r -d com.apple.quarantine /Applications
```

提取 EDID，以及注入 DisplayVendorID 和 DisplayProductID

```
ioreg -lw0 | grep -i "IODisplayEDID" | sed -e 's/.*<//' -e 's/>//'
ioreg -lw0 | grep IODisplayPrefsKey
```

## 参考资源

* https://comsysto.github.io/Display-Override-PropertyList-File-Parser-and-Generator-with-HiDPI-Support-For-Scaled-Resolutions/
* https://www.tonymacx86.com/threads/broadcom-wifi-bluetooth-guide.242423/
* https://www.tonymacx86.com/threads/an-idiots-guide-to-lilu-and-its-plug-ins.260063/
* https://blog.daliansky.net/Mac-frequently-used-to-the-command---continuous-update.html
* https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/

`- eof -`