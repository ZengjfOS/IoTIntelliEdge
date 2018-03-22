# 端计算主程序

程序使用golang编写，可以根据配置文件启动iotedge。

## MQTT协议
MQTT(Message Queuing Telemetry Transport)是一个客户端服务端架构的发布/订阅模式的消息传输协议。它的设计思想是轻巧、开放、简单、规范，易于实现。这些特点使得它对很多场景来说都是很好的选择，特别是对于受限的环境如机器与机器的通信(M2M)以及物联网环境(IoT)。

目前iotedge的MQTT协议支持度如下：
- 支持 Connect、Disconnect、Unsub、Pub、Sub、Ping等功能
- 支持 QoS=0、1的消息发布和订阅
- 支持 Retain、Will Message、Clean Session
- 支持订阅**特定范围**的以"$baidu/"为前缀的主题
- 支持订阅含有"+"、"#"等通配符的主题
- 支持发布**特定范围**的以"$baidu/iot/shadow/"为前缀的主题
- 支持符合约定的clientID和payLoad的校验
- 暂时**不支持**client的Keep Alive特性以及QoS=2的发布和订阅

**注意**：
* clientID支持大小写字母、数字及"-"、"_"字符和空字符(空字符表示client为临时连接，强制cleanSession=true), 最大长度不超过128位；payLoad最大长度不超过32KB；发布、订阅主题中含有的分隔符"/"最多不超过8个，主题名称长度最大不超过255
* 可允许发布的以"$baidu/iot/shadow/"为前缀的主题为：
  - $baidu/iot/shadow/{devicename}/update
  - $baidu/iot/shadow/{devicename}/get
  - $baidu/iot/shadow/{devicename}/delete
  - $baidu/iot/shadow/{devicename}/delta/rejected
* 可允许订阅的以"$baidu/iot/shadow/"为前缀的主题为：
  - $baidu/iot/shadow/{devicename}/update/accepted
  - $baidu/iot/shadow/{devicename}/update/rejected
  - $baidu/iot/shadow/{devicename}/update/documents
  - $baidu/iot/shadow/{devicename}/update/snapshot
  - $baidu/iot/shadow/{devicename}/get/accepted
  - $baidu/iot/shadow/{devicename}/get/rejected
  - $baidu/iot/shadow/{devicename}/delta
  - $baidu/iot/shadow/{devicename}/delete/accepted
  - $baidu/iot/shadow/{devicename}/delete/rejected

## 连接配置

端计算提供支持TCP、SSL(TCP+SSL)、WS(WebSocket)及WSS(WebSocket+SSL)四种方式的连接。

### 配置项
```yaml
listen:
  - tcp://:1883
  - ssl://:1884
  - ws://:8080/mqtt
  - wss://:8884/mqtt
certificate:
  cert: 'conf/server.pem'
  key: 'conf/server.key'
principals:
  - username: 'test'
    password: 'be178c0543eb17f5f3043021c9e5fcf30285e557a4fc309cce97ff9ca6182912'
    permissions:
      - action: 'pub'
        permit: ['+', '#', 'test/+', 'test/#', 'test', 'benchmark', 'test/中文']
      - action: 'sub'
        permit: ['+', '#', 'test/+', 'test/#', 'test', 'benchmark', 'test/中文']
      - action: 'shadow'
        permit: ['edge_test_device_1']
```
| 配置项 | 含义 |
| ----- | --- |
| listen | 各连接方式下的address地址及端口信息 |
| certificate | iotedge启动所需要的认证信息 |
| certificate.cert | iotedge启动所需要的认证公钥信息 |
| certificate.key |  iotedge启动所需要的认证私钥信息 |
| principals | principals是iotedge连接的安全配置，client连接必须传递账号密码，并且配置该账号密码下可发布和订阅的主题信息，及与云端物管理通信的设备信息 |
| principals.username | client连接iotedge所使用的账号 |
| principals.password | client连接iotedge所使用的密码，以sha256编码形式存储 |
| principals.permissions | 可连接iotedge的client在上述账号和密码下可发布、订阅及与云端物管理通信的主题列表和设备列表 |
| principals.permissions.action | 可连接iotedge的client在上述账号和密码下对某主题所拥有的"发布"或"订阅"或与云端物管理通信的设备权限 |
| principals.permissions.permit | 可连接iotedge的client在上述账号和密码下拥有"发布"或"订阅"权限主题列表，及与云端物管理通信的权限设备列表 |

