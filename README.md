## 1. Introduction
I just porting Jiangming's [sniffer codes](https://github.qualcomm.com/mjiang/qca6174A_rome_sniffer) to x86 platform, so you can capture Wi-Fi sniffer log using Linux laptop with an external QCA6174A PCIe mode, **no need to use MSM8996 MTP**.

Host driver baseline codes for x86 platform:
[QCA6174A.LA.1.0.5 BIT PKG 01.00.005 wiki](http://qwiki.qualcomm.com/qca_wcnss/QCA6174A.LA.1.0.5_BIT_PKG_01.00.005)

## 2. Required Hardware
* Laptop running 32-bit Linux, and has an ExpressCard interface
* PCIe interface QCA6174A module
* Two external antennas
* ExpressCard adapter (to PCIe)

## 3. Required Software
* 32-bit Linux, version: 3.18.24
* QCA6174 monitor mode host driver
* QCA6174 monitor mode firmware binary
* utf.bin, otp.bin, Data.msc, qcom_cfg.ini, fakeboar.bin
* Wireshark

## 4. Procedure
### 4.1. Sniffer Hardware Setup
Connect two antennas to 6174A module
![Antenna](https://github.com/mayuanjason/hello-world/blob/master/antenna.JPG)

Connect 6174A module to ExpressCard Adapter
![ExpressCard](https://github.com/mayuanjason/hello-world/blob/master/expresscard.JPG)

Connect ExpressCard adapter to laptop
![Laptop](https://github.com/mayuanjason/hello-world/blob/master/laptop.JPG)

### 4.2. Install 3.18.24 Linux kernel
**Download 32-bit 3.18.24 Linux header and image**
* http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.18.24-vivid/linux-headers-3.18.24-031824-generic_3.18.24-031824.201511031331_i386.deb
* http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.18.24-vivid/linux-headers-3.18.24-031824_3.18.24-031824.201511031331_all.deb
* http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.18.24-vivid/linux-image-3.18.24-031824-generic_3.18.24-031824.201511031331_i386.deb

**Switch to root user**

**Install Linux header**
* ```dpkg -i linux-headers-3.18.24-031824_3.18.24-031824.201511031331_all.deb```
* ```dpkg -i linux-headers-3.18.24-031824-generic_3.18.24-031824.201511031331_i386.deb```

**Install Linux image**
* ```dpkg -i linux-image-3.18.24-031824-generic_3.18.24-031824.201511031331_i386.deb```
 
### 4.3. Build Wi-Fi host driver
* ```git clone https://github.qualcomm.com/yuanm/qca6174A_x86_sniffer.git```
* ```cd qca6174A_x86_sniffer/fixce/AIO/build```
* ```make drivers```

When compiled successfully, there will generate 3 files under "qca6174A_x86_sniffer/fixce/AIO/rootfs-ra-r105.build/lib/modules", compat.ko, cfg80211.ko, wlan.ko

### 4.4. Copy Bin files
* Configuration: contains 1 file under "qca6174A_x86_sniffer/fw_related_files", qcom_cfg.ini, copy this file to /lib/firmware/wlan
* Firmware: contains 5 files under  "qca6174A_x86_sniffer/fw_related_files", athwlan.bin, Data.msc, fakeboar.bin, otp.bin, utf.bin, copy these files to /lib/firmware

### 4.5. Install Wireshark
```sudo apt-get install wireshark```

### 4.6. Live capture usage
**Switch to root user**

* ```insmod compat.ko```
* ```insmod cfg80211.ko```
* ```insmod wlan.ko con_mode=4```
* ```ifconfig wlan0 up```
* ```iwpriv wlan0 setMonChan 149 2```
* ```/usr/bin/wireshark```

Open the **Capture Options** dialog

![Start capture options](https://github.com/mayuanjason/hello-world/blob/master/Start%20capture%20Options.png)

Select **wlan0** interface

![Cpature options](https://github.com/mayuanjason/hello-world/blob/master/Capture%20Options.png)

Start a new live capture

![Start new live capture](https://github.com/mayuanjason/hello-world/blob/master/Start%20live%20capture.PNG)

Enjoy the live wireshark wlan capture

![Enjoy the capture](https://github.com/mayuanjason/hello-world/blob/master/Start%20capture%20Options.png)

### 4.7. On the fly channel bandwidth change
* ```iwpriv wlan0 setMonChan <channel> <bandwidth>```
* channel: channel number
* band_width: 0 for 20MHz, 1 for 40MHz, 2 for 80MHz, 3 for 5MHz, 4 for 10MHz
