# BLE物联网关网络侧API

| 版本  |   日期   |                    备注                    |
| :---: | :------: | :----------------------------------------: |
| 1.5.0 | 20220324 | 对该版本之前的所有版本进行整理，形成此文档 |

1. 本协议文档可能不是最新版本；
2. 本协议文档描述，吾控健康COSITEA蓝牙网关接口相关协议；
3. 开发者若需要硬件对接请联系销售人员,  https://www.imyfit.com ；
4. 若协议文档有更新，不再另行通知，强烈建议开发者到 https://imyfit.gitee.io 获取在线最新协议；
5. 若对协议不理解或发现协议有bug请到issues区域提出（或者提交一个 pull request），愿我们的付出对您的开发事半功倍。

## 概述

本文档适用于蓝牙网关与服务器(云端)之间的通信，接口协议使用MQTT协议，名词解释：

无线网关/路由器/gateway：均指网关。

设备：指蓝牙手环，手表等可移动设备（也称为终端）。

云端/服务器：指后台服务（例如：Java服务端）。

MQTT服务器：可自己部署到云端服务器，也可为第三方MQTT代理服务器。

### MQTT主题

默认MQTT协议Topic格式约定：类型/接收方/发送方。

|       主题        | 定义                                      |
| :---------------: | ----------------------------------------- |
| sys/gateway/cloud | 系统广播消息，从服务器发往所有网关。      |
|  sys/{MAC}/cloud  | 系统定向消息，从服务器发往指定MAC的网关。 |
|  sys/cloud/{MAC}  | 网关消息，从某MAC的网关发往服务器。       |

如网关MAC为8CD495000123，服务器消息发往该设备的主题则为：sys/8cd495000123/cloud，网关消息发往云端服务器的主题为：sys/cloud/8cd495000123，注意字母均为小写。在MQTT主题协议中常用通配符号’#’、’+’、‘/’，‘/’为分层符，订阅主题sys/ cloud/#可以订阅所有网关消息，订阅主题sys/+/cloud可以订阅所有定向消息，订阅主题sys/#可订阅所有消息，发布消息不支持通配符。默认服务器和主题供参考使用，客户亦可根据需求进行更改合适的配置。主题配置：电脑连接网关[有线（需要接入路由器）/无线]，浏览器输入192.168.8.1登录，进入服务修改，默认root无密码。参考文档[使用说明_蓝牙网关_v1.0_20200302.docx](使用说明_蓝牙网关_v1.0_20200302.docx)配置网关服务。

### 消息格式

数据传输采用JSON格式。每条数据交互都包含消息类型type、消息流水号时间戳msgId、客户端编号clientId和消息内容。如果没有的数据，值传空字符串“ ”。整体JSON格式满足如下表 ，值的类型采用string或者二级JSON格式。**键类型命名遵循第一单词首字母小写，往后每个单词首字母大写的模式；值类型命名遵循每个单词首字母大写**。其中下行的msgId网关响应的msgId相等。固定四层json嵌套（{ { { { } } } }）。

JSON格式

|    健    | 值                                                           |
| :------: | ------------------------------------------------------------ |
| comType  | 值为数据交互的JSON                                           |
|   type   | 数据类型，Up（上行）或者Down（下行）                         |
|  msgId   | UTC时间，从1970年到现在的秒数，消息id，选用当前时间戳，例如，1566197383，表示2019-08-19 14/49/43 |
| clientId | 网关以太网路由模组的MAC地址+3个随机数，云端下行时字段留空    |
| content  | 交互的具体内容                                               |

```json
{ "comType": { "type": "Up", "msgId": "1548850996", "clientId": "8CD495000156", "content": { "type": "WifiInfo", "data": { … } } } }
```

## 网关上行接口

### 所有信息上行

描述：在网关MQTT客户端启动和重连时上报。注意：不是定时上报。content的内容如下。路由未连接到网络时ip为空值。

表格 1 路由信息主动上行

