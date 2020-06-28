---
title: Paho Python Client 学习笔记
date: 2017-05-23T08:00:00+08:00
draft: false
toc:
comments: true
---



Paho 是一个开源的 MQTT 客户端项目，提供多种语言的 MQTT 客户端实现，包括 C、C++、C#、Java、Python、JavaScript 等，完全支持 MQTT v3.1 和 v3.1.1 。Paho Python Client 是它的 Python 语言版本，支持 Python 2.7 和 3.x 。更多特性可以查看 <http://www.eclipse.org/paho/clients/python/> ，源码和文档在 <https://github.com/eclipse/paho.mqtt.python> 。

该项目提供了一个测试用的 MQTT broker ：`iot.eclipse.org` ，端口 1883 ，无密码。

## 1. 安装

在 Python 环境中用 `pip install paho-mqtt` 命令安装，或者下载源码：

    git clone https://github.com/eclipse/paho.mqtt.python.git
    cd org.eclipse.paho.mqtt.python.git
    python setup.py install

下面是一个简单的例子，连接一个 borker ，订阅系统默认话题，获取 broker 的版本号：

    import paho.mqtt.client as mqtt
    
    def on_connect(client, userdata, flags, rc):
        print("Connected with result code "+str(rc))
        client.subscribe("$SYS/broker/version")
    
    def on_message(client, userdata, msg):
        print(msg.topic+" "+str(msg.payload))
    
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    
    client.connect("iot.eclipse.org", 1883, 60)
    
    client.loop_forever()

保存到 paho-mqtt.py 文件，执行：

    $ python paho-mqtt.py
    Connected with result code 0
    $SYS/broker/version mosquitto version 1.4.10
    
## 2. 编程

paho.mqtt 包提供了三个类，Client、Publish 和 Subscribe。Publish 和 Subscribe 提供了简单的方法，一次性的发送或者接收消息，不会保持连接。Client 包含了新建客户端、连接、订阅、发送、回调函数等方法。通常的编程步骤是新建一个 Client 的实例，然后调用它提供的连接、发布和订阅等方法与 broker 通讯：

1. 新建一个 Client 实例
2. 用一个 `connect*()` 函数连接 broker 
3. 用一个 `loop*()` 函数，维持与 broker 的连接
4. 用 `subscribe()` 函数订阅一个话题，接收消息
5. 用 `publish()` 函数发布消息
6. 用 `disconnect()` 函数断开连接

下面主要介绍 Client 提供的方法，使用前先导入：
    
    import paho.mqtt.client as mqtt
        
### 2.1. 初始化

新建一个 Client 实例：

    Client(client_id="", clean_session=True, userdata=None, protocol=MQTTv311, transport="tcp")
    
这是 Client 类的构造函数，参数的含义：

* client_id ，设置客户端的 ID ，应该是一个字符串，连接时向 broker 提交。如果为空，会随机生成一个 id ，此时，clean_session 必须设为 True 。
* clean_session ，布尔型，如果为 True ，断开连接时，broker 会清除关于这个 client 的所有信息。如果为 False ，断开连接时，broker 会保留这个客户端的订阅信息和消息队列。
* userdata ，用户自定义的数据，可以是任何类型，传递给回调函数。可以用 `user_data_set()` 函数更新。
* protocol ，设置 MQTT 协议的版本，`MQTTv31` 或者 `MQTTv311` 。
* transport ， 传输协议，默认还是 `tcp` ，可以设为 `websockets` 。

构造实例：

    import paho.mqtt.client as mqtt
    mqttc = mqtt.Client()

可以调用 `reinitialise()` 重新初始化 Client ：

    reinitialise(client_id="", clean_session=True, userdata=None)

### 2.2. 配置

这些函数用来设置 Client 的一些特性，通常在连接 broker 之前调用。

    max_inflight_messages_set(self, inflight)
    
这个函数可以设置当 QoS>0 时，最多可以存在几条动态消息（已经发送，还没有确认成功的消息）。默认是 20 ，增加这个值会占用更多的内存，但是可以提升吞吐量。

    max_queued_messages_set(self, queue_size)
    
这个函数可以设置当 QoS>0 时，发送消息队列的最大值，默认是 0 ，表示无限制。当队列满时，旧消息会丢弃。

    message_retry_set(retry)

