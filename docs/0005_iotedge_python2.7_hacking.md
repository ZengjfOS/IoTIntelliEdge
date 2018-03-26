# iotedge python2.7 hacking

## `iotedge_python2.7`程序类型

```Shell
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ file libexec/iotedge_python2.7
libexec/iotedge_python2.7: Python script, ASCII text executable
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
```

如上可知，程序是Python脚本，也就意味着可以直接进行解读了。

## Log日志

* 打开日志输出`libexec/iotedge_python2.7`：
  ```Python
  def get_logger():
      """
      get logger
      """
      logger = logging.getLogger("function")
      # logger.setLevel(logging.INFO)
      logger.setLevel(logging.DEBUG)
  
      # create a file handler
      handler = logging.FileHandler('iotedge_python2.7.log')
      handler.setLevel(logging.DEBUG)
  
      # create a logging format
      formatter = logging.Formatter(
          '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
      handler.setFormatter(formatter)
  
      # add the handlers to the logger
      logger.addHandler(handler)
      return logger
  ```
* 运行Log日志信息
  ```Shell
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ cat var/run/iotedge_python2.7.log
  2018-03-26 05:22:39,402 - function - DEBUG - ['/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python2.7', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python2.7', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/example/function/sayhi', 'sayhi.handler', '63a9e42dbe7843daa79947d3da5d1709']
  2018-03-26 05:22:39,403 - function - DEBUG - start
  2018-03-26 05:23:14,406 - function - DEBUG - 010143000e
  2018-03-26 05:23:14,407 - function - DEBUG - ('\x01', 323, 14)
  2018-03-26 05:23:14,408 - function - DEBUG - receive: [{"clientID":"MQTTClient","functionInstanceID":"63a9e42dbe7843daa79947d3da5d1709","functionInvokeID":"ec7bda17-3960-448d-9218-52c667443036","functionName":"sayhi","invokeid":"ec7bda17-3960-448d-9218-52c667443036","messageQOS":0,"messageRetain":false,"messageTopic":"test","packetID":0,"sequenceID":22,"timestamp":1522041794}][{"i":5,"id":5}]
  2018-03-26 05:23:14,410 - function - DEBUG - send: [{"clientID":"MQTTClient","functionInstanceID":"63a9e42dbe7843daa79947d3da5d1709","functionInvokeID":"ec7bda17-3960-448d-9218-52c667443036","functionName":"sayhi","invokeid":"ec7bda17-3960-448d-9218-52c667443036","messageQOS":0,"messageRetain":false,"messageTopic":"test","packetID":0,"sequenceID":22,"timestamp":1522041794}][{"messageTopic": "test", "functionName": "sayhi", "functionInvokeID": "ec7bda17-3960-448d-9218-52c667443036", "i": 5, "py": "\u4f60\u597d\uff0c\u4e16\u754c\uff01", "userID": "baidu", "messageQOS": 0, "invokeid": "ec7bda17-3960-448d-9218-52c667443036", "functionInstanceID": "63a9e42dbe7843daa79947d3da5d1709", "messageRetain": false, "id": 5, "packetID": 0}]
  ```

## `libexec/iotedge_python2.7` Code Hacking

