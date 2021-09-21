# MSI-B460-MORTAR-10700-UHD630

### 平台配置 Big Sur 11.5 (OC 0.7.3)
##### 因为是从7代7700+MSIB250M+RX560（Mojave）换平台到10代10700的。这个过程中因为10代没有原生支持Mojave，所以直接通过Mojave仿冒CPU升级到BigSur11.5,这个过程也遇到不少问题，主要在于显卡驱动（UHD630，放弃RX560），网卡驱动，以及USB驱动。仅供参考，PS：EFI中三码需要重新生成。
##### 当前EFI配置主要参考官方Cometlake的配置 https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#deviceproperties
##### 整机待机在40W左右，日常使用在40W～80W之间,使用360水冷，风扇基本不转。变频节能接近完美，也是我期望的。 
```
CPU i7-10700
主板 MSIB460MORTAR
内存 Kingston 8G（2400-CL16） * 2 / Tigo 8G（2400-CL16） * 2
显卡 核显UHD 630
网卡 博通BCM943224PCIEBT2BX (淘宝63带pci卡，但是只有100M)
SSD SanDisk Extreme Pro 500GB
```


### 显卡驱动
+ 问题1：10代在BigSur主要支持问题是不支持核显直接输出，需要通过仿冒CoffeeLake进行驱动，即最合适的AAPL,ig-platform-id=07009B3E，如果有独显，可以直接使用AAPL,ig-platform-id=0300C89B，我用RX4602G版本刷成迪兰的RX5602G版本是可以完全没有问题的，但是我是用来做后端开发，对图像渲染没有要求，所以折腾到极致就是弃用独立显卡，尽最大能力折腾这个10700上的UHD630，所以只考虑核显情况。
+ 问题2：使用DP+HDMI同时输出4k+2k时会有闪屏或者1～3秒的息屏。这个问题折腾的比较久，目前不算完全解决，偶尔还是会有。缓冲帧见代码。（第一版本，已经不用）
```
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
  <key>AAPL,ig-platform-id</key>
  <data>BwCbPg==</data>
  <key>AAPL,slot-name</key>
  <string>Internal@0,2,0</string>
  <key>device-id</key>
  <data>mz4AAA==</data>
  <key>device_type</key>
  <string>VGA compatible controller</string>
  <key>enable-hdmi-dividers-fix</key>
  <data>AQAAAA==</data>
  <key>enable-hdmi20</key>
  <data>AQAAAA==</data>
  <key>framebuffer-con0-alldata</key>
  <data>AQUSAAAIAADHAwAAAgYSAAAEAADHAwAAAwQSAAAEAADHAwAA</data>
  <key>framebuffer-con0-enable</key>
  <data>AQAAAA==</data>
  <key>framebuffer-patch-enable</key>
  <data>AQAAAA==</data>
  <key>framebuffer-unifiedmem</key>
  <data>AAAAwA==</data>
  <key>model</key>
  <string>Intel CoffeeLake-H GT2 [UHD Graphics 630]</string>
</dict>
```
+ 问题2的后续改进：还是针对集成显卡UHD630。升级到最新Lilu.kext,WhateverGreen.kext,AppleALC.kext又通过一段时间的折腾，我发现这并不是因为缓冲帧的问题，其实10代UHD630本身就能使用07009B3E驱动，问题在于4K分辨率问题，我历经N次探索发现微星的这个B460M迫击炮在BIOS里虽然有GFlook的设置，但是却没有DVMT内存相关配置，只有一个集显的默认1-64M显存设置，我知道再怎么调试这个缓冲帧也无用，因为主板默认的DVMT内存没给够。通过HackinTool查看[应用补丁]-[基本信息]看到动态显存只有19M，这个容量不足以驱动4K显示器，但是微星的这个主板没法设置这个值，我查阅了一些资料只能修改BIOS或者使用EFI-Shell 可以setup_var修改DVMT Pre-Allocated的大小。但是自己修改BIOS不现实，而EFI-Shell这个感觉有破坏性我没有进行尝试了。所以我尝试framebuffer-stolenmem:00003001 -> 00000008(19M改128M)，实际我调试59M～128M均不能进入系统，没找到原因，头疼折腾NAS去了。只能调试以下缓冲帧可以实现自动到最高57M的动态显存，暂时能保证4K的输出，但是偶尔还是会有息屏，目前只能这样不影响使用，先接受了（希望微星后面BIOS能放出对DVMT的内存设置）。后续如果有更好的方案我会再进行补充。
```
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
  <key>AAPL,ig-platform-id</key>
  <data>BwCbPg==</data>
  <key>device-id</key>
  <data>kj4AAA==</data>
  <key>framebuffer-con0-enable</key>
  <data>AQAAAA==</data>
  <key>framebuffer-con0-type</key>
  <data>AAgAAA==</data>
  <key>framebuffer-fbmem</key>
  <data>AAAAAw==</data>
  <key>framebuffer-patch-enable</key>
  <data>AQAAAA==</data>
  <key>framebuffer-unifiedmem</key>
  <data>AAAAgA==</data>
  <key>model</key>
  <string>Intel UHD Graphics 630</string>
</dict>
```

### 声卡驱动
+ 声卡驱动直接通过hackintool和显卡缓冲帧一起生成补丁
```
<key>PciRoot(0x0)/Pci(0x1F,0x3)</key>
<dict>
  <key>AAPL,slot-name</key>
  <string>Internal@0,31,3</string>
  <key>device_type</key>
  <string>Audio device</string>
  <key>hda-gfx</key>
  <string>onboard-1</string>
  <key>layout-id</key>
  <data>AQAAAA==</data>
  <key>model</key>
  <string>Audio</string>
</dict>
```

### 网卡驱动
+ 使用AirportBrcmFixup.kext之前需要对这个kext进行处理，不然会panic，这个我参考的这位大哥的视频 ! https://www.bilibili.com/video/BV1k64y1D738/ 主要移除了里面AirPortBrcm4360_Injector.kext

### USB驱动
+ 新系统需要定制USB，这个相对显卡驱动比较简单，先需要USBInjectAll.kext 并且设置XhciPortLimit为yes，然后使用HackinTool打开USB，把所有USB口通过对应的USB设备插拔标记（2.0和3.0需要相应设备），然后将无用的USB设备进行移除（这里有很多人都说USB口有15个的限制，但是又有人说很多USB口是公用走的通道，不在乎15口限制，我因为没有那么多USB口，所以没有做验证，暂且认为15口限制吧），最终导出得到USBPorts.kext和SSDT-EC-USBX.aml，SSDT-UIAC.aml然后分别进行导入，再删除USBInjectAll.kext设置XhciPortLimit为no即可完成USB定制。流程就是这样，这部分网上教程比较多可以随意找到。