当 Qos>0 时，如果发送消息后超过一定时间还没有收到确认报文，就要重发消息，这个函数用于设置超时时间，单位是秒。默认是 5 秒，通常不用修改。

    
配置 SSL 证书验证的函数，必须在 `connect*()` 函数之前调动。几个参数的含义：

* ca_certs ，指定 CA 根证书的路径。
* certfile,keyfile ，指定客户端私钥和证书的路径。
* cert_reqs ，设置客户端对 broker 证书的需求，默认是 ssl.CERT_REQUIRED ，表示 broker 必须提供一个证书。
* tls_version ，设置 SSL/TLS 协议的版本，默认是 TLS v1 。
* ciphers ，设置本次连接的加密密码，默认是 None 。

设置用户名和密码：

username_pw_set(username, password=None)

设置遗嘱：

    will_set(topic, payload=None, qos=0, retain=False)
    
当这个 client 断开连接时，broker 会发布这个遗嘱消息。参数的含义：

* topic ，遗嘱消息的话题
* payload ，遗嘱消息的内容，字符串类型，如果设为 None ，会发送一条长度为 0 消息。如果设置了 int 或者 float 类型的值，会当做字符串发送，如果你想发送真正的 int 或者 float 值，需要用 `struct.pack()` 生成消息。
* qos ，遗嘱消息的安全等级
* retain ，如果设为 True ，遗嘱消息会被设为保留消息

如果参数设置错误，函数会抛出 ValueError 异常。

### 2.3. 连接

最基本的连接方法是 `connect()` ：

    connect(host, port=1883, keepalive=60, bind_address="")

连接到 broker ，这是一个阻塞函数，参数的含义：

* host ，broker 的 hostname 或者 IP 
* port ，broker 的开放端口，默认是 1883 ，如果使能了 SSL/TLS ，端口可能是 8883 
* keepalive ，心跳间隔，单位是秒，如果 broker 和 client 在这段时间内没有任何通讯，client 会给 broker 发送一个 ping 消息
* bind_address ，如果 client 的本地计算机有多个网络接口，可以用这个参数绑定其中的一个

client 调用该函数发起连接后，如果收到 broker 发来的 CONNACK 消息，就会执行 `on_connect()` 回调函数。除此之外，还有 `connect_async()` 和 `connect_srv()` 两种函数可以连接 broker 。`connect_async()` 需要配合 `loop_start()` 函数以非阻塞的方式连接 broker。`connect_srv()` 是从 SRV DNS 获取 broker 的地址，然后再连接。

调用过 `connect*()` 函数之后，可以调用 `reconnect()` 用现有的参数重新连接。调用 `disconnect()` 函数可以从 broker 断开连接，断开连接后，会执行 `on_disconnect()` 回调函数。

### 2.4. 网络循环

网络循环的函数有四种，它们运行在后台，处理收发的消息。最基本的是 `loop()` ：

    loop(timeout=1.0, max_packets=1)
    
这个函数会通过 `select()` 函数阻塞，直到有消息需要收发，阻塞的时间用 timeout 参数设置，不能超过心跳时间 keepalive ，否则你的 client 会定时从 broker 断开。max_packets 参数已经过时，无需设置。

另一个循环函数是 `loop_forever()` ，它会一直阻塞，直到 client 调用了 `disconnect()` ，并且，它会自动重连：

    loop_forever(timeout=1.0, max_packets=1, retry_first_connection=False)
    
timeout 和 max_packets 参数已经过时，无需设置。

### 2.5. 发布

    publish(topic, payload=None, qos=0, retain=False)

向指定话题发送一条消息，参数的含义：

* topic ，这条消息所属的话题
* payload ，消息内容，字符串类型，如果设为 None ，会发送一条长度为 0 消息。如果设置了 int 或者 float 类型的值，会当做字符串发送，如果你想发送真正的 int 或者 float 值，需要用 `struct.pack()` 生成消息。
* qos ，消息的安全等级
* retain ，如果设为 Ture ，这条消息会被设为保留消息

如果参数设置错误，会抛出 ValueError 异常。消息发送成功后，会执行 `on_publish()` 回调函数。

### 2.6. 订阅

    subscribe(topic, qos=0)