#### "#" 策略

对于"#"策略涉及的主题(含通配符"#"的主题)，支持符合MQTT协议标准的匹配规则，如pub行为的permit列表中配置有"#"主题，则不再需要配置其他所有主题，即可允许向所有满足MQTT协议规则的主题发布消息；同样，对于sub行为的permit列表中配置有"#"主题，亦不再需要配置其他所有主题，即可允许向所有满足MQTT协议规则的主题订阅消息

#### "+" 策略

对于"+"策略涉及的主题(含通配符"+"的主题)，支持符合MQTT协议标准的匹配规则，如pub行为的permit列表中配置有"+"主题，则不再需要配置其他所有单层主题，即可允许向所有满足MQTT协议规则的单层主题发布消息；同样，对于sub行为的permit列表中配置有"+"主题，亦不再需要配置其他所有单层主题，即可允许向所有满足MQTT协议规则的单层主题订阅消息；

**注意**:
* principals中password的配置仅支持以sha256编码形式进行存储
* principals中permit列表中的主题约束标准依据上文提到的发布、订阅主题的约束标准，特别地，对于shadow行为permit列表中的设备名称，仅允许大小写字母、下划线和数字，且长度范围为[3,32]
* 上述principals配置中pub和sub行为对应的permit列表中的主题不直接依赖MQTT协议主题规则，而是依赖具体的"#"、"+"策略，非"#"、"+"策略的主题，依据MQTT协议普通主题(不含通配符"#"和"+"的主题)匹配规则
* 对于"#"和"+"策略配置的检查、校验依赖MQTT协议主题过滤器中的主题标准，对于以"$baidu/iot/shadow/"为前缀主题的配置，则依赖上文提到的发布和订阅规则
* 对于需要在principals配置项中配置大量发布和订阅主题的用户，推荐采用"#"和"+"策略

### 使用步骤

