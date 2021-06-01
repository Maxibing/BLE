# 蓝牙协议 - 扫描 #
----------
* 介绍
* SCAN_REQ（扫描请求）
* SCAN_RSP（扫描响应）
* SCAN_REQ 报文分析


## 介绍 ##
扫描是主机监听从机广播数据包和发送扫描请求的过程，主机通过扫描，可以获取`从机广播数据包`和`广播扫描回应数据包`。主机可以对已扫描到的从机设备发起连接请求，从而连接从机设备并通信。

* 被动扫描：主机监听广播信道的数据，当接收到广播包时，协议栈将向应用层传递广播包；
* 主动扫描：除完成被动扫描动作外，还将向从机发送一个`扫描请求`，从机收到该请求，会再发送`扫描呼应数据包`；
* 扫描请求：SCAN_REQ，由链路层处于扫描态的设备发送，链路层处于广播态的设备接收；
* 扫描响应：SCAN_RSP，由链路层处于广播态的设备发送，链路层处于扫描态的设备接收；
* 扫描窗口：scan window，一次扫描进行的时间宽度；
* 扫描间隔：scan interval，两个连续的扫描窗口起始时间之间的时间差，包括扫描休息的时间和扫描进行的时间；

![](https://imgconvert.csdnimg.cn/aHR0cDovL2RvYy5pb3R4eC5jb20vaW1hZ2VzLzUvNWQvQkxFJUU2JThBJTgwJUU2JTlDJUFGXyVFNiU4OSVBQiVFNiU4RiU4RiVFNyVBQSU5NyVFNSU4RiVBMyVFNSU5MiU4QyVFNiU4OSVBQiVFNiU4RiU4RiVFOSU5NyVCNCVFOSU5QSU5NC5qcGc?x-oss-process=image/format,png#pic_center)


## SCAN_REQ（扫描请求） ##
扫描请求PDU载荷如下图所示，由ScanA（扫描设备地址）和AdvA（广播设备地址）组成，ScanA是扫描设备的公共或随机地址（由TxAdd确定），AdvA是广播设备的公共或随机地址（由RxAdd确定）。

![](https://images2015.cnblogs.com/blog/845932/201601/845932-20160120101728265-456533573.jpg)

广播报文中的TxAdd指示了扫描设备使用的是公共地址还是随机地址

* TxAdd = 0 -> 公共地址
* TxAdd = 1 -> 随机地址

广播报文中的RxAdd只是了扫描设备使用的是公共地址还是随机地址

* RxAdd = 0 -> 公共地址
* RxAdd = 1 -> 随机地址

## SCAN_RSP（扫描响应） ##
扫描响应PDU载荷如下图所示，由AdvA和ScanRspData（扫描响应数据）组成。

![](https://images2015.cnblogs.com/blog/845932/201601/845932-20160120102058781-1418681008.jpg)

广播报文的长度域指示了载荷的字节数（AdvA和ScanRspData）。


## SCAN_REQ 报文分析 ##
![](https://images2015.cnblogs.com/blog/845932/201601/845932-20160120102451703-1004326565.jpg)

    在分析数据之前，需要说明：广播包含扫描请求和扫描响应，所以扫描请求和扫描响应得包格式遵循广播包的格式。
    分析报文时，需要注意一下报文各个域的字节序。

* 完整报文：D6 BE 89 8E 83 0C 7F 0F 72 DD DF 68 DA B5 E9 D2 CC F3 BD BF 27

* 接入地址：D6 BE 89 8E，对广播来说是固定值，注意一下这里的字节序，接入地址传输时是低字节在前的，对应地址（0x8E89BED6）；
* PDU：
    * 广播报文报头：83 -> 1000 0011；
        * PDU Type：bit0~bit3 -> 0011 -> SCAN_REQ -> 扫描请求；
        * RxAdd：bit7 -> 1 -> 广播设备是随机地址；
        * TxAdd：bit6 -> 0 -> 扫描设备是公共地址；
    * 长度：0c -> 0000 1100； 
        * RFU: bit6~bit7 -> 0 -> 保留；
        * length：bit0~bit5 -> 00 1100 -> 12 -> 表示SCAN_REQ报文的长度是12个字节；
* 扫描设备的公共地址：7F 0F 72 DD DF 68
* 广播设备的随机地址：DA B5 E9 D2 CC F3
* 校验：BD BF 27

## SCAN_RSP 报文分析 ##
![](https://images2015.cnblogs.com/blog/845932/201601/845932-20160120103725703-1041514883.jpg)

* 完整报文：D6 BE 89 8E 44 06 DA B5 E9 D2 CC F3 61 6A 0F

* 接入地址：D6 BE 89 8E
* PDU：
    * 广播报文报头：44 -> 0100 0100;
    * PDU Type：bit0~bit3 -> 0100 -> SCAN_RSP -> 扫描响应；
    * TxAdd：bit6 -> 1 -> 广播设备使用的随机地址；
    * 长度：06 -> 0000 0110 -> 表示SCAN_RSP报文的长度是6个字节；
* 接入地址：DA B5 E9 D2 CC F3，广播设备的地址；
* 校验:61 6A 0F
