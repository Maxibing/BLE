# 蓝牙协议 - BLE广播 #
----------
* 介绍
* Advertising interval（广播间隔）
* Advertising_Type（广播类型）
* Own_Address_Type（自身地址类型）
* Direct_Address_Type（定向地址类型）
* Direct_Address（定向地址）
* Advertising_Channel_Map（广播信道）
* Advertising_Filter_Policy（广播过滤策略）
* Advertising And ScanResponse Data（广播和扫描回应数据）


## 介绍 ##
低功耗蓝牙设备通过广播信道发现其他设备，一个设备进行广播，而另一个设备进行扫描。当从机处于广播态时才能被主机发现，广播包会在37/38/39三个信道上依次广播。

## Advertising interval（广播间隔） ##
设备每次广播时，会在3个广播信道上发送相同的报文，这些报文被称为一个广播事件。除了定向广播以外，其他广播事件均可以选择20ms-10.28s不等间隔。通常，一个广播中的设备会每一秒广播一次。两个相邻广播事件之间的时间称为广播间隔。

但是，设备周期性的发送广播会有一个问题：由于设备间的时钟会不同程度的漂移，两个设备可能在很长一段时间同时广播而造成干扰。为了防止这一情况的发生，除定向广播之外的其他广播类型，发送时间均会被扰动。实现该扰动的方式为，在上一次广播事件后加入0-10ms的随机延时。这意味着，即使两个设备广播间隔相同，并在相同信道及时间点上发送造成了冲突，但它们发送下一个广播事件时也会有很大可能不再冲突。

所以相邻两个广播事件之间的事件间隔为：T_AdvEvent = advInterval + advDelay