* step1: 启动端计算主程序文件iotedge
* step2: 采用[MQTTBox](http://workswithweb.com/mqttbox.html)或[MQTTfx](http://mqttfx.bceapp.com/)作为连接客户端，依据配置文件conf/iotedge.yml中的连接可用user信息进行连接配置操作
* step3: 连接成功后依据conf/iotedge.yml文件, 配置对应连接用户所具有订阅权限的主题信息
* step4: 与step3类似, 选择向step3中连接使用user具备发布权限的主题推送消息
* step5: 如上述步骤正常无误，即可通过iotedge和client观察消息传输过程和收发状态

**注意**：
* 如果采用同一个client既作为发布client又作为订阅client，就要求对某一主题同时具备发布和订阅权限
* 如果是采用两个不同的client，并选择其中一个作为发布client，另外一个作为订阅client，则只需满足两个不同user对某主题一方具备订阅权限，另一方具备发布权限即可
* 如果想要与云端物管理进行通信，通信时的主题请参照上文可允许发布/订阅的以"$baidu/iot/shadow/"为前缀的约定范围主题
* 如果采用SSL(推荐)或WSS方式连接，还需要选择连接配置所用的证书文件置于conf目录下，然后按照上述方式配置、连接即可
* 对于Function的函数测试，建议使用json格式的消息进行传输

## 规则引擎
规则引擎帮助用户灵活地转发和处理设备消息。用户可通过配置subscriptions的source和target，实现设备消息从source到target的转发;通过在配置项中添加函数计算，实现对消息的数据筛选和变形。

具体的配置方法可参考下面的例子

#### 1. 消息的简单转发
```YAML
subscriptions:
  - source: 'talks'
    target: 'mytalk'
```
如上配置，可以实现talks到mytalk的消息转发。
比如client1向edge订阅mytalk消息,client2向edge发布talks消息,这个时候edge会将talks消息转发给订阅了mytalk消息的client1.

#### 2. 消息通过函数计算
```YAML
subscriptions:
  - source: 'talks'
    target: '$function/filter'
```
如上配置，用户可自定义函数对消息进行筛选或其它操作，详见[函数计算](#函数计算)

#### 3. 消息的多级传递
```YAML
subscriptions:
  - source: 'talk1'
    target: 'talk2'
  - source: 'talk2'
    target: 'talk3'
```
如上配置,实现了消息的多级传递,用户可以使用这种级联的方式,进行更加灵活的规则配置,如下面的两个示例。

+ **示例1:**   通过函数计算的多级传递,对talks消息进行多次数据变形。
```YAML
subscriptions:
  - source: 'talks'
    target: '$function/filter'
  - source: '$function/filter'
    target: '$function/sayhi'
```

+ **示例2:**  通过一对多的映射关系,对同一个消息,进行不同的路由转发。
```YAML
subscriptions:
  - source: 'test'
    target: '$function/func'
  - source: 'test'
    target: '$function/filter'
  - source: 'test'
    target: '$function/sayhi'
```

**提示**:
* 配置中的source和target需配对使用，如果重复配置最后一个起作用，如下列配置等同于将主题b路由到主题e：
```YAML
subscriptions:
  - source: 'a'
    source: 'b'
    target: 'c'
    target: 'd'
    target: 'e'
```
* 配置中不允许出现多条相同的配置，如下列配置不合法：
```YAML
subscriptions:
  - source: 'a'
    target: 'b'
  - source: 'a'
    target: 'b'
```
* 请用户避免出现闭环配置，如下列配置不合法：
```YAML
subscriptions:
  - source: 'case1'
    target: 'case1'
  - source: 'case2_a'
    target: 'case2_b'
  - source: 'case2_b'
    target: 'case2_a'
```

另外,用户无法直接订阅以$function为前缀的主题,若想获取函数计算的结果,可通过如下配置,将函数计算的结果路由到一个非$function起头的主题
```YAML
subscriptions:
  - source: 'talks'
    target: '$function/filter'
  - source: '$function/filter'
    target: 'point'
```

**提示**:
* subscriptions配置项source和target中以$function/为前缀的主题，其后面函数名称规则即CFC函数命名规则。CFC函数命名规则为:
  - ^[a-zA-Z0-9_-]+$
  - 长度不得超过140个字符
* subscriptions配置项source和target中非系统主题(即不以$为前缀的主题)，其应遵循的配置规则如下：
  - source配置规则依据MQTT协议订阅主题规则，target规则依据MQTT协议发布主题规则
  - 主题中含有的分隔符"/"最多不超过8个，主题名称长度最大不超过255

## 函数计算

函数计算提供基于MQTT消息机制，弹性、高可用、扩展性好、极速响应的计算能力。兼容[百度云-函数计算CFC](https://cloud.baidu.com/product/cfc.html)。

函数通过一个或多个具体的实例执行，每个实例都是一个独立的进程。

### 函数配置

所有函数配置在functions配置项中。如下图所示，配置了两个函数，分别是名叫filter的SQL函数和名叫sayhi的python函数。

```YAML
functions:
  - name: 'filter'
    runtime: 'sql'
    handler: 'select uuid() as uuid, topic() as topic, * where id < 10'
  - name: 'sayhi'
    runtime: 'python2.7'
    handler: 'sayhi.handler'
    codedir: './example/function/sayhi'
    env:
      USER_ID: 'LDF'
    instance:
      min: 0
      max: 3
      timeout: 30
      memory:
        high: '8M'
        max: '10M'
      cpu:
        period: 1000000
        max: 500000
      pids:
        max: 2
```

函数配置项如下：

* name: 函数名称，区分大小写，需保持唯一。
* runtime: 运行语言，目前支持sql和python2.7。
* handler: 运行的函数名，sql特指select语句。
* codedir: python函数包所在文件夹的路径。
* env: 供python函数使用的环境变量，键值对。
* instance 函数实例配置项
  * timeout: 实例处理消息的超时时间（单位为秒），默认5分钟。实例处理某个消息一旦超时，该实例会被重置，并输出一条错误消息。
  * min: 最少实例数，默认0个。每10分钟清理一次空闲的实例，最多减到min个。
  * max: 最大实例数，默认1个。消息并发数增加，实例数会自动增加，最多增加到max个。
  * memory 函数实例内存配置项
    * high: 需要cgroup支持，表示实例使用内存的软额度，缺省单位为b（字节），支持b、k、m和g四个单位。如果系统内存不足时，会优先回收超过软限额的进程占用的内存，使之向限定值靠拢。对应cgroup v1中的memory.soft_limit_in_bytes。
    * max: 需要cgroup支持，表示实例使用内存的额度，缺省单位为b（字节），支持b、k、m和g四个单位。一旦使用的内存超过限定值，实例会被立即重置，被处理的消息会跳过。对应cgroup v1中的memory.limit_in_bytes。
  * cpu 函数实例CPU配置项
    * period和max: 需要cgroup支持，单位都是微妙，period配置CPU时间周期长度，max配置period周期长度内所能使用的CPU时间数，这两个配置项相互配合表达实例使用CPU的上限。比如period=1000000且max=500000表示实例最多可以使用半个CPU。两个配置项分别对应cgroup v1中的cpu.cfs_period_us和cpu.cfs_quota_us。
  * pids 函数实例进程配置项
    * max: 需要cgroup支持，表示实例允许创建的最大进程数，实例本身算一个进程。对应cgroup v1中的pids.max。

**注意**: 目前只开放了部分资源限制相关的配置，且只支持cgroup，需要linux系统（内核版本大于等于3.10），root权限启动。启动cgroup需要在配置文件中配置：

```YAML
control:
  cgroup: true
```

**注意**: 如果函数执行错误，函数计算会返回如下格式的消息，供后续处理。

```JSON
{
  "errorType":"error type",
  "errorMessage":"error message",
  "stackTrace":"error callstack if exists"
}
```

### 函数触发

函数有两种触发方式：
* 消息路由到函数
* 函数间的相互调用（详见[SQL函数](#SQL函数)和[Python函数](#Python函数)）

这里介绍一下第一种方式：函数配置好后，在subscriptions配置项中指定触发函数计算的消息主题。如下图所示，前面配置的filter函数订阅了talks主题的消息，并输出主题为$function/filter的新消息，再将$function/filter输出的消息转成主题point。sayhi函数类似。

```YAML
subscriptions:
  - source: 'talks'
    target: '$function/filter'
  - source: '$function/filter'
    target: 'point'
  - source: 'sayhi'
    target: '$function/sayhi'
  - source: '$function/sayhi'
    target: 'point'
```

同时还需要配置主题的pub和sub权限，举例如下：

```YAML
principals:
  - username: 'test'
    password: 'be178c0543eb17f5f3043021c9e5fcf30285e557a4fc309cce97ff9ca6182912'
    permissions:
      - action: 'pub'
        permit: ['talks', 'sayhi']
      - action: 'sub'
        permit: ['talks', 'point', 'sayhi']
```

### SQL函数

SQL函数与[百度云-物联网-规则引擎Rule Engine](https://cloud.baidu.com/product/re.html)功能类似，通过SQL语法对数据进行过滤和转换。
SQL函数的输入输出必须是JSON格式，嵌套字段用“.”连接。

SQL函数使用举例：

```SQL
select msg.name as item.key, msg.data as item.value where msg.id < 10
```

如果输入消息为：

```JSON
{"msg":{"id":1,"name":"a","data":"aaa"}}
```

则输出消息为：

```JSON
{"item":{"key":"a","value":"aaa"}}
```

如果输入消息为：

```JSON
{"msg":{"id":11,"name":"a","data":"aaa"}}
```

则不输出任何消息。

SQL函数调用其他函数:

```SQL
select @@add(select a, b) as sum
```

上述SQL表示，将‘select a, b’的结果作为参数传给add函数，并将add结果赋给sum字段输出。

### Python函数

Python函数与[百度云-函数计算CFC](https://cloud.baidu.com/product/cfc.html)功能类似，用户通过编写的自己的函数来处理消息，可进行消息的过滤、转换和转发等，使用非常灵活。

Python函数的输入输出可以是JSON格式也可以是二进制形式。消息payload在作为参数传给函数前会尝试一次JSON解码（json.loads(payload)），如果成功则传入字典（dict）类型，失败则传入原二进制数据。

Python函数支持读取环境变量，比如os.environ['PATH']。

Python函数支持读取上下文，比如context['functionName']。

Python函数实现举例：

```PYTHON
#!/usr/bin/env python
#-*- coding:utf-8 -*-
"""
module to say hi
"""

def handler(event, context):
    """
    function handler
    """
    event['context'] = context
    event['functionName'] = context['functionName']
    event['functionInvokeID'] = context['functionInvokeID']
    event['functionInstanceID'] = context['functionInstanceID']
    event['invokeid'] = context['invokeid']
    event['packetID'] = context['packetID']
    event['messageQOS'] = context['messageQOS']
    event['messageTopic'] = context['messageTopic']
    event['messageRetain'] = context['messageRetain']
    event['sayhi'] = '你好，世界！'
    return event
```

Python函数调用其他:

```PYTHON
#!/usr/bin/env python
#-*- coding:utf-8 -*-
"""
module to say hi
"""

def handler(event, context):
    """
    function handler
    """
    event['sum'] = context.invoke("add", {"a":event['a'],"b":event['b']})
    return event
```

上述Python表示，将{"a":event['a'],"b":event['b']}字典数据作为参数传给add函数，并将add结果（字典）存入event字典。

**提示**：需要自行安装python2.7，对于Windows系统，启动时候如果提示缺少win32file，请通过[这里](https://github.com/mhammond/pywin32/releases)下载安装

## 其他配置

其他配置及其默认值如下：

```yaml
broker:
  status:
    loggable: false
    interval: '60s'
  message:
    ingress:
      batchsize: 100
      buffersize: 200
    egress:
      batchsize: 50
      buffersize: 100
    offset:
      batchsize: 300
      buffersize: 600
    cleanup:
      disposable: false
      retention: '1d'
      interval: '60s'
session:
  qos1ack:
    interval: '20s'
    buffersize: 20
logger:
  level: 'info'
  format: 'text'
  console: false
  path: 'var/log/iotedge.log'
  maxage: 15
  maxsize: 50
  maxbackups: 15
```

* broker: 消息broker配置项。
  * status: broker状态相关的配置项。
    * loggable: 是否打印broker状态信息。
    * interval： broker状态信息打印间隔。
  * message: 消息相关的配置项。
    * ingress: 消息接收相关配置，所有iotedge收到的消息都会写入数据库（持久化）。
      * batchsize: 批量写消息到数据库（持久化）的最大条数，消息持久化成功后会回复确认（ack）。
      * buffersize: 等待持久化的消息缓存大小，增大缓存可提高消息接收的性能，但潜在的风险是iotedge异常退出（比如设备掉电）会丢失缓存的消息，如果qos为0的消息会直接丢失，如果qos为1的消息会丢失且不回复确认（puback）。iotedge正常退出会等待缓存的消息处理完，不会丢失数据。
    * egress: 消息发送相关配置。
      * batchsize: 批量从数据库读取消息的最大条数。
      * buffersize: 消息发送后，未确认（ack）的消息缓存大小，缓存满后，不再读取新消息，一直等待缓存中的消息被确认。qos为0的消息发送成功后立马确认，qos为1的消息发送给客户端成功后等待客户端确认（puback），如果客户端在规定时间内没有回复确认，iotedge会一直重发，直到客户端回复确认或者session关闭。
    * offset: 消息序列号持久化相关配置。
      * batchsize: 批量写消息序列号到数据库的最大条数。
      * buffersize: 未被确认（ack）的消息的序列号的缓存队列大小。比如当前批量发送了qos为1且序列号为1、2和3的三条消息给客户端，客户端确认了序列号1和3的消息，此时序列号1会入列并持久化，序列号3虽然已经确认，但是还是得等待序列号2被确认入列后才能入列。该设计可保证iotedge异常重启后仍能从持久化的序列号恢复消息处理，保证消息不丢，但是会出现消息重发，也因此暂不支持qos为2的消息。
    * cleanup: 持久化消息清理配置项。
      * interval： 消息清理间隔。
      * retention：消息保存在数据库中的时间，超过该时间的消息会在清理时物理删除。
      * disposable: 如果为true，则消息清理时会额外删除已被所有订阅者处理的消息，打开可有效降低存储空间。但是可能导致cleansession为false的订阅者无法收到重连前的消息。
* session: MQTT客户端连接iotedge的session配置项。
  * qos1ack: qos为1的消息确认配置项。
    * buffersize: qos为1的消息发送给客户端后，等待确认的消息缓存大小，缓存满后不再发送新消息给客户端，而是不停的重发缓存中序列号最小的消息，直到该消息被确认（puback）。
    * interval: 消息重发的时间间隔。
* logger: 日志相关配置项。
  * level: 日志等级，支持debug、info、warn和error。
  * format: 日志打印格式，支持text和json。
  * console: 日志是否输出到控制台。
  * path: 日志存储路径，支持相对路径。
  * maxage: 日志文件保留的最大天数。
  * maxsize: 日志文件大小限制，单位MB。
  * maxbackups: 日志文件保留的最大数量。
