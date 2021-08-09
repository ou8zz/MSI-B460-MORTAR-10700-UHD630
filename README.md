# MSI-B460-MORTAR-10700-UHD630

### 平台配置 Big Sur 11.5 (OC 0.7.2)
##### 因为是从7700+MSIB250M（Mojave）换平台到10700的。这个过程中因为10代没有原生支持Mojave，所以直接通过Mojave仿冒CPU升级到BigSur11.5,这个过程也遇到不少问题，主要在于显卡驱动，网卡驱动，以及USB驱动， EFI配置主要根据官方CometLake的配置 https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#deviceproperties ，整机待机在40W左右，日常使用在40W～80W之间,使用360水冷，风扇基本不转。变频节能接近完，也是我期望的。
```
CPU i7-10700
主板 MSIB460MORTAR
内存 Kingston 8G（2400超频到2666-CL16） * 2 / Tigo 8G（2400超频到2666-CL16） * 2
显卡 核显UHD 630
网卡 博通BCM943224PCIEBT2BX (淘宝63带pci卡)
SSD SanDisk Extreme Pro 500GB
```


### 显卡驱动
+ 问题1：10代在BigSur主要支持问题是不支持核显直接输出，需要通过仿冒CoffeeLake进行驱动，即最合适的AAPL,ig-platform-id=07009B3E，如果有独显，可以直接使用AAPL,ig-platform-id=0300C89B，或者默认，也能驱动，但是解码可能会有问题，我这边不打算独显，所以只考虑核显情况。
+ 问题2：单4K使用DP或者HDMI都可以正常输出，但是使用DP+HDMI同时双输出4k+2k时会有闪屏或者1～3秒的息屏。这个问题折腾的比较久，目前不算完全解决，偶尔还是会有。缓冲帧见代码。单独把显存提高到3G好很多了。目前暂时没有发现问题，后续继续观察，如果有人和我一样DP+HDMI同时输出高分辨率，可以参考。
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
+ 后续补充
