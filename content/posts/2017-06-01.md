---
title: 使用 Python 操作 3G 模块
date: 2017-06-01T08:00:00+08:00
draft: false
toc:
comments: true
---



以 Telit HE910 模块为例，在 Linux 下的 AT 命令端口是 /dev/ttyACM3 。可以通过 pyserial 库直接读写端口来与模块通信，也可以使用更高级的 python-gsmmodem 。

## 1. Pyserial

如果要直接发送 AT 指令，需要串口读写库，我们常用的是 pyserial ，Python 2.7 默认没有安装这个库，需要自行下载，参考 <https://pypi.python.org/pypi/pyserial/2.7>，下面是一个简单的例子：

    #!/usr/bin/python
    import serial
    
    ser=serial.Serial(port='/dev/ttyACM3', baudrate=115200, bytesize=8, parity='N', stopbits=1, timeout=1, xonxoff=False, rtscts=False, dsrdtr=False)
    cmd="AT\r"
    ser.write(cmd.encode())
    msg=ser.read(64)
    print(msg)

保存到文件 atcommand.py ，执行：

    ~# ./atcommand.py   
    AT
    OK
    
## 2. Python-gsmmodem

### 2.1. 简介

python-gsmmodem 是一个用于控制 GSM modem 的 Python 包，基于 pyserial ，提供了 API 和一些命令行交互工具。在这里下载，然后安装：<https://github.com/faucamp/python-gsmmodem>。它的文档在源码的 docs 目录下，Sphinx 格式，你的电脑需要安装 Sphinx ，然后在 docs 目录下执行 `make html` 命令，就会在 _build 目录下生产 html 格式的文档，在浏览器中打开即可。python-gsmmodem 有如下几个特性：

* 提供了发送短信、查询信号强度等功能的简单方法。
* 提供了处理 USSD 业务和语音通话的 API 。
* 通过回调函数接电话、收短信。
* 支持短信的 PDU 和 Text 模式。
* 支持跟踪短信状态报告。
* 在 Python 的异常处理中封装了 AT 指令的错误信息。
* 直接向模块发送 AT 指令，读取返回值。
* 提供测试套件。

该模块提供了几个工具：

* GSMTerm ：一个简单的串口终端，用于与 GSM modem 通信，内建有 AT 指令帮助。启动后，输入 help 可以获取帮助信息，可以执行 AT 命令，还可以加载外部脚本：

        ~# gsmterm.py -b 115200 /dev/ttyACM3
        
        GSMTerm connected to /dev/ttyACM3 at 115200bps.
        Press CTRL+] or CTRL+D to exit.
        
        GSM> help
        
        == GSMTerm Help ==
        
        Command History: Press the up & down arrow keys to move backwards or forwards through your command history.
        
        Command Completion: Press the TAB key to provide command completion suggestions. Press the TAB key after a command is fully typed (with or without a "=" character) to quickly see its syntax.
        
        Command Documentation: Type a command, followed with two quesetion marks to access its documentation, e.g. "<COMMAND>??". Alternatively, precede the command with a question mark ("?<COMMAND>"), or type "help <COMMAND>".
        
        List Available Commands: Type "ls [category]" to list the available AT commands known to GSMTerm for the given category (or all commands if no category is specified).
        Type "lscat" to see a list of categories.
        
        Load Script: Type "load <filename>" to load and execute a file containing AT commands, separated by newlines, e.g. "load ./myscript.txt".
        
        To exit GSMTerm, press CTRL+] or CTRL+D.
        
        GSM> AT
        OK              
        GSM> AT+CSQ??
        
        Signal Quality (AT+CSQ)
        
        Category: Network Service
        Description: This command determines the received signal strength indication (<rssi>) and the channel bit error rate (<ber>) with or without a SIM card inserted
        Values: No parameters.
        Response Values: 
         <rssi> 0: -113 dBm or less
                1: -111 dBm
                2 to 30: -109 to -53 dBm
                31: -51dBm or greater
                99: not known or not detectable
         <ber> 0...7: as RXQUAL values in the table GSM 05.08
        Command Syntax:
         AT+CSQ

* sendsms.py ：发送短信的命令行脚本，执行 `` 可以获取帮助信息，下面是一个简单的例子：

        ~# sendsms.py -i /dev/ttyACM3 -b 115200 13824741490
        Connecting to GSM modem on /dev/ttyACM3...
        Checking for network coverage...
        
        Please type your message and press enter to send it:
        > Hello World!
        
        Sending SMS message...
        Message sent.
        
