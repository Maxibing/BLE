# BLE协议栈学习1 -介绍 #
---
* 协议框架
* Physical Layer
* Link Layer
    * 功能介绍
    * 解决共享问题
        * 数据量比较少、发送不频繁、对时延不是很敏感的场景
        * 数据量较大、发送频率较高、对时延敏感的场景
    * 状态（state)和角色（role）的定义
    * Air Interface Protocol
        * 定义在Physical Channel上收发的数据包的格式（Packet Format）
        * 定义不同类型的PDU及其格式
        * 以白名单（White List）的形式定义Link Layer的数据过滤机制
        * 执行广播信道上实际的packet收发操作
        * 定义连接建立的方式及过之后的应答、流控等机制
    * Link Layer Control
* HCI
* L2CAP
    * 功能介绍
    * Protocol/Channel Multiplexing
        * 基于连接的方法
        * 无连接的方法
* Attribute Protocol
* Generic Attribute Profile
    * 功能介绍
    * 层次结构
* Security Manager（SM）
* Generic Access Profile（GAP）


## 协议框架 ##
BLE的协议可分为Bluetooth Application和Bluetooth Core两大部分，而Bluetooth Core又包括BLE Control和BLE Host两部分。

![](http://www-x-wowotech-x-net.img.abc188.com/content/uploadfile/201603/572c1f6e9ab5a5855251ac38ca3890be20160319143211.gif "LE协议框架")


## Physical Layer ##
任何一个通信系统，首先要确认的就是通信介质（物理通道，Physical Channel），BLE也不例外。在BLE协议中，“通信介质”的定义是由Physical Layer负责。
<br />
Physical layer是这样描述BLE通信介质的：

    1. 由于BLE属于无线通信，其通信介质是一定频率范围下的频带资源（Frequency Band）；
    2. BLE市场定位是个体和民用，因此使用免费的ISM频段（频率范围是2.400 - 2.4835 GHz）；
    3. 为了同时支持多个设备，将整个频带分为40份，每份的带宽为2MHz，称为RF Channel；

根据上面的定义，频点`f=2.402GHz + k*2MHz, k=0,...39(2.402GHz - 2.480GHz)`，可以看到在较低频率部分，与2.4GHz ISM最低频率有2MHz是间隙，而在较高的频率部分，有3.5MHz的间隙。

如下图，0~36是数据通道，37~39是广播信道，这样分布是为将广播通道避开wifi的1、6、11信道。

![](https://img-blog.csdnimg.cn/20181128105740913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4ODc3MTI1,size_16,color_FFFFFF,t_70 "信道映射")


## Link Layer ##
### 功能介绍 ###
Link Layer的主要功能就是在Physical Channel上收发数据，控制RF收发相关的参数。除此之外，Link Layer需要解决下面几个问题：

    1. 首先，Physical Layer只提供了40个Physical Channel，而BLE中参与通信的实体的数量，肯定不是这个数量级。Link Layer需要解决Physical Channel的共享问题；
    2. 其次，通信是两个实体之间的事情，对这两个实体来说，它们希望看到一条为自己独享的传输通道（logic link，逻辑链路），这也是Link Layer需要解决的；
    3. 再则，Physical Layer是不可靠的，任何数据传输都可能由于干扰等问题而损毁、丢失，这些对于应用来说是不可接受的。因此Link Layer需要提供校验，重传等机制，确保数据传输的可靠性；


### 解决共享问题 ###
Link Layer将BLE的通信场景分为两类。

#### 数据量比较少、发送不频繁、对时延不是很敏感的场景 ####

针对这种场景，BLE采用广播通信的方式：

    1. 40个Physical Channel中选取3个，作为广播通道（advertising channel）；
    2. 在广播信道上，任何参与者，爱发就发，爱收就收，随便；
    3. 所有参与者，共享一个逻辑传输通道（广播通道）；

广播方式存在很多问题，例如

    1. 不可靠、相互干扰、传输不可靠；
    2. 效率低，一个区域多个节点在广播数据，接收者需要接收所有数据，不管是不是它关心的；
    3. 没有安全保障；

针对上面的问题，Link Layer定义了一些策略，尽可能的提高广播的便利性，至于无法解决的，可以选择连接的方式；

#### 数据量较大、发送频率较高、对时延敏感的场景 ####
BLE会从剩余的37个Physical Channel中，选取一个，为这种场景里面的通信双方提供建立单独的通道（data channel)，这就是连接（connection）的过程。

同时，为了增加容量，增大抗干扰能力，连接不会长期使用一个固定的Physical Channel，而是在多个Channel（如37个）之间随机但有规律的切换，这就是跳频（Hopping）技术。

### 状态（state)和角色（role）的定义 ###
基于上面解决共享问题的思路，BLE协议在Link Layer抽象出5种状态（协议每层都可能有状态定义，有的名字是相同的，需要区分哪一层）

    1. Standby state, 就绪态，这个状态是初始状态，不发也不收数据。根据上层实体的命令（位于host中的GAP），可由其他任何一种状态进入，
      也可以切换到除connection态的任意一种状态；
    2. Advertising State，广播态，可以通过广播信道发送数据的状态，由Standby态进入。它广播的数据可以由处于Scanning或Initialing态的实体接收。
      上层实体可以通过命令将Advertising状态切换回Standby态。另外，连接成功后，可切换为connection态；
    3. Scanning State，监听/扫描态，可以通过广播通道接收数据的状态，由Standby态进入。根据Advertiser所广播的数据类型，
      有些Scanner还可以主动向Advertiser请求一些额外数据。上层实体通过命令将Scanning状态切回Standby状态；
    4. Initialing State，发起态，这是一种特殊的接收状态，由Standby状态进入，只能接收Advertiser广播的connectable的数据，
      并在接收到数据后，发送连接请求，以便和Advertiser建立连接。当连接建立成功后，Initialer和对应的Advertiser都会切换到Connection状态；
    5. Connection State，连接态，是和某个实体建立单独通道后的状态，在通道建立后，由Initialer或者Advertiser自动切换而来。
      通道断开后，会重新回到Standby状态；

通道建立后（已连接），处于connection状态的双方，分别有两种角色Master和Slave：

    1. Initialer成为Master；
    2. Advertiser成为Slave；

五种状态的转换关系如下图：

![](https://imgconvert.csdnimg.cn/aHR0cDovL2Jsb2cuY2hpbmF1bml4Lm5ldC9hdHRhY2htZW50LzIwMTYwMy8zMS8yMTQxMTIyN18xNDU5NDE2Mjg2ejhIay5wbmc?x-oss-process=image/format,png)

### Air Interface Protocol ###
状态和角色定义完成后，剩下需要完成两件事：

1. 提供某一状态下，和其他实体对应状态之间的数据交换机制；
2. 根据上层实体的指令，以及当前的实际情况，负责状态之间的切换；

BLE协议中，这些事情都是由一个叫做空中接口协议（Air Interface Protocol）负责。

#### 定义在Physical Channel上收发的数据包的格式（Packet Format） ####
在BLE，advertising channel和data channel统一使用同一种packet format（PDU不同）。
![](https://img-blog.csdnimg.cn/20190212192825497.png "数据链路层报文结构")

#### 定义不同类型的PDU及其格式 ####
##### Advertising channel中Advertising有关的PDU： #####

    1. ADV_IND，Advertiser发送的、可被连接的、无方向的广播数据（connectable undirected advertising event）；
    2. ADV_DIRECT_IND，Advertiser发送的、可被连接的、单向广播数据（connectable directed advertising event）；
    3. ADV_NONCONN_IND，Advertiser发送的、不可被连接的，无方向的广播数据（non-connectable undirected advertising event）；
    4. ADV_SCAN_IND，Advertiser发送的、可接受SCAN_REQ请求的、无方向的广播数据（scannable undirected advertising event）；


##### Advertising channel中Scanning有关的PDU： #####
    1. SCAN_REQ，Scanner发送的、向Advertiser请求额外信息的数据包（一般需要在收到ADV_SCAN_IND后才可发送）；
    2. SCAN_RSP，Advertiser发送的、响应SCAN_REQ请求的数据包；

##### Advertising channel中Initialing有关的PAD： #####
    1. CONNECT_REQ，Initialer发起的、请求建立连接的数据包；

##### Data channel中LL data有关的PDU： #####
已连接双方进行数据通信所用的PDU，有效的payload长度为0~251bytes；

##### Data channel中LL control有关的PDU： #####
用于管理、维护、更新已连接的数据通道的PDU

    1. LL_CHANNEL_MAP_REQ，请求更改所使用的Physical Channel的数据包；
    2. LL_TERMINATE_IND，告知对方此次连接即将结束，以及结束的原因；

##### 广播通道的数据包格式： #####

![](https://img-blog.csdnimg.cn/2019021219271134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lrMTUwOTE1,size_16,color_FFFFFF,t_70 "广播通道的 PDU 格式")

##### 数据通道的数据包格式： #####

![](https://img-blog.csdnimg.cn/20190212193138478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lrMTUwOTE1,size_16,color_FFFFFF,t_70 "数据通道的数据包格式")

#### 以白名单（White List）的形式定义Link Layer的数据过滤机制 ####
主要针对广播信道，因为随着通信设备的增多，空中的广播数据将呈几何级的增长，为了避免资源的浪费（特别是BLE Host），有必要在Link Layer过滤掉一些数据包，例如根据蓝牙地址，过滤掉不是给自己的packet；

#### 执行广播信道上实际的packet收发操作 ####
上层软件只需要定义一些参数，例如：

    1. Advertising State下的Advertising Channel的选择、间隔、PDU类型等；
    2. Scanning State/Initialing State下的ScanWindow、ScanInterval等；

#### 定义连接建立的方式及过之后的应答、流控等机制 ####


### Link Layer Control ###
经过Air Interface Portocol的抽象，BLE实体已经具备广播通信、点对点连接的建立和释放、点对点通信等基本能力。除此之外，Link Layer又抽象出一个链路控制协议（Link Layer Control），用于管理、控制两个Link Layer实体之间所建立的这个Connection，主要功能包括：

    1. 更新Connection相关的参数，如transmitWindowSize、transmitWindowOffset、connInterval等；
    2. 更新该连接所使用的调频图谱（使用哪些Physical Channels）；
    3. 执行链路加密（Encryption）有关的过程；


## HCI ##
定义Host和Controller之间的通信协议，可基于uart、USB等物理介质，对理解蓝牙协议来说，是无关紧要的，这里暂不介绍；

## L2CAP ##
### 功能介绍 ###
经过Link Layer的抽象后，两个BLE设备之间可存在两条逻辑上的数据通道：

    1. 无连接的广播通道；
    2. 基于连接的数据通道，是一个点对点（Master - Slave）的逻辑通道，成为Logic Channel；

其中逻辑通道的使用，有以下几个问题需要考虑：

    1. Logic Channel只有一条，而要利用它传输数据的上层应用却不止一个，怎么复用？
    2. Logic Channel所能传输的有效payload最大只有251bytes，上层应用要传输大于这个长度的数据怎么办？
    3. Logic Channel仅提供了简单的应答和流控机制，如果传输的数据出错怎么办？

以上问题都是由L2CAP，一个介于应用程序（Profile、Application）和Link Layer之间的protocol负责。

L2CAP(Logic link Control and Adaptation Portocol)提供的主要功能包括：

    1. Protocol/Channel Multiplexing，协议/通道的多路服用；
    2. Segmentation and reassembly，上层应用数据（L2CAP Service Data Units, SDUs）的分割（和重组），生成协议数据单元（L2CAP Packet Data Units, PDUs)，
       以满足用户数据传输时延的要求，以便于后续的重传、流控等机制的实现；
    3. Flow control per L2CAP channel，基于L2CAP Channel的流控机制；
    4. Error control and retransmissions，错误控制和重传机制；
    5. Support for Streaming，支持流式传输（如音频、视频等，不需要重传或者只需要优先重传）；
    6. Fragmentation and Recombination，协议数据单元（PDUs）的分片（和重组），生成符合Link Layer传输要求的数据片；
    7. Quality of Service，QoS的支持；

### Protocol/Channel Multiplexing ### 
这里先介绍Protocol/Channel Multiplexing功能，有利于理解上层协议（ATT、GATT等），其他部分在后续介绍。

可用于传输用户数据的逻辑链路只有一条，而L2CAP需要服务的上层Profile和Application的数目，肯定不止这个数量。因此，需要使用多路复用的手段，将这些用户数据map到有限的链路资源上去，这就是Multiplexing，多路复用；

Multiplexing的方法：

    1. 数据发送时，将用户数据分割为一定长度的数据包（L2CAP Packet Data Units, PDUs)，加上一个特定“ID”的header后，通过逻辑链路发送出去；
    2. 数据接收时，从逻辑链路接收数据，解析其中的“ID”，并以此判断需要将数据转发给哪个应用；

这里所说的“ID”就是多路复用的方法，分为基于连接的方法和无连接的方法。
#### 基于连接的方法 ####
这里的连接是L2CAP connection（需要和Link Layer的connection区分开），是指基于L2CAP的应用程序，在通信之前，先建立一个基于Logic Channel的虚拟通道（称作L2CAP Channel，和TCP/IP中的端口类似）。L2CAP会为这个通道分一个编号，称作Channel ID（简称CID）。

L2CAP channel建立之后，就可以把CID放到数据包的header中，以达到multiplexing的目的。这些基于CID实现的多路复用，就成为channel multiplexing（基于通道的多路复用）。

有一些固定用途的L2CAP Channel，其CID是固定值，另外一些则是动态分配的，具体如下：
![](http://www.wowotech.net/content/uploadfile/201603/71aaf224eacfe7636162f9bd7aa6e07220160312143241.gif "BLE CID 分配")

有关CID具体数值可参考：
[Core_v4.2.pdf](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=286439&_ga=1.139498554.805272089.1452604944)
, Volume 3, Part A - Logical Link Control and Adaptation Protocol Specification

#### 无连接的方法 ####
为了提高数据传输的效率，方便广播通信等应用场景，L2CAP在提供基于连接的通信机制外，也提供了无连接的数据传输方法。基于这种方法，CID不存在了，取而代之的是一个称作Protocol/Service Multiplexing(PSM)的字段。

这种多路复用的手段则称为Protocol Multiplexing（基于协议的多路复用）。

这种方法只允许在BR/EDR controller中使用，这里不做介绍。


## Attribute Protocol ##
由上面介绍可以知道，在BLE协议栈中：

    1. Physical Layer负责提供一系列的Physical Channel；
    2. 基于这些Phsical Channel，Link Layer可在两个设备之间建立用于点对点通信的Logic Channel；
    3. 而L2CAP则将这个Logic Channel划分为一个个的L2CAP Channel，以便提供应用程序级别的通道复用。
       到此之后，基本协议栈已经构建完毕，应用程序已经可以基于L2CAP运行起来了；

谈起应用程序，就不得不说BLE的初衷 -- 物联网。物联网中传输的数据和传统的互联网不同，物联网最重要、最广泛的一类应用是：信息的采集。
>这些信息往往都很简单，如温度、湿度、速度、位置信息、电量、水压等等。

>采集的过程也很简单，节点设备向中心设备汇报信息数据，或者中心设备在需要的时候主动查询；

基于信息采集的需要，BLE抽象出一个协议：Attribute Protocol，该协议将这些“信息”以“属性（Attribute）”的形式抽象出来，并提供一些方法，供远端设备（remote device）读取、修改这些属性的值（Attribute value）。

Attribute Protocol的主要思路包括：

1. 基于L2CAP，使用固定的Channel ID（0x004，可参考上图CID介绍）；


2. 采用client-server形式。提供信息（后面都成为attribute）的一方称作ATT server（一般是那些传感器节点），访问信息的一方称作ATT client；


3. 一个Attribute由Attribute Type、Attribute Handle和Attribute value组成；

    * Attribute Type用于标示Attribute的类型，类似于我们常说的“温度”、“湿度”等，使用UUID（Universally Unique IDentifier, 16-bit、32-bit或128-bit的数值）进行区分；
    * Attribute Handle是一个16-bit的数值，用作唯一识别ATT server上的所有Attribute。Attribute Handle的存在有如下意义：
        * 一个server上可能存在多个相同type的Attribute，显然，client有区分这些Attribute的需要；
        * 同一类型的多个attribute，可以组成一个group，client可以通过这个Group中的起、始handle访问所有的attributes；
    * Attribute value代表Attribute的值，可以是任何固定长度或可变长度的octet array（理解为字节类型数组即可）；

4. Attribute可以定义一些权限（Permissions），以便server控制client的访问行为，包括：

    * 访问有关的权限（access permissions)，Readable、Writeable以及Readable and Writeable；
    * 加密有关的权限（encryption permissions），Encryption required和No encryption required；
    * 认证有关的权限（authentication permissions），Authenticaiton Required 和 No Authentication Required；
    * 授权有关的权限（authorization permissions），Authorization Required和No authorization Required；

5. 根据所定义的Attribute PDU的不同，client可以对server有多种访问方式，包括：

    * Find Information，获取Attribute type和Attribute Handle的对应关系；
    * Reading Attributes，有Read by type、Read by handle、Read by blob（只读取部分信息）、Read Multiple（读取多个handle和type）等方式；
    * Writing Attribute，包括需要应答的writing和不需要应答的writing；


## Generic Attribute Profile ##
### 功能介绍 ###
ATT之所以称作Protocol，是因为它还比较抽象，仅仅定义了一套机制，允许client和server通过attribute的形式共享信息。而具体共享哪些信息，ATT并不关心，这是GATT（Generic Attribute Protocol)的工作。

GATT相对ATT只多了一个“G”，但含义却大不相同，因为GATT是一个profile（更准确说是profile framework）；

在蓝牙协议中，profile一直是一个比较抽象的概念，我们可以把它理解为“应用场景、功能、使用方式”都被规定好的Application。BLE很大一部分的应用场景是信息（Attribute）的共享，因此BLE协议栈基于Attribute Protocol，定义了一个称作GATT的profile framework，用于提供通用的、信息的存储和共享功能。

### 层次结构 ###
作为一个profile framework，GATT profile提出了如下层次结构：

![](http://www.wowotech.net/content/uploadfile/201603/c30a59962849cad4ab665c0532ea6c3520160317144849.gif "GATT Profile hierarchy")

由上图可知，GATT profile的层次结构依次是：Profile - Service - Characteristic。

    1. “Profile”是基于GATT所派生出的真正的profile，位于GATT profile hierarchy的最顶层，由一个或多个和某一应用场景有关的Service组成；
    2. 一个Serive包含一个或多个Characteristic（特征），也可以通过Include的方式，包含其他Service；
    3. Characteristic则是GATT profile最基本的数据单位，由一个Properties、一个Value、一个或多个Descriptor组成；
    4. Characteristic Properties定义了characteristic的Value如何被使用，以及Characteristic Descriptor怎么被访问；
    4. Characteristic value是特征的实际值，例如一个温度特征，其Characteristic Value就是温度值；
    5. Characteristic Descriptor则保存了一些和Characteristic Value相关的信息（比较抽象，后续介绍）；

以上除了Profile外的每一个定义，Service、Characteristic、Characteristic Properties、Characteristic value和Characteristic Descriptor等等，都是作为一个个Attribute存在的，具备Attribute的所有特征：Attribute 
Handle、Attribute Types、Attribute Value和AttributePermissions。

基于GATT架构，Application的存在形式，不再是单一的Profile，很多简单的应用场景，可以直接以Servive的形式存在（Profile的概念隐藏在了GATT中），这大大简化了一些传感器节点的设计复杂度；


## Security Manager（SM） ##
Security Manager负责BLE通信中有关安全的内容，包括配对（pairing）、认证（authentication）和加密（encryption）等过程。

## Generic Access Profile（GAP） ##
上面的介绍都是和基于连接的data channel有关，至于无连接的advertising channel，以及连接建立的过程都没有涉及。虽然Link Layer已经做出了定义，但它们没有提现到Applicaiton（或者Profile）层面，毕竟Link Layer太底层。

因此BLE协议栈定义了一个称作Generic Access（通用访问）的Profile，以实现如下功能：
* 定义GAP层的蓝牙设备角色（role）：

    1. 和Link Layer的role类似，只不过GAP层的role更接近用户（可以等同于从用户角度看到的蓝牙设备的role；
    2. Broadcast Role，设备正在发送advertising events；
    3. Observe Role，设备正在接收advertising events；
    4. Peripheral Role，设备接受Link Layer连接（对应Link Layer的slave角色）；
    5. Centric Role，设备发起Link Layer连接（对应Link Layer的master角色）；

* 定义GAP层的、用于实现各种通信的操作模式（Operational Mode）和过程（Procedures），包括：

    1. Broadcast mode and observation procedure，实现单向的、无连接的通信方式；
    2. Discovery modes and procedures，实现蓝牙设备的发现操作；
    3. Connection modes and procedures，实现蓝牙设备的连接操作；
    4. Bonding modes and procedures，实现蓝牙设备的配对操作；

* 定义User Interface有关的蓝牙参数，包括：

    1. 蓝牙地址（Bluetooth Device Address）；
    2. 蓝牙名称（Bluretooth Device Name）；
    3. 蓝牙的pincode（Bluetooth passkey）；
    4. 蓝牙的class（Class of Device，和发射功率有关）；
    5. 等等；

* Security有关的定义，暂不介绍；


## Application ##
暂不介绍