向 broker 订阅话题，参数 topic 设置话题名称，qos 设置安全等级。如果只订阅一个话题，直接设置两个参数即可，例如：

    subscribe(("my/topic", 1))

如果要订阅多个话题，可以将每个话题放在一个元组中，多个话题组成一个列表：

    subscribe([("my/topic", 0), ("another/topic", 2)])
    
当 broker 确认订阅有效后，client 会执行 `on_subscribe()` 回调函数。如果要取消订阅某个话题，可以调用 `unsubscribe(topic)` ，参数是字符串型，如果是取消多个话题，参数应该是一个字符串列表。取消成功的话，会执行 `on_unsubscribe()` 回调函数。
    
### 2.7. 回调函数

当 broker 对 client 的连接请求做出回应时，会调用 `on_connect()` 回调函数，可以在该函数中判断连接是否成功:

    on_connect(client, userdata, flags, rc)

参数 client 是当前 client 的实例，userdata 是 `Client()` 或 `userdata_set()` 设置的用户数据。flags 是 broker 发送的回应 flags ，字典类型。rc 表示连接结果，整数型，0 表示连接成功，连接失败可能的值有：

* 1 ，错误的协议版本
* 2 ，无效的 client ID
* 3 ，服务器不可用
* 4 ，错误的用户名或密码
* 5 ，无法验证

使用实例：

    def on_connect(client, userdata, flags, rc):
        print("Connection returned result: "+connack_string(rc))
    
    mqttc.on_connect = on_connect
    ...
    
对应的，与 broker 断开连接后，会执行 `on_disconnect()` 回调函数：

    on_disconnect(client, userdata, rc)
    
rc 表示断开连接的状态，如果是 0 ，表示是调用了 `disconnect()` 引起的断开连接，其他结果表示意外断开，比如网络中断。使用实例：

    def on_disconnect(client, userdata, rc):
        if rc != 0:
            print("Unexpected disconnection.")
    
    mqttc.on_disconnect = on_disconnect
    ...

当 client 接收到已订阅的话题的消息时，会调用 `on_message()` 回调函数，在该函数中判断是哪个话题的消息，并处理消息内容：

    on_message(client, userdata, message)
    
参数 message 是 MQTTMessage 类的实例，这个类包含的成员有 `topic` ，`payload` ，`qos` ，`retain` 。使用实例：

    def on_message(client, userdata, message):
        print("Received message '" + str(message.payload) + "' on topic '"
            + message.topic + "' with QoS " + str(message.qos))
    
    mqttc.on_message = on_message
    ...
    
如果要用通配符同时处理多个话题的消息，例如用 sensors/# 匹配 sensors/temperature 和 sensors/humidity 话题，可以用 `message_callback_add()` 设置回调函数：

    message_callback_add(sub, callback)

参数 sub 是一个使用通配符的话题过滤器，字符串型，用 callback 参数指定回掉函数，与 `on_message()`  相同的类型。

如果同时设置了 `on_message()` 和 `message_callback_add()` 回调函数，会首先寻找合适的 `message_callback_add()` 定义的话题过滤器，如果没有匹配，才会调用 `on_message()` 。

### 2.8. 实例

假设 broker 要求提供用户名、密码、证书和密钥，下面是一个简单的 client 例子：

    $ cat path-mqtt.py
    #!/usr/bin/python
    
    import paho.mqtt.client as mqtt
    
    cafile = "/etc/mosquitto/ca/ca.crt"
    certfile = "/home/ubuntu/CA/client.crt"
    keyfile = "/home/ubuntu/CA/client.key"
    user = "guest"
    passwd = "12345678"
    server = "localhost"
    port = 8883
    
    def on_connect(client, userdata, flags, rc):
        print("Connected with result code "+str(rc))
        client.subscribe("$SYS/broker/version")
    
    def on_message(client, userdata, msg):
        print(msg.topic+" "+str(msg.payload))
    
    client = mqtt.Client()
    client.tls_set(cafile,certfile,keyfile)
    client.username_pw_set(user,passwd)
    client.on_connect = on_connect
    client.on_message = on_message
    
    client.connect(server, port, 60)
    
    client.loop_forever()
    
执行：

    $ ./path-mqtt.py
    Connected with result code 0
    $SYS/broker/version mosquitto version 1.4.11
