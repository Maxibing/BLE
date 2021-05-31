# 蓝牙协议 - BLE报文结构 #
---

* 报文结构
    * 字节序与比特序
    * 前导
    * 报头
        * 广播报文报头
        * 数据报文报头
    * 长度
    * 净荷（数据）
    * 循环冗余校验
    * AD Structure
        * AD Type
        * AD Data


## 报文结构 ##
BLE报文结构如下，由多个域组成。

![](https://images2.pianshen.com/374/28/281d006e1288b232255960254a68be7e.png "BLE报文格式")

### 字节序与比特序 ###
报文是一比特一比特发送的， 但它们同时又是由数据的字节组成。当数据的各个字节传输时，总是从最低位开始。因此，“0x80”会按照“00000001”发送，而“0x01”会按照“10000000”发送。

大多数字节域是从低字节开始发送，因此“0x010203”会按照“11000000 01000000 10000000”发送。

### 前导 ###
前导是一个8比特的交替序列，格式会根据接入地址的第一个比特而不同，具体如下:

    1. 若接入地址的第一个比特为0，前导为01010101；
    2. 若接入地址的第一个比特为1，前导为10101010；

接收机可以根据前导的无线信号强度来配置自动增益控制。

### 接入地址 ###
    1. 广播接入地址：固定为0x8E89BED6，在广播、扫描、发起连接时使用，之所以选择这个地址，是因为其相关性非常好；
    2. 数据接入地址：随机值，在连接建立之后的两个设备间使用；

对于数据信道，数据接入地址是一个随机值，但需要满足以下几个要求：

    1. 不能超过6个连续的0或1；
    2. 不能与广播地址相同；
    3. 4个字节的值必须互不相同；
    4. 不能有超过24次翻转（比特0到1/比特1到0称为1次翻转）；
    5. 最后6个比特需要至少两次比特翻转；
    6. 符合上面条件的有效随机接入地址约2^31个；

### 报头 ###
包头的内容取决于该报文是广播报文还是数据报文。

#### 广播报文报头 ####
* 广播报文报头包含4bit广播报文类型、2bit保留位、1bit发送地址类型和1bit接收地址类型：

![](https://images3.pianshen.com/582/fe/fe0f5933f3fbb75ea37b5ad2c4589a26.png "广播报文格式")

* 共有7种广播类型，如下图所示：

![](https://img-blog.csdn.net/201806151358060?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3YzE3MjU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70 "广播报文类型")

* 通用广播指示抓包：

![](https://img-blog.csdn.net/20160415221110726 "广播报文抓包")

* 发送/接收地址类型：发送/接收地址类型指示了设备使用公共地址（Public Address）还是随机地址（Random Address）。公共地址和随机地址的长度一样，都包含6个字节共48位。BLE设备至少要拥有这两种地址类型种的一种，当然也可以同时拥有两种。

    1. Public Address：公共地址，由制造商给从欧冠IEEE申请，由IEEE注册机构为该制造商分配的机构唯一表示OUI。这个地址是独一无二的，不能修改的；
    2. Random Address：随即地址，包含静态地址（Static Device Address）和私有地址（Private Device Address）；

#### 数据报文报头 ####
![](https://images3.pianshen.com/599/96/96a1d120cec4b02ea0906d6acfeec877.png "数据报文格式")

    1. LLID：链路标识符，表示此包数据是LL Data PDU还是LL Control PDU；
    2. MD：这个标志位是用来通知对方设备，自己是否还有其他数据要发送。0表示没有更多数据要发送，1表示有更多数据要发送。如果还有数据要发送，连接事件会自动扩展。
       一旦没有数据要发送，连接事件立刻关闭；
    3. SN：只有一个bit位，所以值是在0和1之间进行切换，如果SN与之前的一样，则位重传报文，否则是新报文；
    4. ENSN：预期SN，它是接收方希望解到的下一包的SN，也就是数据包的确认标志。当设备接收到SN=0的报文后，在发给对方的数据包中，应将ENSN设为1，
       这样对方接收到这个包后，就会发送一个新的数据包过来，否则就会重发上一次SN=0的报文。这个标志可以用来判断数据包是否被正确接收还是需要重传；

* 数据报文抓包：

![](https://img-blog.csdn.net/20160415222001579 "数据报文格式")

* 重传过程：

![](https://upload-images.jianshu.io/upload_images/1806858-e1b89fbc9dfde035.png?imageMogr2/auto-orient/strip|imageView2/2/w/669 "重传过程")

### 长度 ###
    1. 广播报文长度：该字段由6个bit组成，剩余2个bit留作未来使用，有效值范围是6~37；
    2. 数据报文长度：该字段由5给bit组成，深恶关于3个bit留作未来使用，有效值范围是0~31；

它们长度bit不同的原因是，广播报文多了6个字节的设备地址需要携带，因此需要6bit来标识长度。

### 净荷（数据） ###
最大传输的数据是31字节，但是如果数据被加密，需要留出4个字节进行消息完整性检查，那么实际的净荷被减少到27个字节。为了使得链路层设计得到简化，未加密报文的净荷也不允许超过27字节限制，以减低链路层缓存的复杂度。

MIC（ Message Integrity Check）：信息完整性检测。涉及到加密操作，图中是用虚线表示的，并不是一定要有此项。

在蓝牙5.0中，有效载荷增加到了255个字节。

### 循环冗余校验 ###
报文的最后是3个字节的CRC（循环冗余校验）。CRC对报头、长度域以及净荷域进行计算。24位的CRC强大到足以检测所有奇数位错误，以及所有2位或4位错误。这意味着低功耗蓝牙可以检测出所有的1/2/4/5/7/9比特错误。

### AD Structure ###
* 先看一下报文整体架构：

![](https://img-blog.csdnimg.cn/20190212193902497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lrMTUwOTE1,size_16,color_FFFFFF,t_70 "BLE 报文格式")


    1. AdvData：包含N个AD Structure；
    2. AD Structure：由Length、AD Type和AD Data组成；
    3. Length：AD Type和AD Data的长度；
    4. AD Type：指示AD Type的含义；

#### AD Type ####

![](https://images4.pianshen.com/793/0c/0c0a41c5ff837042a502b1a769277f71.png)

#### AD Data ####
* Flags:

![](https://images4.pianshen.com/767/42/420d75abe65b1418b85500b2c840e037.png)

* Service:

![](https://images1.pianshen.com/112/26/2619a46b551cff1dcaed93695491d968.png)

* Local Name:

![](https://images2.pianshen.com/458/74/7411de890bd5eb79d1b00e98ebe729da.png)

* TX Power Level:

![](https://images2.pianshen.com/427/e4/e4846aa2acf93b52dadce942b954c973.png)