其中，advInterval必须是0.625ms的整数倍，范围是20ms-10.24s之间。对于可扫描非定向广播和不可连接非定向广播这两种广播类型，该值最好不小于100ms（即16个0.625ms）。advDelay是Link Layer（链路层）分配的一个伪随机数，它的范围是0-10ms。
![](https://img-blog.csdn.net/20160415232643607)

蓝牙4.x，广播有效载荷最多是31个字节。而在蓝牙5.0中，通过添加额外的广播信道和新的广播PDU，将有效载荷增加到了255个字节

当然，实际设置过程中没有广播间隔参数，而是设置Advertising_Interval_Min（最小广播间隔）和Advertising_Interval_Max（最大广播间隔）这两个参数来调整广播间隔。如果要固定广播间隔为某一个值，只需要将这两个参数设置为同一个有效数值即可。

## Advertising_Type（广播类型） ##

![](https://img-blog.csdn.net/201806151358060?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3YzE3MjU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70 "广播报文类型")

1. 可连接的非定向广播：Connectable Undirected Event Type（ADV_IND），这是一种用途最广的广播类型，包括广播数据和扫描响应数据，它表示当前设备可以接受其他任何设备的连接请求。
<<<<<<< HEAD
2. 
=======
<<<<<<< HEAD
2. 
=======

    这种广播类型用的最多，需要注意，在一个广播事件中，前一个“ADV_IND_PDUs”的开始到相邻下一个“ADV_IND_PDUs”的开始处，间隔时间要小于等于10ms。

    第一种情况，在广播事件中，只有PDUs：
    
    ![](https://img-blog.csdn.net/20160419152234398)

    第二种情况，在广播事件的中间有"SCAN_REQ"和"SCAN_RSP"PDUs：
    ![](https://img-blog.csdn.net/20160419153006588)

    第三种情况，在广播事件的结尾有"SCAN_REQ"和"SCAN_RSP"：
    ![](https://img-blog.csdn.net/20160419153011231)

    第四种情况，在广播事件的中间接收到"CONNECT_REQ"PDU的情况：
    ![](https://img-blog.csdn.net/20160419153016778)


2. 可连接的定向广播：Connectable Directed Event Type（ADV_DIRECT_IND）,定向广播类型是为了尽快建立连接。这种报文包含2个地址，广播者的地址和发起者的地址。发起者收到发给自己的定向广播报文之后，可以立刻发送连接请求作为回应。

    定向广播类型有特殊的时序要求。完整的广播事件必须3.75ms重复一次，这一要求使得扫描设备只需要扫描3.75ms便可以收到定向广播设备的消息。

    这么快的发送会让报文充斥着广播信道，进而导致该区域内的其他设备无法进行广播。因此，定向广播不可持续1.28s以上的时间。如果主机没有主动要求停止，或者连接没有建立，控制器会自动停止广播。一旦到了1.28s，主机便只能使用ADV_IND广播，让其它设备来连接。

    当使用定向广播时，设备不能主动扫描。此外，定向广播中的净荷只能包含两个必须的地址，不能带有其他数据。

3. 不可连接的非定向广播：Non-connectable Undirected Event Type（ADV_SCAN_IND），仅仅发送广播数据。
4. 可扫描的的非定向广播：Scannable Undirected Event Type（ADV_SCAN_IND），这种广播不能用于发起连接，但允许其他设备扫描该广播设备。这意味着该设备可以被发现，既可以发送广播数据，也可以响应扫描发送扫描回应数据，但不能建立连接。这是一种适用于广播数据的广播形式，动态数据可以包含于广播数据中，而静态数据可以包含于扫描响应数据中。

所谓的定向和非定向针对的是广播的对象，如果是针对特定的对象进行广播（在广播PDU中会包含目标对象的MAC）就是定向广播，反之就是非定向的。

而可连接和不可连接是指是否接受连接请求，如果是不可连接的广播类型，它将不回应连接请求。可扫描广播类型是指回应扫描请求。

![](https://img-blog.csdn.net/20160416101401758)


## Own_Address_Type（自身地址类型） ##
![](https://img-blog.csdn.net/20160416101841517)

Public Device Address：共有设备地址是设备所持有的并且不可改变的。类似网络设备的MAC地址，它的长度是48位，由两部分组成：
![](https://img-blog.csdn.net/20160416101936979)

Random Device Address：随机设备地址（私有设备地址），它也是48位，组成如下：
![](https://img-blog.csdn.net/20160416102035292)


## Direct_Address_Type ##
定义与Own_Address_Type相同


## Direct_Address ##
定向设备地址如下：
![](https://img-blog.csdn.net/20160416102351137)


## Advertising_Channel_Map（广播信道） ##
广播信道如下：
![](https://img-blog.csdn.net/20160416102515228)

在一个广播事件中，一个广播包会在每个信道上进行传输。显示如下：
![](https://img-blog.csdn.net/20160416102813670)


## Advertising_Filter_Policy（广播过滤策略） ##
广播过滤策略，对发来请求包的设备采用的过滤策略，如下所示：
![](https://img-blog.csdn.net/20160416102553013)

    1. value 0x00：接收任何设备的的扫描请求或连接请求；
    2. value 0x01：接收白名单中特定设备的扫描请求，但是接收任何设备的连接请求；
    3. value 0x02：接收任何扫描请求，但只接受白名单中特定设备的接入请求；
    4. value 0x03：只接收白名单中的特定设备的扫描请求和接入请求；
    5. value 0x04 - 0xFF：保留；


## Advertising And ScanResponse Data（广播和扫描回应数据） ##
广播数据和扫描回应数据，它们的长度都不能超过31字节(0~31)，数据的格式必须满足下图的要求，可以包含多个AD数据段，但是每个AD数据段必须由"Length: Data"组成，其中Length占用1个octet，Data部分占用Length个字节，所以一个AD段的长度为Length+1，如下图所示：
![](https://img-blog.csdn.net/20160416103047452)
>>>>>>> 07f902d... Add BLE Advertising and scan
>>>>>>> 470d696 (Add advertising and scan)
