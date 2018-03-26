# Example test Hacking

## `conf/iotedge.yml`相关配置

```yaml
[...省略]
subscriptions:
  - source: 'test'
    target: '$function/filter'
  - source: '$function/filter'
    target: 'point'
  - source: 'sayhi'
    target: '$function/sayhi'
  - source: '$function/sayhi'
    target: 'point'
functions:
  - name: 'filter'
    runtime: 'sql'
    handler: 'select clientid() as cid, topic() as t, @@sayhi(select *) as sayhi where id < 10'
  - name: 'sayhi'
    runtime: 'python2.7'
    handler: 'sayhi.handler'
    codedir: './example/function/sayhi'
    env:
      USER_ID: 'baidu'
    instance:
      min: 1
      max: 3
      timeout: '30s'
      memory:
        high: '8M'
        max: '10M'
      cpu:
        period: 1000000
        max: 500000
      pids:
        max: 2
[...省略]
```

**工作原理**：

* `test`主题消息给`sql`规则引擎；
* `sql`规则引擎会调用`sayhi`函数；

## 确认`sayhi.py`模块文件

```Shell
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ ls -al example/function/sayhi/sayhi.py
-rw-r--r-- 1 pi pi 2033 Mar 21 11:19 example/function/sayhi/sayhi.py
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ file example/function/sayhi/sayhi.py
example/function/sayhi/sayhi.py: Python script, UTF-8 Unicode text executable
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
```

## 调试输出

这里的测试是与下面给出的`sayhi.py`代码相关，`event_context.txt`是自己添加的信息导出调试文件。

* 以`test`主题发送以下数据为例：
  ```JSON
  {
    "i": 5,
    "id": 5
  }
  ```
* 查看`event_context.txt`
  ```
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ cat var/run/event_context.txt
  event type: <type 'dict'>
  context type: <class '__main__.Context'>
  event data: {u'i': 5, u'id': 5}
  context data: {u'messageTopic': u'test', u'functionName': u'sayhi', u'functionInvokeID': u'08e000a1-c229-4d0e-ad3f-e987dd5667dd', u'timestamp': 1521768977, u'sequenceID': 28, u'messageQOS': 0, u'clientID': u'MQTTClient', u'invokeid': u'08e000a1-c229-4d0e-ad3f-e987dd5667dd', u'functionInstanceID': u'e8dde20093b046eab0fee28224e86a38', u'messageRetain': False, u'packetID': 0}
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
  ```

## `sayhi.py` Code Hakcing

在分析这部分的时候，其实有了前一小节的`event_context.txt`中的内容，读代码基本上没任何问题。

```Python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
"""
module to say hi
"""

import os
import time
import threading


def handler(event, context):
    """
    function handler
    """
    # 这部分对照这前面的event_context.txt输出信息看，基本上属于一看就懂
    f_ec = open('event_context.txt', 'w+')
    f_ec.write("event type: " + str(type(event)) + "\r\n")
    f_ec.write("context type: " + str(type(context)) + "\r\n")
    f_ec.write("event data: " + str(event) + "\r\n")
    f_ec.write("context data: " + str(context) + "\r\n")
    f_ec.close()

    if 'i' in event:
        if event['i'] > 10:
            return None

    if 't' in event:
        time.sleep(event['t'])
        return None

    if 'e' in event:
        event['e'] = 1 / 0

    if 's' in event:
        size = event['s']  # MB
        data = ' ' * (size * 1024 * 1024)
        event['l'] = len(data)

    if 'f' in event:
        try:
            f_o = open('sayhi.txt', 'w')
            f_o.write('Hello World')
            event['f_w_n'] = f_o.name
            f_o.close()
        except BaseException as ex:
            event['f_w_e'] = str(ex)
        try:
            f_o = open('../../conf/iotedge.yml', 'r')
            event['f_r_n'] = f_o.name
            event['f_r_d'] = f_o.read()[:10]
            f_o.close()
        except BaseException as ex:
            event['f_r_e'] = str(ex)

    if 'p' in event:
        thr = threading.Thread(target=run)
        thr.setDaemon(True)
        thr.start()
        time.sleep(5)

    if 'c' in event:
        while True:
            pass

    if 'invoke' in event:
        res = context.invoke(event['invoke'], event['invokeArgs'])
        res['invoked'] = True
        return res

    if 'USER_ID' in os.environ:
        event['userID'] = os.environ['USER_ID']

    event['functionName'] = context['functionName']
    event['functionInvokeID'] = context['functionInvokeID']
    event['functionInstanceID'] = context['functionInstanceID']
    event['invokeid'] = context['invokeid']
    event['packetID'] = context['packetID']
    event['messageQOS'] = context['messageQOS']
    event['messageTopic'] = context['messageTopic']
    event['messageRetain'] = context['messageRetain']
    event['py'] = '你好，世界！'

    return event


def run(event):
    """
    function run thread
    """
    for i in range(1, 10):
        event['run.thread.times'] = i
        time.sleep(5)
```