|      键      | 值                                                           |
| :----------: | ------------------------------------------------------------ |
|     type     | RouterInfo，路由信息                                         |
|     mac      | 网关以太网路由模组的MAC地址                                  |
|      ip      | 网关的公网ip                                                 |
|     ssid     | 无线名称                                                     |
|   password   | 无线密码                                                     |
|  encryption  | 加密方式（0-PSK；1-PSK2；2-PSK-MIXED）                       |
| wifiVisibled | Wi-Fi的ssid是否隐藏（0-隐藏；1-可见）                        |
| wifiEnabled  | Wi-Fi是否使能（0-未启用；1-已启用）                          |
|   version    | 网关以太网路由模组固件的版本号，主版本+次版本+修改版本，例如：210 |

```json
{ "comType": { "type": "Up", "msgId": "1548851110", "clientId": "8CD495000156", "content": { "type": "RouterInfo", "data": { "mac": "8c:d4:95:00:01:56", "ip": "", "ssid": "IMYFIT-156", "password": "goodlife", "encryption": "2", "wifiVisibled": "1", "wifiEnabled": "1", "version": "1500" } } } } 
```

### Wi-Fi信息上行

描述：用户修改了Wi-Fi的信息后上报。非实时上报，间隔20秒检查一次。

表格 2 Wi-Fi信息上行

|      键      | 值                                             |
| :----------: | ---------------------------------------------- |
|     type     | Wi-Fi信息修改                                  |
|     ssid     | 修改后的无线名称                               |
|   password   | 修改后的无线密码                               |
|  encryption  | 修改后的加密方式（0-PSK；1-PSK2；2-PSK-MIXED） |
| wifiVisibled | Wi-Fi的ssid是否隐藏（0-隐藏；1-可见）          |
| wifiEnabled  | Wi-Fi是否使能（0-未启用；1-已启用）            |

```json
{ "comType": { "type": "Up", "msgId": "1548850996", "clientId": "8CD495000156", "content": { "type": "WifiInfo", "data": { "ssid": "IMYFIT-156", "password": "goodlife", "encryption": "2", "wifiVisibled": "1", "wifiEnabled": "1" } } } }
```

### 数据流量上行

描述：每个30分钟上报一次从开机到此时的总流量，数据仅供参考。

表格 3 数据流量上行

|  键  | 值                              |
| :--: | ------------------------------- |
| type | DataFlow，数据流量              |
|  rx  | 接收的流量，单位MB，保留1位小数 |
|  tx  | 发送的流量，单位MB，保留1位小数 |

```json
{ "comType": { "type": "Up", "msgId": "1548851311", "clientId": "8CD495000156", "content": { "type": "DataFlow", "data": { "rx": "0.2", "tx": "0.1" } } } }
```

## 服务器下行接口

### 路由固件升级

描述：网关上报RouterInfo后，云端检查网关固件版本，如果上报版本小于用户拥有的版本，则下发升级指令。网关收到指令检查版本后会创建一个线程进行更新并响应准备更新固件，线程等待5秒后下载固件更新，下载失败则不执行更新并发送更新失败响应。

表格 4 路由固件升级下行

|     键      | 值                                                |
| :---------: | ------------------------------------------------- |
|    type     | Upgrade，路由固件升级                             |
|     url     | 固件下载地址                                      |
|   version   | 新固件版本号                                      |
| keepSetting | 是否保留之前的配置信息（0-保留；1-不保留，默认0） |

表格 5 路由固件升级上行

|     键      | **值**                                                       |
| :---------: | ------------------------------------------------------------ |
|    type     | Upgrade，路由固件升级                                        |
|     url     | 固件下载地址                                                 |
|   version   | 新固件版本号                                                 |
| keepSetting | 是否保留之前的配置信息（0-保留；1-不保留，默认0）            |
|  response   | Latest-固件版本小于当前拥有版本<br />ReadyUpdate-准备更新，将于5秒后进行下载固件更新<br />Fail-更新线程创建失败 |

表格 6 路由固件升级失败上行

