# iotedge Passwd Change

## 密码修改原理

* `principals.password`：client连接`iotedge`所使用的密码，以sha256编码形式存储；
* `conf/iotedge.yml`
  ```yaml
  [...省略]
  principals:
    - username: 'test'
      password: 'be178c0543eb17f5f3043021c9e5fcf30285e557a4fc309cce97ff9ca6182912'
      [...省略]
  [...省略]
  ```
* sha256验证密码：
  ```Shell
  desk@desk-ubuntu:~$ echo -n hahaha | sha256sum 
  be178c0543eb17f5f3043021c9e5fcf30285e557a4fc309cce97ff9ca6182912  -
  desk@desk-ubuntu:~$ 
  ```
* 用sha256sum生成你想要的密码sha256编码，并保存在yml文件中。
