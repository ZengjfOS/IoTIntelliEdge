# iotedge python3 Config

* 测试环境：树莓派3B；
* 测试语言：Python 3.5；
* 测试结论：目前运行正常；

## `conf/iotedge.yml` Config

```yaml
[...省略]
functions:
  [...省略]
  - name: 'sayhi'
    runtime: 'python3'
    [...省略]
[...省略]
```

## Python3脚本

目前测试发现Python2.7可以直接拷贝为Python3脚本，主要是因为没有使用到有差异的部分，至于为什么直接拷贝就行，因为软件调用是字符串连接合成调用的，操作如下：

```Shell
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ cp libexec/iotedge_python2.7 libexec/iotedge_python3
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ ls libexec/
iotedge_python2.7  iotedge_python3  iotedge_sql
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
```

## Python3 Run Log

下面的Python3 log日志中的文件名称未修改，所以名称还是原来Python2.7的。

```Shell
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ cat var/log/iotedge.log
time="2018-03-26T07:09:13Z" level=debug msg="Function instance start arguments: [/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python3 /home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python3 /home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/example/function/sayhi sayhi.handler 9e15158b122e4824bafb490ac460b6a5]" component=function name=sayhi
time="2018-03-26T07:09:13Z" level=info msg="Create function instance successfully" component=funclet instance=9e15158b122e4824bafb490ac460b6a5 name=sayhi process=2464
time="2018-03-26T07:09:13Z" level=info msg="Setup function manager successfully" component=service
time="2018-03-26T07:09:13Z" level=info msg="Setup broker successfully" component=service
time="2018-03-26T07:09:13Z" level=info msg="Device not configured" component=device_manager error="cloud.device not found"
time="2018-03-26T07:09:13Z" level=info msg="[TCP] endpoint=tcp://:1883, ssl=false" component=server
time="2018-03-26T07:09:21Z" level=debug msg="Create session successfully" component=session_manager
time="2018-03-26T07:09:21Z" level=debug msg="Get 0 subscription(s) successfully: clientID=ses.MQTTClient" component=recorder
time="2018-03-26T07:09:21Z" level=debug msg="Clean session state successfully" clientID=MQTTClient component=session
time="2018-03-26T07:09:21Z" level=info msg="Session connected successfully" clientID=MQTTClient component=session
time="2018-03-26T07:09:24Z" level=info msg="Subscribe successfully" clientID=MQTTClient component=session subTopic=point
time="2018-03-26T07:09:24Z" level=debug msg="Get 0 retaind message(s) successfully" component=recorder
time="2018-03-26T07:09:27Z" level=info msg="Subscribe successfully" clientID=MQTTClient component=session subTopic=test
time="2018-03-26T07:09:27Z" level=debug msg="Get 0 retaind message(s) successfully" component=recorder
time="2018-03-26T07:09:29Z" level=debug msg="Receive message successfully: pid=0, dup=false, retain=false, qos=0, topic=test" clientID=MQTTClient component=session
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=ses.MQTTClient
time="2018-03-26T07:09:29Z" level=debug msg="Route message to 1 subscription(s)" component=router sequence=34 sinker=ses.MQTTClient topic=test
time="2018-03-26T07:09:29Z" level=debug msg="Persist 1 message(s) successfully" component=broker
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=function
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=topic
time="2018-03-26T07:09:29Z" level=debug msg="Route message to 1 subscription(s)" component=router sequence=34 sinker=function topic=test
time="2018-03-26T07:09:29Z" level=debug msg="Function instance start arguments: [/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_sql /home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_sql /home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm select clientid() as cid, topic() as t, @@sayhi(select *) as sayhi where id < 10 d3fc00f56ed847a0b97c17c805ba1a8e]" component=function name=filter
time="2018-03-26T07:09:29Z" level=info msg="Create function instance successfully" component=funclet instance=d3fc00f56ed847a0b97c17c805ba1a8e name=filter process=2470
time="2018-03-26T07:09:29Z" level=debug msg="Persist 1 offset(s) successfully" component=broker
time="2018-03-26T07:09:29Z" level=debug msg="Function invoke elapsed time: 4.468257ms" component=funclet instance=9e15158b122e4824bafb490ac460b6a5 name=sayhi process=2464
time="2018-03-26T07:09:29Z" level=debug msg="Function invoke elapsed time: 32.169937ms" component=funclet instance=d3fc00f56ed847a0b97c17c805ba1a8e name=filter process=2470
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=topic
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=function
time="2018-03-26T07:09:29Z" level=debug msg="Route message to 1 subscription(s)" component=router sequence=35 sinker=topic topic="$function/filter"
time="2018-03-26T07:09:29Z" level=debug msg="Persist 1 message(s) successfully" component=broker
time="2018-03-26T07:09:29Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=ses.MQTTClient
time="2018-03-26T07:09:30Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=function
time="2018-03-26T07:09:30Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=topic
time="2018-03-26T07:09:30Z" level=debug msg="Fetch 1 message(s) successfully" component=sinker sinker=ses.MQTTClient
time="2018-03-26T07:09:30Z" level=debug msg="Route message to 1 subscription(s)" component=router sequence=36 sinker=ses.MQTTClient topic=point
time="2018-03-26T07:09:30Z" level=debug msg="Persist 1 message(s) successfully" component=broker
time="2018-03-26T07:09:30Z" level=debug msg="Persist 1 offset(s) successfully" component=broker
time="2018-03-26T07:09:30Z" level=debug msg="Persist 2 offset(s) successfully" component=broker
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ cat var/run/iotedge_python2.7.log
2018-03-26 07:09:13,603 - function - DEBUG - ['/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python3', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python3', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/example/function/sayhi', 'sayhi.handler', '9e15158b122e4824bafb490ac460b6a5']
2018-03-26 07:09:13,604 - function - DEBUG - start
2018-03-26 07:09:29,958 - function - DEBUG - 010143000e
2018-03-26 07:09:29,958 - function - DEBUG - ('\x01', 323, 14)
2018-03-26 07:09:29,959 - function - DEBUG - receive: [{"clientID":"MQTTClient","functionInstanceID":"9e15158b122e4824bafb490ac460b6a5","functionInvokeID":"bcaf0898-f56e-47fc-86d9-4c48d863ad67","functionName":"sayhi","invokeid":"bcaf0898-f56e-47fc-86d9-4c48d863ad67","messageQOS":0,"messageRetain":false,"messageTopic":"test","packetID":0,"sequenceID":34,"timestamp":1522048169}][{"i":5,"id":5}]
2018-03-26 07:09:29,961 - function - DEBUG - send: [{"clientID":"MQTTClient","functionInstanceID":"9e15158b122e4824bafb490ac460b6a5","functionInvokeID":"bcaf0898-f56e-47fc-86d9-4c48d863ad67","functionName":"sayhi","invokeid":"bcaf0898-f56e-47fc-86d9-4c48d863ad67","messageQOS":0,"messageRetain":false,"messageTopic":"test","packetID":0,"sequenceID":34,"timestamp":1522048169}][{"messageTopic": "test", "functionName": "sayhi", "functionInvokeID": "bcaf0898-f56e-47fc-86d9-4c48d863ad67", "i": 5, "py": "\u4f60\u597d\uff0c\u4e16\u754c\uff01", "userID": "baidu", "messageQOS": 0, "invokeid": "bcaf0898-f56e-47fc-86d9-4c48d863ad67", "functionInstanceID": "9e15158b122e4824bafb490ac460b6a5", "messageRetain": false, "id": 5, "packetID": 0}]
```

## 进程信息

即使前面已经修改为Python 3，但这里进程信息依然是保留在Python 2.7。

```
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ ps
  PID TTY          TIME CMD
  951 pts/0    00:00:10 bash
 2706 pts/0    00:01:14 iotedge
 2711 pts/0    00:00:00 python2.7
 2743 pts/0    00:00:00 ps
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
```