|   键   | **值**                                                    |
| :----: | --------------------------------------------------------- |
|  type  | Upgrade，路由固件升级，二次响应，只有更新失败时才会发出。 |
| update | 404-固件下载失败，更新失败                                |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1543971785", "clientId": " ", "content": { "type": "Upgrade", "data": { "url": "http://192.168.8.207:8668/openwrt-ramips-mt76x8-cositea_z01-squashfs-sysupgrade.bin", "version": "1502", "keepSetting": "1" } } } }
```

响应：

```json
{ "comType": { "msgId": "1543971785", "content": { "type": "Upgrade", "data": { "url": "http://192.168.8.207:8668/openwrt-ramips-mt76x8-cositea_z01-squashfs-sysupgrade.bin", "version": "1502", "keepSetting": "1", "response": "ReadyUpdate" } }, "type": "Up", "clientId": "8cd495000156" } }
```

更新失败上行：固件下载失败才会响应

```json
{ "comType": { "type": "Up", "msgId": "1548851630", "clientId": "8CD495000156", "content": { "type": "Upgrade", "data": { "update": "404" } } } }
```

## 功能接口

### 更新时间

描述：发送指令更新蓝牙主机UTC时间。需要注意的是time的值最大为“4294967295”，超过这个值虽然也能设置成功，但是时间不会对，溢出了。

表格 7更新时间下行

|  键  | 值                    |
| :--: | --------------------- |
| type | TimeUpdate            |
| time | 更新的时间，UTC时间。 |

表格 8 时间更新上行

|    键    | 值                         |
| :------: | -------------------------- |
|   type   | TimeUpdate，时间更新       |
| response | Ok-更新成功，Fail-更新失败 |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1543883404", "clientId": "", "content": { "type": "TimeUpdate", "data": { "time": "12312323" } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "1543883404", "clientId": "8CD495000156", "content": { "type": "TimeUpdate", "data": { "response": "Ok" } } } }
```

###  连接指定设备（短连接）

描述：建立短连接，30秒内没有数据交互则自动断开连接，设备mac长度输入不对时响应的mac会补0或者删减。

表格 9设备短连接下行

|  键  | 值                      |
| :--: | ----------------------- |
| type | ConnectDevice，请求连接 |
| mac  | 连接设备的mac地址       |

表格 10设备短连接上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | ConnnectDevice，应答                                         |
|   mac    | 连接设备的mac地址                                            |
| response | 连接结果<br />Ok-成功，<br />NotFound-连接失败，未找到该地址（连接超时）， <br />Fail-连接失败，连接请求发送失败，请重新发送，<br />Max-连接失败，蓝牙连接数已经达到最大，请断开其他设备重新连接，<br />Repeat-请求已存在) |

请求：

```json
{ "comType": { "type": "Down", "msgId": "252, "clientId": "", "content": { "type": "ConnectDevice", "data": { "mac": "EB3BC5F7C3DF" } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "252", "clientId": "8CD495000156", "content": { "type": "ConnectDevice", "data": { "mac": "EB3BC5F7C3DF", "response": "Ok" } } } }
```

### 与指定设备断开连接

描述：断开设备连接。一般用于断开长连接。

表格 11 断开设备连接下行

|  键  | 值                         |
| :--: | -------------------------- |
| type | DisconnectDevice，请求连接 |
| mac  | 设备的mac地址              |

表格 12 断开设备连接上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | DisconnectDevice，应答                                       |
|   mac    | 设备的mac地址                                                |
| response | 断开连接结果(Ok-断开成功，Fail-断开失败，该设备未连接，Error-应答错误) |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1543883404", "clientId": "", "content": { "type": "DisconnectDevice", "data": { "mac": "EB3BC5F7C3DF" } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "1543883404", "clientId": "8CD495000156", "content": { "type": "DisconnectDevice", "data": { "mac": "EB3BC5F7C3DF", "response": "Ok" } } } }
```

### 查询指定设备的连接状态

描述：查询指定设备的连接状态。一般在不确定设备是否连接成功的情况下使用。

表格 13 查询设备连接状态下行

|  键  | 值                             |
| :--: | ------------------------------ |
| type | DeviceStatus，查询设备连接状态 |
| mac  | 要查询的设备的mac地址          |

表格 14 查询设备连接状态上行

|   键   | 值                                              |
| :----: | ----------------------------------------------- |
|  type  | DeviceStatus，应答                              |
|  mac   | 设备的mac地址                                   |
| status | 查询结果(Connect-连接状态，Disconnect-断开状态) |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1573008799", "clientId": "", "content": { "type": "DeviceStatus", "data": { "mac": "EB3BC5F7C3DF" } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "1573008799", "clientId": "8CD495000156", "content": { "type": "DeviceStatus", "data": { "mac": "EB3BC5F7C3DF", "status": "Disconnect" } } } }
```

### 查询当前已连接设备

描述：查询当前已连接设备，网关最多可连接8个设备。