* identify-modem.py ：用于连接和调试 GSM modem 。

python-gsmmodem 主要提供了如下几个编程模块：

* serial_comms ，基于 pyserial ，提供了用于低级别串口通讯的类 SerialComms 。
* modem ，提供了几个操作 GSM modem 的类，例如 GsmModem ，SentSms 。
* exceptions ，提供了一些额外的方法，比如超时处理，中断处理，供 GsmModem 使用。
* pdu ，提供了对短信进行 PDU 编码的方法。

### 2.2. serial_comms

serial_comms 的主类是 SerialComms ，使用前先导入：

    from serial_comms import GsmModem
    

### 2.3. modem

modem 模块的主类是 GsmModem 类，它的父类是 SerialComms ，提供了与 GSM Modem 交互的大部分方法。使用前要先从 modme 模块导入：

    from gsmmodem.modem import GsmModem
        
GsmModem 类的构造函数：

    GsmModem(self, port, baudrate=115200, incomingCallCallbackFunc=None, smsReceivedCallbackFunc=None, smsStatusReportCallback=None)

参数的含义：

* port ，设置 AT 指令端口
* baudrate ，波特率
* incomingCallCallbackFunc ，处理接听电话的回调函数
* smsReceivedCallbackFunc ，处理接收短信的回调函数
* smsStatusReportCallback ，处理短信状态报告的回调函数

新建实例后，要先连接端口，初始化 GSM Modem：

    connect(self, pin=None)

这个函数会打开 AT 指令端口，初始化 GSM Modem 和 SIM 卡。参数 pin 用于设置 SIM 卡内的 PIN 码 ，字符串类型，通常 SIM 卡没有启动 PIN 码锁定，设为 None 即可。如果 SIM 卡需要 PIN 码而这里没有设置，函数会抛出 PinRequiredError 异常。如果 PIN 码设置错误，函数会抛出 IncorrectPinError 异常。相应的，结束时应该调用 `close()` 关闭串口，这是从 `SerialComms` 类继承的方法。

> 新购买的 SIM 卡背面提示中会写有 PIN 码初始值，也可以在运营商官网查询修改，通常是四位数字。PIN 码的作用是增强 SIM 卡安全性，例如可以在手机上为 SIM 卡启动 PIN 码锁定，这样每次开机都要输入 PIN 码。

初始化之后，应该确认 GSM Modem 已经成功接入运营商网络。可以调用 `waitForNetworkCoverage()` ，它会引起阻塞，直到成功入网，或者超时：

    waitForNetworkCoverage(self, timeout=None)
    
参数 timeout 可以设置超时时间，int 类型，单位是秒。超时会抛出 TimeoutException 异常，如果有其他错误，会抛出 InvalidStateException 。调用结束后会返回当前的信号强度。

还有一个单独的函数，可以获取信号强度：

    signalStrength(self)

返回值是整数，范围在 0~99 表示信号强度，-1 表示未知。

还有一些功能函数，可以获取 Modem 的一些状态，这些函数都是执行相关的 AT 指令，如果要执行其他 AT 指令，可以通过 SerialComms 类提供的 wirte 方法直接向 Modem 发送：

* manufacturer(self) ，返回 Modem 的厂家信息
* model(self) ，返回 Model 的名称
* revision(self) ， 返回 Modem 的固件版本
* imei(self) ，返回 Modem 的 IMEI 编码
* imsi(self) ，返回 SIM 卡的 IMSI 编码
* networkName(self) ，返回当前 Modem 所注册的网络的名称
* supportedCommands(self) ，返回 Modem 支持的 AT 指令的列表

设置短信编码的格式可以调用 `smsTextMode()` ：

    smsTextMode(self, textMode)

参数 textMode 是布尔型，设为 True 表示 text 格式，否则为 PDU 格式。text 格式是明文传输，不适合传输中文，国内目前都是 PDU 格式，初始化时，已经默认设为 PDU 格式。




发送文本类型的短信可以用 `sendSms()` 函数：

    sendSms(self, destination, text, waitForDeliveryReport=False, deliveryTimeout=15, sendFlash=False)
    
参数的含义：

* destination ，字符串类型，目标手机的电话号码
* text ，要发送的文本内容
* waitForDeliveryReport ，布尔型，如果设为 True ，函数会阻塞，知道收到短信状态的返回报告
* deliveryTimeout ，整数，或者浮点数，当 waitForDeliveryReport=True 时有效，表示阻塞超时时间。

如果发送过程过程中出现错误，会抛出 CommandError 异常。如果超时，会抛出 TimeoutException 异常。
