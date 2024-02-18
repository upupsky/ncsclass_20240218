# 蓝牙网关配置

| 版本 | 日期 | 备注 |
|-------|-----|------|
| 1.0.0 | 20220727 | 对安装过程进行整理，形成此文档|

1. 本文档（PDF版本时）可能不是最新版本；
2. 开发者若需要硬件对接请联系销售人员,  https://www.imyfit.com ；
3. 若文档有更新，不再另行通知，强烈建议开发者到 https://imyfit.gitee.io 获取在线最新文档；
4. 若有疑问请到ISSUE区提出 https://gitee.com/imyfit/imyfit-issue 愿我们的付出对您的开发事半功倍。

## 初始检查

确认外观完好无损坏污渍，关键信息完整。

![输入图片说明](picture_gateway\outA.jpg)

蓝牙网关使用ＰＯＥ供电方式，插上POE网线连接POE适配器后自动开机，初始化完成后向外广播IMYFIT-xxx无线信号 (xxx为mac地址后三位，mac地址见POE网口处)

![输入图片说明](picture_gateway\outB.jpg)

使用电脑连接该网络，初始密码为goodlife。



连接后打开浏览器输入192.168.8.1进入后台控制页面，默认无密码直接登录。

![输入图片说明](picture_gateway\imyfit_web.png)

成功进入后台可以看到当前设备的总览信息，表明设备运行正常无故障。

![输入图片说明](picture_gateway\imyfit_all.png)

## 联网设置

联网方式主要有两种，根据现场情况而定

### 有线连接

这是网关的默认连接方式，POE适配器有两个网线插口，POE口插入网线连接蓝牙网关，LAN口插入网线连接路由设备，通电启动，网关会自行联网并获取IP地址。 

![输入图片说明](picture_gateway\POE1.jpg)

**此方法局限为需要网线布设，有可能增加施工难度和成本。**


​     
### 无线连接

在网关后台页面找到 网络 --> 无线 --> 找到radio0 点击扫描开始寻找附近的wifi信号

![输入图片说明](picture_gateway\imyfit_wifi.png)

在扫描结果中寻找既定wifi网络，点击加入网络，输入密钥，其他不变，点击提交，而后点击保存并应用。

![输入图片说明](picture_gateway\imyfit_wifi_add.png)

![输入图片说明](picture_gateway\imyfit_psk.png)

![输入图片说明](picture_gateway\imyfit_wifi_submit.png)



此时网关会重新启动，电脑与网关的连接会断开，等待网关重新启动后向外发送wifi信号，重复[1、初始检查]重新连接网关即可。    
**此方法不需要网线布设，不受空间限制，但可能受到路由设备信号影响导致连接不稳定，实施过程中需注意。**

## 升级设备

完成联网后将设备进行升级以适配最新的设备和程序。 **升级并非必须，依据实际情况而定，如已装配最新程序可不必升级**   
蓝牙网关升级分为两部分，第一部分为蓝牙系统升级，第二部分为网关系统升级。

### 蓝牙系统升级

蓝牙系统更新升级需要手机安装nrf connect官方软件，打开软件搜索wk_xxxxxxxx的蓝牙设备信号(xxxxx为蓝牙设备mac地址，mac地址见poe网口处)

![输入图片说明](picture_gateway\server_1.jpg)

点击connect连接该蓝牙设备 

连接成功后找到Nordic UART Service（过于老旧的设备可能会显示Unknown Service）并点开

![输入图片说明](picture_gateway\RX_up.jpg)

在RX Characteristic上行发送 0xb1启动升级 设备会断开蓝牙连接
**注意：0x已显示**

![输入图片说明](picture_gateway\b01.jpg)    


在scanner中重新搜寻DFuTarg 连接这个蓝牙信号

![输入图片说明](picture_gateway\dft_serc.jpg)

点击右上角DISCONNECT旁DFU图标，选择ZIP升级，点击OK

![输入图片说明](picture_gateway\dfu.jpg)

![输入图片说明](picture_gateway\zip.jpg)

进入本机文件系统寻找ZIP升级包，选择后自动开始升级。    

![输入图片说明](picture_gateway\select_zip2.jpg)

![输入图片说明](picture_gateway\100.png)

升级过程中不可断电，不可打断。若闪退或未到100%自动退出代表升级失败，可能由于版本不可用或其他未知原因。请再次尝试，若仍无法完成更新，需要拆卸设备进行J-Link硬件烧写。硬件烧写请联系硬件工程师，不在此赘述。

### 网关系统升级

在后台web页面中的 系统 --> 备份/升级 --> 刷写新的固件 --> 选择文件 --> 选择保存好的升级 .bin文件      

![输入图片说明](picture_gateway\imyfit_up.png)

![输入图片说明](picture_gateway\imyfit_up_select.png) 

确定正确无误后点击<刷写固件>    

![输入图片说明](picture_gateway\imyfit_up_over.png)

固件刷写过程中会重新启动，电脑与网关的连接会断开，等待网关重新启动后向外发送wifi信号，重复[**初始检查**]重新连接网关即可。

## MQTT服务
### 网关配置MQTT
网关后台web页面中选择 服务 --> ble host control
勾选启用服务
主机指mqtt代理服务器的地址或域名(此处broker安装到本机172.16.67.134)
端口默认为1883
主题不要变动
保存并应用即可

![输入图片说明](picture_gateway\mqtt.png)

### MQTT测试

打开MQTT.fx,点击齿轮进入配置

![输入图片说明](picture_gateway\fx1.png)

点击配置界面左下角加号新增一个连接，更改连接名称，在Broker Address处输入MQTT服务地址，其他可无需变动，点击右下角Apply，提交后关闭界面。

![输入图片说明](picture_gateway\fx_profile.png)

关闭配置界面后重新回到主界面，打开下拉菜单选择刚才的连接配置，点击Connect尝试连接MQTT服务。

![输入图片说明](picture_gateway\fx_conn1.png)

右上角小锁打开，状态灯变绿表明我们已经成功连接到MQTT服务。
点击Subscribe进入订阅功能界面，在主题栏输入我们要订阅的主题，即蓝牙网关的发布主题**见网关配置MQTT**
随后蓝牙网关上传的数据就会在下方显示。
![输入图片说明](picture_gateway\fx_conn2.png)

