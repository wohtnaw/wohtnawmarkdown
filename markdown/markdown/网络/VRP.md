# 什么是VRP
华为公司数据通信产品的通用操作平台
# VRP的功能
1. 实现统一的用户界面和管理界面
2. 实现控制平面功能，并定义转发平面接口规范
3. 实现个产品转发平面与VRP控制平面之间的交互
4. 屏蔽各产品链路层对于网络层的差异
# VRP的发展
VRP1到VRP8
VRP8支持多进程，组件化设计，支持多CPU、多框
# 文件系统
文件系统是指对存储器中文件、目录的管理
## 系统软件
是设备启动、运行的必备软件，常见后缀为.cc
## 配置文件
是用户将配置命令保存的文件，常见后缀为.cfg,.zip,.dat
## 补丁文件
是一种与设备软件兼容的软件，常见后缀为.pat
## PAF文件
是根据用户对产品需要提供了一个简单有效的方式来裁剪产品的资源占用和功能特性，常见后缀为.bin
# 存储设备
1. SDRAM：同步动态随机存储器
2. Flash：属于非易失存储器
3. NVRAM：非易失随机读写存储器
4. SD Card：断电后，不会丢失数据，存储容量交大，一般会出现在主控板上
5. USB：接口，用于外接大容量存储设备
# 设备初始化过程
华为网络设备中，VRP（Versatile Routing Platform）的启动过程可以概括为以下几个步骤：

1. 加载引导程序（BootROM），进行硬件自检，初始化内存和外设等操作。

2. 加载并运行引导文件（Boot文件），建立控制通道，进行一些基本设置，如设定管理口IP地址、时间等。

3. 读取并加载系统镜像文件（System文件），包括VRP软件和配置文件等。

4. 运行VRP软件，进行路由协议的初始化、配置读取等操作，最终完成设备的启动。

# 视图
1. 用户视图：以`<>`开头
2. 系统视图：用户视图下，执行`system-view`进入系统视图
3. 接口视图
4. 协议视图
# 用户权限等级

用户级别|命令级别|级别名称|说明
:-----:|:-----:|:-----:|:-----:
0|0|参观级|可执行网络诊断工具命令、远程访问命令等
1|0,1|监控级|用于系统维护，包括display命令等
2|0,1,2|配置级|业务配置命令，包括路由、各个网络层次的命令
3-15|0,1,2,3|管理级|用于系统基本运行的命令，对业务提供支撑作用
# 如何连接华为设备
## VTY与TTY
在华为设备中，vty和tty是两个不同的命令行接口。

* VTY：Virtual Teletype是虚拟远程终端，通常用于远程登录。在华为设备中，vty是通过Telnet或SSH协议连接到设备的远程终端，可以通过vty接口对设备进行配置和管理，最大支持16个虚拟终端同时登录设备。
* TTY：Teletype是电传打字机，通常用于直接连接设备的本地终端，作为控制台使用。在华为设备中，tty是通过设备的本地控制台访问设备，可以通过tty接口对设备进行配置和管理。在华为交换机中，tty接口通常是以Console口或AUX口的形式提供。
  
在华为设备中，vty和tty均可用命令行方式配置，具体使用方式如下：

* 配置VTY：进入系统视图 -> 用户界面子视图 -> VTY命令视图
* 配置TTY：进入系统视图 -> 用户界面子视图 -> 控制台命令视图

## 回环口
回环口是路由器或交换机上的虚拟接口，类似PC上的127.0.0.1
配置方法：
```
interface  loopback 0   //定义回环口
ip address 1.1.1.2 32   //回环口的掩码必须为32
```

## 远程连接网络设备的流程
1. 在设备上配置ip：

```
//R1
ip address 12.1.1.1 24

//R2
ip address 12.1.1.2 24

//查看路由器ip详情
display ip interface brief
```
2. 测试两台设备是否可以通信：`ping 12.1.1.2`
3. 配置vty：`user-interface vty 0 4`(同时设置5个vty连接，可同时供5个telnet连接此设备，当超出5个时，无法连接)
4. 配置认证模式：
在华为设备中，aaa（Authentication, Authorization, and Accounting，身份验证、授权和计费）是一种网络安全机制，提供了一套灵活的认证、授权和账户计费服务。
而password（密码）是指用于保护设备或账户的访问的一种字符串，设备或账户只有在正确提供密码后才能被访问。在华为设备中，设备管理员可以使用password来保护设备或账户的访问权限，以确保设备和账户安全。因此，aaa和password是两个不同的概念，虽然它们都与设备和账户的安全有关，但是它们的作用和应用场景不同。

* password模式
```
//设置认证模式为password
authentication-mode password

//（修改密码）设置密码为Huawei@123
set authentication password cipher Huawei@123

//配置权限等级为15
user privilege level 15

//设置超时时间30分钟0秒
idle-timeout 30 0
```

* aaa模式
```
//设置认证模式为aaa
authentication-mode aaa

//进入aaa模式
aaa

//创建新用户xxx并设置密码为Huawei@123
local-user xxx password cipher Huawei@123

//允许用户使用Telnet进行远程连接
local-user xxx service-type telnet
```
6. 在R1上使用Telnet连接R2：`telnet 12.1.1.2`
7. 注意：配置完成后，必须执行`save`保存配置（在用户视图下）