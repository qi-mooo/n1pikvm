# 给 n1 安装 pikvm 廉价 ipkvm 选择
需要的物料有 n1 一个 ￥90，双公头 USB 数据线 ￥5，MS2109 芯片的采集卡 ￥35。
## 环境
首先需要给 N1 安装 Armbian，这里推荐[amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)，教程已经有很多这里不多赘述。
## 使用 fruity-pikvm 安装 pikvm 并开启 otg 功能
安装完系统第一步肯定是更新
```
apt-get update
```
使用 dtc 工具，编译 n1 设备树文件(dtb) 为可修改的 dts 文件。
```
dtc -I dtb -O dts -o n1.dts /boot/dtb/amlogic/meson-gxl-s905d-phicomm-n1.dtb
```
在生成的 dts 文件中搜索
```
dr_mode = "host";
```
把 host 模式为 peripheral 模式，开启 n1 的 otg 功能。
```
peripheral
```
编译回设备树二进制
```
dtc -I dts -O dtb -o n1.dtb n1.dts
```
移动回 boot 目录
```
mv n1.dtb /boot/dtb/amlogic/
```
编辑 uboot 文件
```
nano /boot/uEnv.txt
```
修改原先的设备树为我们开启了 otg 功能的文件
```
FDT=/dtb/amlogic/n1.dtb
```
拉取 [fruity-pikvm](https://github.com/jacobbar/fruity-pikvm)
```
git clone https://github.com/jacobbar/fruity-pikvm
```
执行 fruity-pikvm 安装脚本
```
cd fruity-pikvm && ./install.sh
```
重启 n1
```
reboot
```
## 配置 pikvm
修改 pikvm 账户(admin)的密码，pikvm 访问地址是你 n1 的 ip 地址，默认账户与密码皆是 admin
```
kvmd-htpasswd set admin
```
删除 /etc/kvmd/override.yaml 中所有内容，修改为以下内容，开启 pikvm 的 Wake on lan 功能，其中 mac 地址填写你被控电脑的 mac 地址，IP 可填可不填，不填时需要删除或者注释该行。
```
kvmd:  
    msd:  
        type: disabled
    gpio:    
        drivers:    
            wol_1:    
                type: wol    
                mac: ff:ff:ff:ff:ff:ff
                ip: 10.0.0.114
        scheme:    
            wol_1:    
                driver: wol_1    
                pin: 0    
                mode: output    
                switch: false    
        view:    
            header:      
                title: 网络唤醒
            table:    
                - ["#PC", "wol_1|唤醒"]    
```
重启 pikvm 服务
```
systemctl restart kvmd kvmd-nginx
```
此时你的 pikvm 已经可以正常工作，但是没有虚拟光驱功能，由于 n1 默认只有 8g emmc 存储，并且只有两个USB接口，推荐你购买或者自行扩容为 32G emmc 或者插入一张 sd 卡，然后修改 /etc/kvmd/override.yaml 中 msd 的 type 为 enabled 分割出一个新的分区并挂载到 /var/lib/kvmd/msd 挂载后重启n1即可使用虚拟光驱。
## 接口说明
靠近 N1 HDMI 接口的 USB 连接双公头 USB 连接线，远离 HDMI 的接口的 USB 连接 MS2109 采集卡。