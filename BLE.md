# Bluetooth Low Energy - BLE #

----------

### 基本概念 ###
* BLE介绍

### 系统结构 ###
* 控制器
* 主机
* 应用

### 测试及工具 ###
* 认证测试
* 系统测试
* 抓包
* PTS工具
* CEVA Test Tool


## 基本概念 - BLE介绍 ##
>蓝牙是一种短距的无线通讯技术，可实现固定设备、移动设备之间的数据交换。

>蓝牙标准包括经典蓝牙部分和低功耗蓝牙模块部分。低功耗蓝牙是建立在经典蓝牙基础之上继而发展的，并有别于经典蓝牙模块；

![](https://5b0988e595225.cdn.sohucs.com/images/20170825/7210c0e9f14b491a9d6c8da8291a6037.png "蓝牙分类")

1. 经典蓝牙模块（BT）：泛指支持蓝牙协议在4.0以下的模块，一般用于数据量比较大的传输。经典蓝牙模块可再细分为：传统蓝牙模块和高速蓝牙模块；

2. 低功耗蓝牙模块（BLE）：BLE是为极低功率运行而设计的。为了实现在2.4GHz频段的可靠运行，它采用了强大的跳频扩频方法，可在40个通道上传输数据。BLE为开发人员提供了极大的灵活性，包括多种PHY选项，支持125 Kb/s至2 Mb/s的数据传输速率，多种功率水平，从1mW到100mW，以及多种安全选项。还支持多种网络拓扑结构，包括点对点、广播和mesh 网络。

3. 频带资源：BLE的市场定位是个体和民用，因此使用免费的ISM频段（频率范围是2.400-2.4835 GHz）；为了同时支持多个设备，将整个频带分为40份，每份的带宽为2MHz，称作RF Channel；

4. 我们用的是BLE5.0，相比于经典蓝牙，有以下区别：

| 	                  |Bluetooth Low Energy (LE)                                                                                   |Bluetooth Classic   	                                                       |
| ---                 | :--                                                                                                        | :--                                                                           |
|Frequency Band       |2.4GHz ISM Band (2.402 – 2.480 GHz Utilized)                                                                |2.4GHz ISM Band (2.402 – 2.480 GHz Utilized)                                   |
|Channels 	          |40 channels with 2 MHz spacing<br>(3 advertising channels/37 data channels)                                 |79 channels with 1 MHz spacing                                                 |
|Channel Usage        |Frequency-Hopping Spread Spectrum (FHSS)                                                                    |Frequency-Hopping Spread Spectrum (FHSS)                                       |
|Modulation           |GFSK									                                                                       |GFSK, π/4 DQPSK, 8DPSK                                                         |
|Power<br>Consumption |~0.01x to 0.5x of reference<br>(depending on use case)                                                      |1 (reference value)                                                            |
|Data Rate            |LE 2M PHY: 2 Mb/s<br>LE 1M PHY: 1 Mb/s<br>LE Coded PHY (S=2): 500 Kb/s<br>LE Coded PHY (S=8): 125 Kb/s      |EDR PHY (8DPSK): 3 Mb/s<br>EDR PHY (π/4 DQPSK): 2 Mb/s<br>BR PHY (GFSK): 1 Mb/s|
|Max Tx Power*        |Class 1: 100 mW (+20 dBm)<br>Class 1.5: 10 mW (+10 dbm)<br>Class 2: 2.5 mW (+4 dBm)<br>Class 3: 1 mW (0 dBm)| Class 1: 100 mW (+20 dBm)<br>Class 2: 2.5 mW (+4 dBm)<br>Class 3: 1 mW (0 dBm)|
|Network<br>Topologies|Point-to-Point (including piconet)<br>Broadcast<br>Mesh                                                     | Point-to-Point (including piconet)                                            |


## 系统结构 - 整体架构 ##
![系统结构](https://img-blog.csdnimg.cn/20190723173502852.png "系统结构")


## 系统结构 - 控制器 ##
* HCI：Host Controller interface 主机控制接口，即主机和控制器之间的交互接口。向上为主机提供软件应用程序接口（API），对外为外部硬件控制接口，可以通过串口、SPI、USB来实现设备控制;
* LL：Link Layer 链路层，用于控制设备的射频状态。即控制设备处于准备（standby）、广播、监听/扫描（scan）、初始化、连接，这五种状态中一种，五种状态机的关系见下图；
  
![](https://imgconvert.csdnimg.cn/aHR0cDovL2Jsb2cuY2hpbmF1bml4Lm5ldC9hdHRhY2htZW50LzIwMTYwMy8zMS8yMTQxMTIyN18xNDU5NDE2Mjg2ejhIay5wbmc?x-oss-process=image/format,png)

* PHY：Physical 物理层，负责数据的物理传输，使用GFSK调制，40个频道，2M频间隔，3个广播(37,38,39)，37个数据通道；


## 系统结构 - 主机 ##
* L2CAP：Logical Link Control and Adaption Protocol 逻辑链路控制和适配协议，是一个复用层，可以让低功耗蓝牙复用三条不同的信道。它也支持数据的分割和重组功能，使得较大的报文可以在底层无线电中传输。可以理解为L2CAP对LL进行了一次简单封装，LL只关心传输的数据本身，L2CAP就要区分是加密通道还是普通通道，同时还要对连接间隔进行管理。
* SMP：Security Manage Protocol 安全管理协议，是用来进行安全管理的，其定义了配对和密钥分发的过程实现；
* ATT：Attribute 属性协议，是GATT和GAP的基础，它定义了BLE协议栈上层的数据结构和组织方式；
* GATT：Generic Attributes 通用属性协议，GATT使用ATT协议定义了服务框架。该框架定义服务的程序和格式以及其特征(characteristics)，定义的程序包括发现，读取，写入，通知和指示特性，以及配置特性的广播。 
* GAP：General Access Profile 通用接入规范，主要描述了两个蓝牙设备建立通信时必要的基本操作，包括设备的发现、链路的建立和配置以及安全性设置。


## 系统结构 - 应用层 ##
>最上层的Profiles层里，包含的公用任务和私有任务，其中公共任务是 SIG 蓝牙协议小组定义的蓝牙任务，私有任务是用户或者企业自定义的蓝牙任务。


## 测试及工具 - 认证测试 ##
>蓝牙认证是任何使用蓝牙无线技术的产品所必须经过的证明程序。蓝牙技术是SIG发明的，要使用它的专利，必须拿到它的授权。所以只有经过蓝牙技术联盟测试认证符合蓝牙标准后，才有资格以蓝牙产品的名称投入市场，否则就是违法的。同时可提高不同厂家产品的互操作性，确保流程一致性。蓝牙技术联盟规定，不仅生产商需有BQB认证，贴牌的品牌的所有者也需要有BQB认证。

![](https://img-blog.csdnimg.cn/2019081310045333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o0OTc1NDQ4NDk=,size_16,color_FFFFFF,t_70 "认证流程")

* BQB：Bluetooth Qualification Body，我们一般称为蓝牙认证;
* SIG：Bluetooth Special Interest Group 蓝牙技术联盟，是一个以制定蓝牙规范，以推动蓝牙技术为宗旨的跨国组织。它拥有蓝牙的商标，负责认证制造厂商，授权他们使用蓝牙技术与蓝牙标志，但是它本身不负责蓝牙装置的设计、生产及贩售;

### RF测试 ###
>为了评估每一款蓝牙设计的射频特性相对蓝牙无线技术规范的符合性需要进行RF一致性测试（需要使用经Bluetooth SIG认证的测试系统进行测试）;

>传统蓝牙(classic BR/EDR)测试样品需要提供控制软件以设定进入测试模式，低功耗蓝牙(BLE)测试样品需要支持直接测试模式;

* DTM：Direct Test Mode 直接测试模式，BLE协议充分考虑了设备的测试问题，在协议栈层面提供了直接测试模式，用于执行BLE设备的RF物理层一致性的测试。核心工作是令设备在指定的频率上发送一段数据序列，在另一端使用测试设备接收数据序列并给出测试报告，或者反过来测试设备发送一段数据序列，BLE设备接收并给出测试报告。测试报告中会给出通信频率的偏移量，频率的功率，通信丢包率PER（Packet Error Rate）等信息，并根据这些信息判断BLE设备的物理层是否满足设计要求。
##### 工作模式介绍: #####
![](http://news.21dianyuan.com/upload/2019/0221/20190221102657233.png "测试示意图")
##### PC向DUT发送测试命令，具体如下： #####
|测试命令            |含义            |
| :---              | :---          |
|LE_TEST_SETUP      |设置测试        |
|LE_TRANSMITTER_TEST|BLE设备发射机测试|
|LE_RECEIVER_TEST   |BLE设备接收机测试|
|LE_TEST_END        |停止测试        |
##### DUT收到命令会向PC返回事件消息，具体如下： ####
|事件消息         |含义                        |
| :---           | :---                      |
|LE_STATUS       |收到命令后，DUT返回命令执行状态|
|LE_PACKET_REPORT|停止测试后，DUT返回测试报告   |

![](https://img-blog.csdnimg.cn/20200506235426484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDEwODM4MA==,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20200506235500281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDEwODM4MA==,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20200506235518380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDEwODM4MA==,size_16,color_FFFFFF,t_70)

### PTS测试 ###
>Profile Tuning Suite（PTS）是一个自动化测试软件，符合测试蓝牙HOST部分指定的功能需求和HCI之上的协议规格。对于蓝牙开发和测试，从产品的角度可以提供完整有效的所有指定功能需求的测试范围;

>PTS可在电脑上运行，并模拟任何蓝牙设备或规范。借助这一工具，企业可以利用多种现有应用来快速测试新设备;

#### 准备工作： ####
1. Bluetooth SIG官网下载安装软件，在windows里安装PTS软件；
2. 查询厂商（如：Nurlink Technology Corp.）的Declaration ID和QDID来下载ICS；
3. 将PTS dongle插入电脑，安装驱动。打开PTS软件，开始建立workspace。（测试LE特性蓝牙协议，需要支持LE特性dongle）；
4. 进行测试，具体工具使用在后面介绍；


## 测试及工具 - 系统测试 ##
### 目前还没有确定系统测试用场景，初步计划采用mesh网关，实现UE和Server的交互： ###

![](https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1926376896,4156363853&fm=26&gp=0.jpg)


## 测试及工具 - 抓包 ##
### Wireshark + Sniffer ###
现在已经使用IK-52832Dongle实现了抓包，可以参考下图

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.itdaan.com%2Fi%2Fc33189108bb299978d160405ff25e5d512.jpg&refer=http%3A%2F%2Fwww.itdaan.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1624698626&t=f69d4f7bc313d486ab1cb0a2615df601)

### Ellisys Bluetooth Analyzer + 蓝牙协议分析仪Ellisys Bluetooth Tracker ###
目前软件已经有了，需要采购蓝牙协议分析仪Ellisys Bluetooth Tracker

![](http://yiqi-oss.oss-cn-hangzhou.aliyuncs.com/aliyun/4695/Company/20180605-4608757175b15f9b6768cc.png)
![](https://img-blog.csdnimg.cn/20181113203755669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjU4MzE0Nw==,size_16,color_FFFFFF,t_70)


## 测试及工具 - PTS工具 ##
目前PTS软件 + LE dongle 已经具备，下面需要熟悉用例和测试方法

![](https://images2.pianshen.com/441/e6/e6dcddb4ba9a65c8da59d66a25a9a961.png)


## 测试及工具 - CEVA Test Tool ##
还未开始熟悉，CEVA提供了一套测试脚本，可以控制dongle和UE