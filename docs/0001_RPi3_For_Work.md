# RPi3 For Work

一个非常重要的理念，把`iotedge`当做一个扩充了百度附加功能的小型**broker**来理解。

## 测试环境说明：

* 本来想直接使用Buildroot文件系统在i.MX6上测试，结果发现程序是用Golang语言开发，由于对Golang不熟悉，仅折腾几天：  
  ![./image/iie_README_golang.png](./image/iie_README_golang.png)
* 查看`iotedge`文件类型：
  ```
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ file bin/iotedge
  bin/iotedge: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, not stripped
  ```
* 由于`iotedge`可执行文件是`statically linked`，在Buildroot文件系统中运行应该是没问题的了，为确保起见，先使用树莓派3熟悉软件。

## RPi3环境搭建

### 打开SSH

[How to Enable SSH on a Raspberry Pi 3](https://www.youtube.com/watch?v=RgUM8ulMfHE)

### 安装工具包

`sudo apt-get install vim`  
备注：`Vim`是个人癖好。

### 解压软件包

1. `unzip iot-intelligent-edge-beta-0.9.3.22-linux-arm.zip`
2. 文档目录：
  ```
  pi@raspberrypi:~/zengjf $ tree
  .
  ├── iot-intelligent-edge-beta-0.9.3.22-linux-arm
  │   ├── bin
  │   │   └── iotedge
  │   ├── conf
  │   │   └── iotedge.yml
  │   ├── example
  │   │   └── function
  │   │       └── sayhi
  │   │           ├── __init__.py
  │   │           └── sayhi.py
  │   ├── libexec
  │   │   ├── iotedge_python2.7
  │   │   └── iotedge_sql
  │   └── README.md
  ├── iot-intelligent-edge-beta-0.9.3.22-linux-arm.zip
  └── __MACOSX
      └── iot-intelligent-edge-beta-0.9.3.22-linux-arm
          ├── bin
          ├── conf
          ├── example
          └── libexec
  
  13 directories, 8 files
  pi@raspberrypi:~/zengjf $
  ```
## iotedge Help

```Shell
pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ bin/iotedge -h
Usage of bin/iotedge:
  -c string
        Config file (default "conf/iotedge.yml")
  -h    Show this help
  -w string
        Working directory (default "/home/pi/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm")
```

## 运行iotedge查看端口监听

* 列出未运行`iotedge`网络监听端口：
  ```Shell
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ netstat -atunlp
  (Not all processes could be identified, non-owned process info
   will not be shown, you would have to be root to see it all.)
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
  tcp        0     64 192.168.23.5:22         192.168.23.1:59575      ESTABLISHED -
  tcp6       0      0 :::22                   :::*                    LISTEN      -
  udp        0      0 0.0.0.0:5353            0.0.0.0:*                           -
  udp        0      0 0.0.0.0:68              0.0.0.0:*                           -
  udp        0      0 0.0.0.0:45652           0.0.0.0:*                           -
  udp6       0      0 :::5353                 :::*                                -
  udp6       0      0 :::56418                :::*                                -
  ```
* 开启`iotedge`：
  ```Shell
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ ./bin/iotedge &
  [1] 2336
  ```
* 列出运行iotedge后网络监听端口：
  ```Shell
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $ netstat -atunlp
  (Not all processes could be identified, non-owned process info
   will not be shown, you would have to be root to see it all.)
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
  tcp        0    256 192.168.23.5:22         192.168.23.1:59575      ESTABLISHED -
  tcp6       0      0 :::22                   :::*                    LISTEN      -
  tcp6       0      0 :::1883                 :::*                    LISTEN      2336/./bin/iotedge
  udp        0      0 0.0.0.0:5353            0.0.0.0:*                           -
  udp        0      0 0.0.0.0:68              0.0.0.0:*                           -
  udp        0      0 0.0.0.0:45652           0.0.0.0:*                           -
  udp6       0      0 :::5353                 :::*                                -
  udp6       0      0 :::56418                :::*                                -
  pi@raspberrypi:~/zengjf/iot-intelligent-edge-beta-0.9.3.22-linux-arm $
  ```
* 对比以上输出信息，可知默认`iotedge`是打开了`tcp 1883`端口监听；

## 配置分析：

```
listen:
  - tcp://:1883                                     # 印证了前面的端口监听
principals:
  - username: 'test'                                # client连接broker的用户名
    password: 'be178c0543eb17f5f3043021c9e5fcf30285e557a4fc309cce97ff9ca6182912'    # client连接broker的密码
    permissions:
      - action: 'pub'                               # 权限类型为push
        permit: ['test', 'benchmark', 'sayhi', 'test/中文']
      - action: 'sub'                               # 权限类型为sub
        permit: ['test', 'benchmark', 'point', 'sayhi', 'test/中文']
      - action: 'shadow'                            # 云端物管理通信的设备影子名称
        permit: ['edge_device']
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
```