```Python
#!/usr/bin/env python2.7
#-*- coding:utf-8 -*-
"""
module to run function of python 2.7
"""

import importlib
import json
import struct
import sys
import traceback
import logging

function_request = chr(1)
function_response = chr(2)
functionName = "functionName"


def main():
    """
    run function and wait to handle event
    """
    log = get_logger()
    # 2018-03-26 05:22:39,402 - function - DEBUG - ['/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python2.7', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/libexec/iotedge_python2.7', '/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/example/function/sayhi', 'sayhi.handler', '63a9e42dbe7843daa79947d3da5d1709']
    log.debug(sys.argv)
    if len(sys.argv) < 5:                                       # 检查参数个数
        return

    # "/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm/example/function/sayhi"
    code_dir = sys.argv[2]
    sys.path.append(code_dir)
    module_handler = sys.argv[3].split('.')                     # [sayhi, handler]
    handler_name = module_handler.pop()                         # "handler"
    module = importlib.import_module('.'.join(module_handler))  # 加载模块
    handler = getattr(module, handler_name)                     # 加载处理函数

    log.debug("start")
    # class Stream：后面有定义，系统标准输入、输出、log日志；
    fstream = Stream(sys.stdin, sys.stdout, log)
    while True:
        try:
            # 获取当前次请求数据
            req = fstream.receive(function_request)
            # 创建返回数据包
            res = Packet(function_response)
            res.context = req.context                   # 上下文是固定的
            res.payload = []
            try:
                context, event = req.loads()            # 这里相当于将文本解析成JSON数据
                context.setStream(fstream)              # 设置输出流
                event = handler(event, context)         # 这里就是调用用户写的函数了，这里的event和context就是对应的参数
                if event is not None:
                    res.payload = json.dumps(event)     # 这里就是将用户处理的函数返回值转化成字符串
            except BaseException as ex:
                traceback.print_exc()
                exc_info = sys.exc_info()
                stack = traceback.format_list(
                    traceback.extract_tb(exc_info[2])[1:])
                res.payload = make_error(ex, stack)
            fstream.send(res)                           # 发送经用户处理函数处理完的数据
        except BaseException as ex:
            traceback.print_exc()
            exc_info = sys.exc_info()
            stack = traceback.format_list(traceback.extract_tb(exc_info[2]))
            log.error(ex)
            log.error(exc_info)
            log.error(stack)
            return


class Packet(object):
    """
    function packet
    """

    def __init__(self, type):
        self.type = type

    def dumps(self, context, event):
        """
        dumps payload(str) to event(dict)
        """
        if not isinstance(event, dict):
            raise TypeError('Event should be a dict')
        self.context = json.dumps(context)
        self.payload = json.dumps(event)

    def loads(self):
        """
        loads event(dict) to payload(str)
        """
        event = {}
        context = json.loads(self.context, object_hook=Context)     # 注意这个Context
        if self.payload:
            try:
                event = json.loads(self.payload)
            except ValueError:
                event = self.payload  # raw data, not json format
        return context, event


class Stream(object):
    """
    function stream
    """

    # fstream = Stream(sys.stdin, sys.stdout, log)
    def __init__(self, reader, writer, log):
        self.r = reader         # sys.stdin
        self.w = writer         # sys.stdout
        self.log = log

    def receive(self, pkt_type):
        """
        wait to receive function packet
        """
        # 1. read limited numbers of characters
        # 2. 符号解读：
        #     >	| big-endian     | 
        #     c	| char           | 1bytes
        #     H	| unsigned short | 2bytes
        rawdata = self.r.read(5)
        header = struct.unpack('>cHH', rawdata)
        self.log.debug(binascii.b2a_hex(rawdata))
        self.log.debug(header)
        
        if header[0] != pkt_type:
            raise IOError('Packet type not expected')
        pkt = Packet(header[0])
        pkt.context = self.r.read(header[1])
        pkt.payload = self.r.read(header[2])
        self.log.debug("receive: [%s][%s]", pkt.context, pkt.payload)
        return pkt

    # 参考receive中的解析过程，这里逆向的过程
    def send(self, pkt):
        """
        send function packet
        """
        context_length = len(pkt.context)
        payload_length = len(pkt.payload)
        self.w.write(struct.pack('>cHH', pkt.type,
                                 context_length, payload_length))
        if context_length > 0:
            self.w.write(pkt.context)
        if payload_length > 0:
            self.w.write(pkt.payload)
        self.w.flush()
        self.log.debug("send: [%s][%s]", pkt.context, pkt.payload)


# 继承自字典
class Context(dict):
    """
    function context
    """

    def setStream(self, stream):
        """
        set stream
        """
        self.stream = stream

    def setLogger(self, log):
        """
        set logger
        """
        self.log = log

    # 目前没有发现这个函数又被调用的迹象
    def invoke(self, func_name, event):
        """
        invoke other edge functon
        TODO: lock this method since muti-threads unsafe
        """
        context = self.copy()                           # 拷贝一份原有数据
        context[functionName] = func_name               # 添加进入functionName字段
        req = Packet(function_request)                  # request type
        req.dumps(context, event)                       # json data to string
        self.stream.send(req)                           # send request data to stdout
        res = self.stream.receive(function_response)    # receive response data from stdin
        _, event = res.loads()                          # string to json data
        return event


def make_error(error, stack):
    """
    [Baidu Cloud CFC] make error
    """
    result = {}
    result['errorMessage'] = str(error)
    result['errorType'] = type(error).__name__
    if stack:
        result['stackTrace'] = stack
    return json.dumps(result)


# 这里可以打开Python日志输出到'iotedge_python2.7.log'中
def get_logger():
    """
    get logger
    """
    logger = logging.getLogger("function")
    logger.setLevel(logging.INFO)

    # # 打开日志信息输出
    # # create a file handler
    # handler = logging.FileHandler('iotedge_python2.7.log')
    # handler.setLevel(logging.DEBUG)

    # # create a logging format
    # formatter = logging.Formatter(
    #     '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    # handler.setFormatter(formatter)

    # # add the handlers to the logger
    # logger.addHandler(handler)
    return logger


# 调用前面的main函数
if __name__ == '__main__':
    main()
```
