# IoT IntelliEdge

主要目标是测试、验证百度的IoT IntelliEdge的`iotedge`程序是否可以运行在Buildroot搭建的文件系统，主板芯片是i.MX6DL，在测试过程中顺带测试了树莓派3，基本可以推理出在嵌入式Ubuntu、Debian系统上运行也是没问题的。

## Docs

* [0006_iotedge_python3_Config.md](docs/0006_iotedge_python3_Config.md)：测试将Python 2.7换成Python 3的语言执行；
* [0005_iotedge_python2.7_hacking.md](docs/0005_iotedge_python2.7_hacking.md)：iotedge python 2.7 代码跟踪，主要是想改成Python 3来处理；
* [0004_iotedge_Passwd_Change.md](docs/0004_iotedge_Passwd_Change.md)：修改Client连接登陆的密码；
* [0003_i.MX6_Buildroot_Run_iotedge.md](docs/0003_i.MX6_Buildroot_Run_iotedge.md)：在i.MX6DL芯片上运行Buildroot文件系统，同时测试`iotedge`程序通信；
* [0002_Example_test_Hacking.md](docs/0002_Example_test_Hacking.md)：分析测试`test`主题工作原理；
* [0001_RPi3_For_Work.md](docs/0001_RPi3_For_Work.md)：在树莓派3上测试`iotedge`通信；
