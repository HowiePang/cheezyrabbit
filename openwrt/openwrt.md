# openwrt


官方网站：https://openwrt.org/zh/start

<img src="./assets/image-20260810143216621.png" alt="image-20260810143216621" style="zoom:50%;" />

## 1、红米AC2100

官方红米ac2100刷机openwrt记录，默认管理后台地址为 **192.168.31.1**

此处选择的openwrt版本为 v23.05.5，固件下载地址为：https://downloads.openwrt.org/releases/23.05.5/targets/ramips/mt7621/

<img src="./assets/image-20260810143254514.png" alt="image-20260810143254514" style="zoom:50%;" />

红米降级固件地址：http://cdn.cnbj1.fds.api.mi-img.com/xiaoqiang/rom/rm2100/miwifi_rm2100_firmware_d6234_2.0.7.bin

下载 breed软件（防止刷机失败已回退）：https://breed.hackpascal.net/

<img src="./assets/image-20260810143320212.png" alt="image-20260810143320212" style="zoom:50%;" />

整理以上所需的资源如下：

<img src="./assets/image-20260810143336779.png" alt="image-20260810143336779" style="zoom:50%;" />

----

**开始安装**

1、降级ROM

登录路由器的管理页面
浏览器访问`192.168.31.1`

<img src="./assets/image-20260810143428436.png" alt="image-20260810143428436" style="zoom:50%;" />

<img src="./assets/image-20260810143439827.png" alt="image-20260810143439827" style="zoom:50%;" />

<img src="./assets/image-20260810143450804.png" alt="image-20260810143450804" style="zoom:50%;" />

<img src="./assets/image-20260810143501975.png" alt="image-20260810143501975" style="zoom:50%;" />

<img src="./assets/image-20260810143513087.png" alt="image-20260810143513087" style="zoom:50%;" />

2、安装Breed

<img src="./assets/image-20260810143540045.png" alt="image-20260810143540045" style="zoom:50%;" />

```go
// 浏览器打开，注意stok值的替换
http://192.168.31.1/cgi-bin/luci/;stok=81b129ec259253e260ef3629ed3ce178/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20https%3A%2F%2Fbreed.hackpascal.net%2Fr1286%2520%255b2020-10-09%255d%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%20-g%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20242d42eb5f5aaa67ddc9c1baf1acdf58d289e3f792adfdd77b589b9dc71eff85)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A
```

<img src="./assets/image-20260810143601363.png" alt="image-20260810143601363" style="zoom:50%;" />

<img src="./assets/image-20260810143626052.png" alt="image-20260810143626052" style="zoom:50%;" />

3、安装OpenWrt固件

<img src="./assets/image-20260810143658804.png" alt="image-20260810143658804" style="zoom:50%;" />

4、openwrt的一些设置

安装新的UI：luci-theme-material

<img src="./assets/image-20260810143726804.png" alt="image-20260810143726804" style="zoom:50%;" />

<img src="./assets/image-20260810143738210.png" alt="image-20260810143738210" style="zoom:50%;" />

安装中文：`luci-i18n-base-zh-cn`

<img src="./assets/image-20260810143756446.png" alt="image-20260810143756446" style="zoom:50%;" />

5、ssh登录

<img src="./assets/image-20260810143815952.png" alt="image-20260810143815952" style="zoom:50%;" />