表格 15 查询已连接设备下行

|  键  | 值                             |
| :--: | ------------------------------ |
| type | QueryConnected，查询连接设备数 |

表格 16 查询已连接设备上行

|    键    | 值                    |
| :------: | --------------------- |
|   type   | QueryConnected，      |
|  number  | N,已连接的设备数量    |
|   mac1   | 已连接的设备的mac地址 |
|    …     | 已连接的设备的mac地址 |
| mac(N-1) | 已连接的设备的mac地址 |
|   macN   | 已连接的设备的mac地址 |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1572891991", "clientId": "", "content": { "type": "QueryConnected", "data": { } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "1572891991", "clientId": "8CD495000156", "content": { "type": "QueryConnected", "data": { "number": "4", "mac0": "EB3BC5F7C3DF", "mac1": "C7248344E0C9", "mac2": "FBB29BB8DE79", "mac3": "F3B67FCF07AB" } } } }
```

### 查询网关固件版本

描述：查询网关蓝牙主机的固件版本，可根据版本号判断固件是否需要更新。

表格 17 查询蓝牙主机固件版本下行

|  键  | 值                            |
| :--: | ----------------------------- |
| type | FirmwareVersion，查询固件版本 |

表格 18 查询蓝牙主机固件版本上行

|     键     | 值                            |
| :--------: | ----------------------------- |
|    type    | FirmwareVersion，查询固件版本 |
|    app     | 固件版本号                    |
| bootloader | 固件版本号                    |
| softdevice | 固件版本号                    |

请求：

```json
{ "comType": { "type": "Down", "msgId": 1572891991, "clientId": "", "content": { "type": "FirmwareVersion", "data": { } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": 1572891991, "clientId": "8CD495000156", "content": { "type": "FirmwareVersion", "data": { "app": 6, "bootloader": 0, "softdevice": 0 } } } }
```

### 连接指定设备（长连接）

描述：对设备建立长连接，不推荐使用，建议使用短连接。

表格 19 设备长连接下行

|  键  | 值                          |
| :--: | --------------------------- |
| type | ConnectDeviceLong，请求连接 |
| mac  | 连接设备的mac地址           |

表格 20 设备长连接上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | ConnectDeviceLong，应答                                      |
|   mac    | 连接设备的mac地址                                            |
| response | 连接结果(Ok-成功，NotFound-连接失败，未找到该地址（连接超时），Fail-连接失败，连接请求发送失败，请重新发送，Max-连接失败，蓝牙连接数已经达到最大，请断开其他设备重新连接，Repeat-请求已存在) |

请求：

```json
{ "comType": { "type": "Down", "msgId": "252", "clientId": "", "content": { "type": "ConnectDeviceLong", "data": { "mac": "EB3BC5F7C3DF" } } } }
```

应答：

```json
{ "comType": { "type": "Up", "msgId": "252", "clientId": "8CD495000156", "content": { "type": "ConnectDeviceLong", "data": { "response": "Ok", "mac": "EB3BC5F7C3DF" } } } }
```

### 重启

描述：蓝牙主机重启，注意不是路由重启。

表格 21 蓝牙主机重启下行

|  键  | 值           |
| :--: | ------------ |
| type | Reboot，重启 |

表格 22 蓝牙主机重启上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | Reboot，重启                                                 |
| response | Ok-应答成功，将进行重启，Fail-应答失败，设备忙，Error-应答错误 |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1543883404", "clientId": "", "content": { "type": "Reboot", "data": { } } } }
```

响应：

```json
{ "comType": { "type": "Up", "msgId": "1543883404", "clientId": "8CD495000156", "content": { "type": "Reboot", "data": { "response": "Ok" } } } }
```

### 恢复出厂设置

描述：蓝牙主机恢复出厂设置，注意不是路由恢复出厂设置。

表格 23 蓝牙主机恢复出厂设置下行

|  键  | 值                         |
| :--: | -------------------------- |
| type | FactoryReset，恢复出厂设置 |

表格 24 蓝牙主机恢复出厂设置上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | FactoryReset，恢复出厂设置                                   |
| response | ok-应答成功，将进行重启，fail-应答失败，设备忙，error-应答错误 |

请求：

```json
{"comType": { "type": "Down", "msgId": "1543883404", "clientId": "", "content": { "type": "FactoryTeset", "data": { } } } }
```

响应：

```json
{ "comType": { "type": "Up", "msgId": "1543883404", "clientId": "8CD495000156", "content": { "type": "FactoryTeset", "data": { "response": "Ok" } } } }
```

### 扫描参数设定

描述：过滤方式会决定数据上报方式接口[[定时上报广播数据](#_定时上报广播数据)]，当前上报方式有三个接口。在上报关闭时，无法对开关外的参数进行修改。

表格 25 扫描参数设定下行

|   键   | 值                                                           |
| :----: | ------------------------------------------------------------ |
|  type  | ScanConfig，扫描参数设定                                     |
| switch | Open-扫描开启，Close-关闭                                    |
|  mode  | Single-周期性扫描，仅扫描一个周期时间并上报数据  Keep-连续扫描，以设定时间周期定时上报数据 |
| cycle  | 0~10，扫描周期时间，单位S。                                  |
| filter | True-开启扫描数据过滤，False-关闭扫描数据过滤，Other-和关闭扫描数据过滤类似，支持扫描第三方蓝牙。 |

表格 26 扫描参数设定上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | ScanConfig，扫描参数设定                                     |
| response | Ok-参数设定成功，Fail-参数设定失败。                         |
|  switch  | Open-扫描开启，Close-关闭                                    |
|   mode   | Single-周期性扫描，仅扫描一个周期时间并上报数据  Keep-连续扫描，以设定时间周期定时上报数据 |
|  cycle   | 0~10，扫描周期时间，单位S。                                  |
|  filter  | True-开启扫描数据过滤，False-关闭扫描数据过滤，Other-和关闭扫描数据过滤类似，支持扫描第三方蓝牙。 |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1093418", "clientId": " ", "content": { "type": "ScanConfig", "data": { "switch": "Open", "mode": "Keep", "cycle": "10", "filter": "False" } } } }
```

响应：

```json
{ "comType": { "type": "Up", "msgId": "1093418", "clientId": "8CD495000156", "content": { "type": "ScanConfig", "data": { "response": "Ok", "switch": "Open", "mode": "Keep", "cycle": "10", "filter": "False" } } } }
```

### 扫描参数查询

表格 27 扫描参数查询下行

|  键  | 值                      |
| :--: | ----------------------- |
| type | ScanQuery，扫描参数设定 |

表格 28 扫描参数设定上行

|   键   | 值                                                           |
| :----: | ------------------------------------------------------------ |
|  type  | ScanConfig，扫描参数设定                                     |
| switch | Open-扫描开启，Close-关闭                                    |
|  mode  | Single-周期性扫描，仅扫描一个周期时间并上报数据  Keep-连续扫描，以设定时间周期定时上报数据 |
| cycle  | 0~10，扫描周期时间，单位S。                                  |
| filter | True-开启扫描数据过滤，False-关闭扫描数据过滤，Other-和关闭扫描数据过滤类似，支持扫描第三方蓝牙。 |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1093418", "clientId": " ", "content": { "type": "ScanQuery", "data": { } } } }
```

**响应：**

```json
{ "comType": { "type": "Up", "msgId": "1093418", "clientId": "8CD495000156", "content": { "type": "ScanQuery", "data": { "switch": "Close", "mode": "Keep", "cycle": "1", "filter": "True" } } } }
```

### 查询固件信息

描述：查询网关蓝牙主机的固件版本，可根据版本号判断固件是否需要更新。

表格 29 查询蓝牙主机固件版本下行

|  键  | 值                          |
| :--: | --------------------------- |
| type | QueryFirmware，查询固件版本 |

表格 30 查询蓝牙主机固件版本上行

|   键    | 值                          |
| :-----: | --------------------------- |
|  type   | QueryFirmware，查询固件版本 |
| version | 版本号                      |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1093418", "clientId": " ", "content": { "type": "QueryFirmware", "data": { } } } }
```

响应：

```json
{ "comType": { "type": "Up", "msgId": "1093418", "clientId": "8CD495000156", "content": { "type": "QueryFirmware", "data": { "version": "s140_6.1.0_Z_PGATEWAY_03241856" } } } }
```

### 下发告警信息

描述：下发告警信息，蓝牙主机将会通过蓝牙广播告警信息。

表格 31 查询蓝牙主机固件版本下行

|     键     | 值                   |
| :--------: | -------------------- |
|    type    | Waring，下发告警信息 |
| waringType | waring               |

表格 32 查询蓝牙主机固件版本上行

|     键     | 值                   |
| :--------: | -------------------- |
|    type    | Waring，下发告警信息 |
| waringType | 告警等级（1~3）      |

请求：

```json
{ "comType": { "type": "Down", "msgId": "1093418", "clientId": " ", "content": { "type": "Waring", "data": { "waringType": "1" } } }
```

响应：

```json
{ "comType": { "type": "Up", "msgId": "1093418", "clientId": "8CD495000156", "content": { "type": "Waring", "data": { "waringType": "1" } } }
```

### 定时上报广播数据

描述：上报方式有三种，一种为带过滤方式，通过键值ReportBordcast上报，并且数据经过处理，第二种为不带过滤通过键值ReportBordcastAll上报，将所有广播数据上报，前两种只上报扫描到的我司蓝牙设备广播数据，第三种和第二种类似，但是支持扫描到第三方设备。

表格 33 定时上报数据上行

|   键   | 值                                             |
| :----: | ---------------------------------------------- |
|  type  | ReportBordcast，定时上报扫描处理提取的广播数据 |
| number | N，扫描到的设备数量                            |
| type1  | 设备1的设备类型                                |
|  mac1  | 设备1的mac地址                                 |
| rssi1  | 设备1的信号强度                                |
| pack1  | 设备1的数据包                                  |
|   …    | …                                              |
| typeN  | 设备N的设备类型                                |
|  macN  | 设备N的mac地址                                 |
| rssiN  | 设备N的信号强度                                |
| packN  | 设备N的数据包                                  |

示例：

```json
{ "comType": { "type": "Up", "msgId": "12325894", "clientId": "8CD495000156", "content": { "type": "ReportBordcast", "data": { "number": 9, "type1": "0483", "mac1": "E72448665E55", "rssi1": "3E", "pack1": "0000000000020000000000C0F12300410EE7", "type2": "0483", "mac2": "EB3BC5F7C3DF", "rssi2": "33", "pack2": "00000000000200000000006C9F93554164AA", "type3": "0483", "mac3": "F828073B3E52", "rssi3": "39", "pack3": "0000000000020000000000B520110041225D", "type4": "0483", "mac4": "F9D74E0F5FB4", "rssi4": "2E", "pack4": "00000000000200000000009FF223004148FD", "type5": "0478", "mac5": "FF14D6DF86B6", "rssi5": "32", "pack5": "AD000000467060338A6A5CAE0800000000214B", "type6": "0483", "mac6": "E41B6876543B", "rssi6": "38", "pack6": "0000000000020000000000511E11004101A4", "type7": "0483", "mac7": "DDFB65987078", "rssi7": "31", "pack7": "0000000000020000000000AE191100414518", "type8": "0435", "mac8": "ED5801311B64", "rssi8": "39", "pack8": "000000000002000000000854EC1800412271", "type9": "0483", "mac9": "FF8241FD2A88", "rssi9": "3F", "pack9": "0000000000020000000000A71E110041095A" } } } }
```

表格 34 定时上报数据上行

|   键   | 值                                                           |
| :----: | ------------------------------------------------------------ |
|  type  | ReportBordcastAll，定时上报扫描到的广播数据，数据包pack未经过过滤处理。 |
| number | N，扫描到的设备数量                                          |
|  mac1  | 设备1的mac地址                                               |
| rssi1  | 设备1的信号强度                                              |
| pack1  | 设备1的数据包                                                |
|   …    | …                                                            |
|  macN  | 设备N的mac地址                                               |
| rssiN  | 设备N的信号强度                                              |
| packN  | 设备N的数据包                                                |

示例：

```json
{ "comType": { "type": "Up", "msgId": "1583563694", "clientId": "8CD495000152", "content": { "type": "ReportBordcastAll", "data": { "number": "3", "mac1": "3322113456A3", "rssi1": "2A", "pack1": "0201060303F0FF08FFB8030064DF13130BFFB62401B73322113456A3100949575F333332323131333435366133", "mac2": "F4BDA509D6C5", "rssi2": "43", "pack2": "02010603030D1816FFB8000000012B0100000000000000000000000000930BFFB60470B7F4BDA509D6C5100952345F663462646135303964366335", "mac3": "FADA62414FDB", "rssi3": "2E", "pack3": "02010603030D1816FFB80000000000FF0000000000716615006200002C0B0BFFB60486B7FADA62414FDB11095237535F666164613632343134666462" } } } }
```

表格 35定时上报数据上行

| **键** | **值**                                                       |
| :----: | ------------------------------------------------------------ |
|  type  | ReportBordcastOther，定时上报扫描到的广播数据，数据包pack未经过过滤处理。 |
| number | N，扫描到的设备数量                                          |
|  mac1  | 设备1的mac地址                                               |
| rssi1  | 设备1的信号强度                                              |
| pack1  | 设备1的数据包                                                |
|   …    | …                                                            |
|  macN  | 设备N的mac地址                                               |
| rssiN  | 设备N的信号强度                                              |
| packN  | 设备N的数据包                                                |

示例：

```json
{ "comType": { "type": "Up", "msgId": "71127", "clientId": "8CD495000152", "content": { "type": "ReportBordcastOther", "data": { "number": "7", "mac1": "CFEBF6A80BBD", "rssi1": "32", "pack1": "02010603030D180BFFB60486B7CFEBF6A80BBD11095237535F636665626636613830626264", "mac2": "E6BB0A7FCDC8", "rssi2": "50", "pack2": "02010612FF107803E80A3B3F00340002004A001D1F00080959543120444338", "mac3": "5EC426270BC0", "rssi3": "43", "pack3": "02011A0AFF4C001005031897E4C5", "mac4": "5CB3D653D345", "rssi4": "45", "pack4": "02011A020A0C0BFF4C001006511E987ECD49", "mac5": "6DCC0DC0F4C5", "rssi5": "51", "pack5": "02011A020A0C0BFF4C001006031A2BDB6564", "mac6": "59E5505049A3", "rssi6": "53", "pack6": "02011A020A080BFF4C001006331A6937A67E", "mac7": "94490C0400FD", "rssi7": "47", "pack7": "0201040303F0FF0FFFB62001B794490C0400FDB80462DE0201040303F0FF0E09426C7565746F6F746820546167" } } } }
```

### 心跳

场景描述：网关间隔10S发送心跳到云端，用于确认网关是否存活。

表格 36 心跳包上行

| **键** | **值**                  |
| :----: | ----------------------- |
|  type  | Heartbeat，网关发送心跳 |

示例： 

```json
{ "comType": { "type": "Up", "msgId": "1543883404", "clientId": "8CD495000156", "content": { "type": "Heartbeat", "data": { } } } }
```

## 数据透传接口

描述：透传数据需遵循我司蓝牙可穿戴产品API通信协议，即一个透传数据包最长不可超过274字节，因为以字符串形式(ascii)传输，实际为不可超过274*2字节。

表格 37 数据透传下行

|  键  | 值                                                |
| :--: | ------------------------------------------------- |
| type | Passthrough，数据透传                             |
| mac  | 透传设备mac                                       |
| data | 透传数据，MAX lenght<=274*2（274字节，ascii格式） |

表格 38 数据透传上行

|    键    | 值                                                           |
| :------: | ------------------------------------------------------------ |
|   type   | Passthrough，数据透传                                        |
| response | Ok-网关应答，透传成功，Fail-网关应答-透传失败，Reply-设备应答 |
|   mac    | 透传设备mac                                                  |
|   data   | 透传数据（ascii格式）                                        |

示例：

以下为使用我司手环进行透传数据示例，**所有透传都需要与设备建立连接。**

透传使用的蓝牙服务为0xFFF0：

| 特征值 | 属性权限 | 说明                         |
| :----: | :------: | ---------------------------- |
| 0xFFF1 |  Notify  | 数据通信必须保证Notify被使能 |
| 0xFFF2 |  Write   | 设备响应通道                 |

透传更新设备时间：

```json
{ "comType": { "type": "Down", "msgId": "1543971785", "clientId": "", "content": { "type": "Passthrough", "data": { "mac": "EB3BC5F7C3DF", "data": "68200400120000009E16" } } } }
```

网关透传失败应答：

```json
{ "comType": { "type": "Up", "msgId": 1543971785, "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "response": "Fail", "mac": "EB3BC5F7C3DF", "data": "68200400120000009E16" } } } }
```

网关透传成功应答：

```json
{ "comType": { "type": "Up", "msgId": 1543971785, "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "response": "Ok", "mac": "EB3BC5F7C3DF", "data": "68200400120000009E16" } } } }
```

设备应答：

```json
{ "comType": { "type": "Up", "msgId": "154397178", "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "response": "Replay", "mac": "EB3BC5F7C3DF", "data": "68A000000816" } } } }
```

更新完成后时间为：00:00:18

 **透传发送短消息：**

消息类型：FE，内容为”12”，详细接口信息请查看相应API手册。

```json
{ "comType": { "type": "Down", "msgId": 1543971785, "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "mac": "EB3BC5F7C3DF", "data": "680B0300FE3132D716" } } } }
```

网关透传成功应答：

```json
{ "comType": { "type": "Up", "msgId": 1543971785, "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "response": "Ok", "mac": "EB3BC5F7C3DF", "data": "680B0300FE3132D716" } } } }
```

**设备应答：**

```json
{ "comType": { "type": "Up", "msgId": "154397178", "clientId": "8CD495000156", "content": { "type": "Passthrough", "data": { "response": "Replay", "mac": "EB3BC5F7C3DF", "data": "68A000000816" } } } }
```

## 广播数据解析示例

广播数据上报方式有三种方式[（见扫描参数设定）](#_扫描参数设定)，在此提供方式1的过滤数据解析。

设定方式1示例：

```json
{ "comType": { "type": "Down", "msgId": "1093418", "clientId": " ", "content": { "type": "ScanConfig", "data": { "switch": "Open", "mode": "Keep", "cycle": "1", "filter": "True" } } } }
```

上报数据：

```json
{ "comType": { "type": "Up", "msgId": "285", "clientId": "8CD495000987", "content": { "type": "ReportBordcast", "data": { "number": "10", "type1": "0491", "mac1": "FFE8416FF71F", "rssi1": "2F", "pack1": "001F000000FF000000000606394A603D002A5C", "type2": "0826", "mac2": "C9E0842C024B", "rssi2": "21", "pack2": "550000006000E5149F180001394A600000004DA4", "type3": "0820", "mac3": "D3C5C4D99DCE", "rssi3": "3A", "pack3": "0000000000FF000000000258332E604108195666", "type4": "0820", "mac4": "D30F33033482", "rssi4": "43", "pack4": "0000000000FF0000000002BF384A6041121B06A6", "type5": "2007", "mac5": "655602058C17", "rssi5": "47", "pack5": "EC1383D3FEAA01EC13000000000000002994", "type6": "2001", "mac6": "94490C0587D7", "rssi6": "50", "pack6": "0447FB", "type7": "2007", "mac7": "B55B03040CA4", "rssi7": "48", "pack7": "E9136747F3AA01E9130000000000000064A4", "type8": "0491", "mac8": "FFE8416FF71F", "rssi8": "36", "pack8": "001F000000FF000000000607394A603D002A5D", "type9": "0820", "mac9": "FBAFDDA6458D", "rssi9": "39", "pack9": "002F0000000000000000024B374A60512E1B42E5", "type10": "0820", "mac10": "F4E7486EC2F3", "rssi10": "30", "pack10": "00007EE037600064F40300001B0400FD" } } } }
```

**其中设备2为测试者佩戴的x3w手环，数据为：**

"type2": "0826", "mac2": "C9E0842C024B", "rssi2": "21", "pack2": "550000006000E5149F180001394A600000004DA4"

（1）根据mac C9E0842C024B确认手。

（2）type确认设备型号为x3w。

（3）rssi为-21dBm（取负）。

（4）解析pack需找到对应型号的手环广播协议文档，根据x3w广播协议可知：

字节 1：心率值

字节 2：步数低字节 bit[0:7]

字节 3：步数中字节 bit[8:15]

字节 4：步数高字节 bit[16:23]

……

字节 19：电池电量（百分比） 

字节 20：异或校验值（0xB8至字节 18 的异或校值，字节B8已被过滤）

550000006000E5149F180001394A600000004DA4

当前心率为0x55：85

步数为0x000000：0

……

电池电量为0x4d：77%

校验值为0xA4







