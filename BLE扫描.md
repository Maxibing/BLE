# 蓝牙协议 - 扫描 #
----------
* 介绍


## 介绍 ##
扫描是主机监听从机广播数据包和发送扫描请求的过程，主机通过扫描，可以获取`从机广播数据包`和`广播扫描回应数据包`。主机可以对已扫描到的从机设备发起连接请求，从而连接从机设备并通信。

* 扫描窗口和扫描间隔：

![](https://imgconvert.csdnimg.cn/aHR0cDovL2RvYy5pb3R4eC5jb20vaW1hZ2VzLzUvNWQvQkxFJUU2JThBJTgwJUU2JTlDJUFGXyVFNiU4OSVBQiVFNiU4RiU4RiVFNyVBQSU5NyVFNSU4RiVBMyVFNSU5MiU4QyVFNiU4OSVBQiVFNiU4RiU4RiVFOSU5NyVCNCVFOSU5QSU5NC5qcGc?x-oss-process=image/format,png#pic_center)

* 被动扫描：主机监听广播信道的数据，当接收到广播包时，协议栈将向应用层传递广播包；
* 主动扫描：除完成被动扫描动作外，还将向从机发送一个`扫描请求`，从机收到该请求，会再发送`扫描呼应数据包`；
