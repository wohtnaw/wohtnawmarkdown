# RSTP
## STP

```plantuml
@startmindmap

right side

+ 生成树系列

++ 使用场景
+++ STP协议

++++ 缺点
+++++ 收敛慢
++++++ 30s/50s

+++++ 浪费资源
++++++ 会阻塞多余端口，多与端口完全不会跑数据


+++ RSTP协议

++++ 在STP基础上，改进了速度，变成了秒级

++++ 缺点
+++++ 浪费资源
++++++ 会阻塞多余端口，多与端口完全不会跑数据

++++ 使用场景
+++++ 中小型企业的业务


+++ MSTP协议

++++ 在RSTP基础上解决了资源浪费的问题，提高了流量利用率

++++ 缺点
+++++ 速度/利用率很普通

++++ 使用场景
+++++ 企业网络

+++ 堆叠/集群+聚合
++++ 主流虚拟化方案，毫秒级

++++ 缺陷
+++++ 只能自己厂家的设备做虚拟化，并且型号严格要求


++++ 使用场景
+++++ 各类场景部署都适用


++ STP原理
+++ 基础概念

++++ BID
+++++ 组成
++++++ 优先级+MAC地址

++++ RID
+++++ 根桥
+++++ 全网当中会选出唯一的根桥
+++++ 设备在传递BPDU的过程中，里面携带了根桥的BID，放在RID的参数当中

++++ COST
+++++ 计算到根桥的路径距离，用来进行选举比较的


++++ PORT
+++++ 优先级+端口

+++ 选举过程
++++ 选举参数
+++++ 最小的根桥ID
++++++ 优先级+MAC地址
++++++ 比较方式
+++++++ 1.先比较优先级，越小越优先
+++++++ 2.比较MAC地址

+++++ 最小的cost
++++++ 最小的根路径开销

+++++ 最小的BID
++++++ 优先级+MAC地址
++++++ 比较方式
+++++++ 1.先比较优先级，越小越优先
+++++++ 2.比较MAC地址

+++++ 最小的PORT ID

++++ 选举步骤
+++++ 1、全网当中选出唯一的根桥（老大）
++++++ 注意：全网初始情况下，所有人都认为自己是老大

+++++ 2、每个非根交换机选出唯一的一个根端口
++++++ 针对所有端口收到的BPDU进行比较

+++++ 3、每一段当中选出一个指定端口
++++++ 两台设备互相发送的BPDU进行比较

+++++ 4、没有角色的端口为阻塞端口

+++ 状态机
++++ 作用：防止出现临时环路
++++ 过程：侦听---15s---学习----15s---转发

+++ 故障切换
++++ 存在阻塞端口的设备出现链路故障
+++++ 根端口挂掉，等待30s完成收敛（阻塞端口是根端口的替代端口）

++++ 不存在阻塞端口的设备出现链路故障
+++++ 根端口挂掉后，等待50s后，完成收敛，20s的超时时间

left side

-- 配置命令
--- stp mode stp
---- 缺省情况下模式为mstp

--- stp priority 8192
---- 修改BID优先级为8192，缺省情况下是32768

--- stp cost 10000
---- 修改当前端口的stp cost参数为10000

--- display stp brief
---- 查看端口stp信息

@endmindmap
```
## RSTP 
### 概念
1. 在IEEE 802.1W中定义
2. 在许多方面对STP进行优化，收敛速度更快，而且兼容STP
3. 端口角色优化：增加两个端口角色

|RSTP端口角色|说明|备注|
|:-:|:-:|:-:|
|root port|根端口：去往根桥路径开销最小的端口||
|designed port|指定端口：向本网段转发配置消息的端口||
|alternate port|替代端口：学习其他网桥发送的BPDU报文而阻塞的端口|根端口的备份|
|backup port|备份端口：学习到自己发送的BPDU而阻塞的端口|指定端口的备份端口|

4. 端口状态优化：精简成3种状态

|STP端口状态|RSTP端口状态|学习mac地址|转发数据|
|:-:|:-:|:-:|:-:|
|Forwarding|Forwarding|是|是|
|Learning|Learning|是|否|
|Listening|Discarding|否|否|
|Blocking|Discarding|否|否|
|Disabled|Discarding|否|否|

5. 快速收敛机制
根端口快速切换：如果根端口失效，则最优的alternate端口成为根端口，进入forwarding状态

指定端口快速切换：如果指定端口失效，那么最优的backup端口将成为指定端口，进入forwarding状态

6. 边缘端口
边缘端口不参与RSTP计算，直接进入forwarding状态

但是，一旦边缘端口收到bpdu，就会丧失边缘端口属性，成为普通STP端口，并重新进行生成树计算，从而引起网络震荡
7. P/A机制：proposal/agreement
加快上游端口进入forwarding状态的速度

当一个端口被选举为指定端口后，会先进入discarding状态，然后通过P/A机制快速进入forwarding状态

P/A机制要求两台交换机之间链路必须是点到点的全双工模式，一旦P/A不成功，指定端口的选择就需要等待两个forward delay，协商过程与STP一样
8. 拓扑变更机制优化
RSTP拓扑变化唯一标准：一个非边缘端口切换到forwarding状态
9. RSTP对STP优化总结

|对比项|STP|RSTP|
|:-:|:-:|:-:|
|角色状态|5个|3个|
|端口角色|2个|4个|
|配置BPDU flag位使用|2位|4位|
|BPDU超时计时|maxage|hello\*3\*time factor|
|处理次优BPDU|等待超时|立即回应最优BPDU|
|稳定后BPDU发送方式|根桥发送|所有交换机|
|快速收敛|无|P/A机制|
|边缘端口|无|有|
|保护功能|无|4中保护机制|
# MSTP
## RSTP/STP的不足

1. 流量无法负载分担
2. 会产生二层次优路径

## MSTP协议概述

* MSTP是在IEEE 802.1S中定义的，兼容RSTP和STP，既可以快速收敛，又提供了数据转发的多个冗余路径，在数据转发过程中实现VLAN数据的负载均衡
* 可以将一个或多个VLAN实例（instance），再基于实例计算生成树，映射到同一个实例的VLAN共享同一棵生成树
## MSTP网络层次

1. MSTP把一个交换网络划分为多个域，每个域内生成多棵生成树，生成树之间互相独立

2. MST Region:多生成树域

* 由交换网络中的多台交换设备以及它们之间的网段构成
* 一个局域网内可以存在多个MST域，各个域之间在物理上直接或间接连接。可以通过MSTP配置命令把多台交换设备划分在同一个MST域中
* MSTP网络内包含一个或多个MST域，每个MST中包含一个或多个生成树实例

## MSTI

1. MSTI：多生成树实例

* 一个MSTP域中可以有多棵生成树，每棵生成树都称为一个MSTI
* MSTI使用instance id标识，华为设备取值为`0~4094`

2. VLAN映射表
MSTP域的属性，描述了VLAN和MSTI间的映射关系

## CST

* CST:公共生成树
* 是连接交换网络内所有的MST域的一棵生成树
* 如果把每个MST域看做一个节点，CST就是这些节点通过生成树协议计算生成的一棵生成树

## IST

* IST:内部生成树
* 是各MST域内的一棵生成树
* IST是一个特殊的MSTI，它的instance id为0

## CIST

* CIST:公共和内部生成树
* 通过生成树协议计算生成的，连接一个交换网络内所有交换设备的单生成树

## SST

1. SST:单生成树
2. 有两种情况：

* 运行生成树协议的交换设备只能属于一个生成树
* MSTP域中只有一个交换设备，这个交换设备构成单生成树

## MSTP网络中的交换机角色
1. 总根

是CIST的根桥

2. 域根

* 分为IST域根和MSTI域根
* IST域根：在MST域中IST生成树中距离总根最近的交换设备
* MSTI域根：每个多生成树实例的域根

3. 主桥

* 是IST Master，它是域内距离总根最近的交换设备
* 如果总根在MST域中，则总根为该域的主桥
# 链路聚合
## Eth-Trunk概念
1. Eth-Trunk是一种将多个以太网接口捆绑成一个逻辑接口的捆绑技术。
2. Eth-Trunk链路聚合模式：

* 手工负载分担模式
* LACP模式

## 手工负载分担模式
1. 当两台设备中至少有一台不支持LACP协议时，可使用手工负载分担模式的Eth-Trunk来增加设备间的带宽及可靠性。
2. 在手工负载分担模式下，加入Eth-Trunk的链路都进行数据的转发

## 配置手工负载分担模式
配置手工负载分担模式的步骤：

* 创建Eth-Trunk；
* 配置Eth-Trunk的工作模式；
* Eth-Trunk中加入成员接口

## LACP模式
LACP模式也称为M:N模式，其中M条链路处于活动状态转发数据，N条链路处于非活动状态作为备份链路

## LACP配置LACP模式
配置LACP模式的步骤：

* 创建Eth-Trunk；
* 配置Eth-Trunk的工作模式；
* Eth-Trunk中加入成员接口；
* （可选）配置系统LACP优先级；
* （可选）配置活动接口数上限阈值；
* （可选）配置接口LACP优先级；
* （可选）使能LACP抢占并配置抢占延时时间

## 负载分担

1. 基于包的负载分担

在使用Eth-Trunk转发数据时，由于聚合组两端设备之间有多条物理链路，如果每个数据帧在不同链路上转发，则有可能导致数据帧到达对端时间不一致，从而引发数据乱序

2. 基于流的负载分担

Eth-Trunk推荐采用流负载分担的方式，即一条相同的流负载到一条链路，这样既保证了同一数据流的数据帧在同一条物理链路转发，又实现了流量在聚合组内各种物理链路上的负载分担

这种分担模式可以选择按地址分类：源IP/源目IP/源MAC/源目MAC，默认是源目IP，可以通过`load-balence`来设置

# VLAN高级技术
## VLAN聚合
### VLAN聚合产生的技术背景

1. 在一般的三层交换机中，通常是采用一个VLAN对应一个VLANIF接口的方式实现广播域之间的互通，这在某些情况下导致了IP地址的浪费。

2. 因为一个VLAN对应的子网中，子网号、子网广播地址、子网网关地址不能用作VLAN内的主机IP地址，且子网中实际接入的主机可能少于可用IP地址数量，空闲的IP地址也会因不能再被其他VLAN使用而被浪费掉

### VLAN聚合概述

1. VLAN聚合（VLAN Aggregation，也称Super-VLAN）: 指在一个物理网络内，用多个VLAN（称为Sub-VLAN）隔离广播域，并将这些Sub-VLAN聚合成一个逻辑的VLAN（称为Super-VLAN），这些Sub-VLAN使用同一个IP子网和缺省网关，进而达到节约IP地址资源的目的。

2. Sub-VLAN：只包含物理接口，不能建立三层VLANIF接口，用于隔离广播域。每个Sub-VLAN内的主机与外部的三层通信是靠Super-VLAN的三层VLANIF接口来实现的。

3. Super-VLAN：只建立三层VLANIF接口，不包含物理接口，与子网网关对应。与普通VLAN不同，Super-VLAN的VLANIF接口状态取决于所包含Sub-VLAN的物理接口状态

### VLAN聚合的原理

每个Sub-VLAN对应一个广播域，多个Sub-VLAN和一个Super-VLAN关联，只给Super-VLAN分配一个IP子网，所有Sub-VLAN都使用Super-VLAN的IP子网和缺省网关进行三层通信

### VLAN聚合的应用

传统VLAN方式每一个VLAN需要划分不同的IP地址网段，在本例中需要耗费4个IP网段和产生4条路由条目；Super-VLAN方式只需要分配一个IP地址网段，下属二层VLAN共用同一个IP地址网段，共用同一个三层网关，同时VLAN之间保持二层隔离

### 相同Sub-VLAN内部通信

同一个Sub-VLAN之间属于同一个广播域，因此相同Sub-VLAN之间可以通过二层直接通信

### 不同Sub-VLAN之间通信举例

![](../../img/不同Sub-VLAN之间的通信.png)

Super-VLAN VLANIF100开启ARP代理之后PC1和PC2之间通信过程如下：

1. PC1发现PC2与自己在同一网段，且自己ARP表无PC2对应表项，则直接发送ARP广播请求PC2的MAC地址。

2. 作为网关的Super-VLAN对应的VLANIF 100收到PC1的ARP请求，由于网关上使能Sub-VLAN间的ARP代理功能，则向Super-VLAN 100的所有Sub-VLAN接口发送一个ARP广播，请求PC2的MAC地址。

3. PC2收到网关发送的ARP广播后，对此请求进行ARP应答。

4. 网关收到PC2的应答后，就把自己的MAC地址回应给PC1，PC1之后要发给PC2的报文都先发送给网关，由网关做三层转发。

### Sub-VLAN与其他设备的二层通信

* 当Sub-VLAN与其他设备进行二层通信时，与普通的VLAN内二层通信无区别。

* 由于Super-VLAN不属于任何物理接口，即不会处理任何携带Super-VLAN标签的报文

## MUX VLAN
### MUX VLAN产生背景

在企业网络中，各个部门之间网络需要相互独立，通常用VLAN技术可以实现这一要求。如果企业规模很大，且拥有大量的合作伙伴，要求各个合作伙伴能够访问公司服务器，但是不能相互访问，这时如果使用传统的VLAN技术，不但需要耗费大量的VLAN ID，还增加了网络管理者的工作量同时也增加了维护量

MUX VLAN（Multiplex VLAN）提供了一种通过VLAN进行网络资源控制的机制

![](../../img/MUX%20vlan产生背景.png)
### MUX VLAN的基本概念

MUX VLAN分为Principal VLAN（主VLAN）和Subordinate VLAN（从VLAN），Subordinate VLAN又分为Separate VLAN（隔离型从VLAN）和Group VLAN（互通型从VLAN）

![](../../img/MUX%20vlan的基本概念.png)

### MUX VLAN的应用

![](../../img/MUX%20vlan的应用.png)

在交换机上，通过把部门A和部门B所在的VLAN分别设置为互通型从VLAN，把访客区所属的VLAN设置为隔离型从VLAN，把服务器所连接口所属VLAN设置为Principal VLAN，即主VLAN。并且所有从VLAN都与主VLAN绑定，从而实现如下网络设计要求：

* 部门A内的用户之间能够实现二层互通。
* 部门B内的用户之间能够实现二层互通。
* 部门A与部门B的用户之间二层隔离。
* 部门A和部门B的员工都能够通过二层访问服务器。
* 访客区内的任意PC除了能访问服务器之外，不能访问其他任意设备，包括其他访客。

## QinQ
### QinQ概述

* 随着以太网技术在网络中的大量部署，利用VLAN对用户进行隔离和标识受到很大限制。因为IEEE802.1Q中定义的VLAN Tag域只有12个比特，仅能表示4096个VLAN，无法满足城域以太网中标识大量用户的需求，于是QinQ技术应运而生。

* QinQ（802.1Q in 802.1Q）技术是一项扩展VLAN空间的技术，通过在802.1Q标签报文的基础上再增加一层802.1Q的Tag来达到扩展VLAN空间的功能。

* 如下图所示用户报文在公网上传递时携带了两层Tag，内层是私网Tag，外层是公网Tag

![](../../img/QinQ概述.png)

### QinQ封装结构

QinQ封装报文是在无标签的以太网数据帧的源MAC地址字段后面加上两个VLAN标签构成

* TPID（Tag Protocol Identifier，标签协议标识）表示帧类型。取值为0x8100时表示802.1Q Tag帧。如果不支持802.1Q的设备收到这样的帧，会将其丢弃。
* CFI （Canonical Format Indicator，标准格式指示位），表示MAC地址在不同的传输介质中是否以标准格式进行封装，用于兼容以太网和令牌环网

![](../../img/QinQ封装结构.png)

### QinQ工作原理

在公网的传输过程中，设备只根据外层VLAN Tag转发报文，并根据报文的外层VLAN Tag进行MAC地址学习，而用户的私网VLAN Tag将被当作报文的数据部分进行传输。即使私网VLAN Tag相同，也能通过公网VLAN Tag区分不同用户

![](../../img/QinQ工作原理.png)

### QinQ实现方式 - 基本QinQ 

![](../../img/基本QinQ.png)

基本QinQ的报文处理过程:（基于端口）

1. SW1收到VLAN ID为10和20的报文，将该报文发给SW2。
2. SW2收到该报文后，在该报文原有Tag的外侧再添加一层VLAN ID 为100的外层Tag。
3. 带着两层Tag的用户数据报文在网络中按照正常的二层转发流程转发。
4. SW3收到VLAN100的报文后，剥离报文的外层Tag（VLAN ID 为100）。将报文发送给SW4，此时报文只有一层Tag（VLAN ID 为10或20）。
5. SW4收到该报文，根据VLAN ID和目的MAC地址进行相应的转发。

### QinQ实现方式 - 灵活QinQ

![](../../img/灵活QinQ.png)

灵活QinQ的报文处理过程：（基于流分类）

1. SW1收到VLAN ID为10和20的报文，将该报文转发给SW2。
2. SW2收到VLAN ID为10的报文后，添加一层VLAN ID 为100 的外层Tag；SW2收到VLAN ID为20的报文后，添加一层VLAN ID为200的外层Tag。
3. 带着两层Tag的用户数据报文在网络中按照正常的二层转发流程转发。
4. SW3收到报文后，剥离报文的外层Tag（VLAN ID 为100或200）。将报文发送给SW4，此时报文只有一层Tag（VLAN ID 为10或20）。
5. SW4收到报文，根据VLAN ID和目的MAC地址进行相应的转发。

# VRRP
## 单网关面临的问题

当网关Router出现故障时，本网段内以该设备为网关的主机都不能与Internet进行通信

![](../../img/单网关面临的问题.png)

## VRRP概述

通过把几台路由设备联合组成一台虚拟的“路由设备”，使用一定的机制保证当主机的下一跳路由设备出现故障时，及时将业务切换到备份路由设备，从而保持通讯的连续性和可靠性

![](../../img/VRRP概述.png)

## VRRP的基本概念 (1)

* VRRP路由器：运行VRRP协议的路由器，如R1和R2。VRRP是配置在路由器的接口上的，而且也是基于接口来工作的。
* VRID：一个VRRP组（VRRP Group）由多台协同工作的路由器（的接口）组成，使用相同的VRID（Virtual Router Identifier，虚拟路由器标识符）进行标识。属于同一个VRRP组的路由器之间交互VRRP协议报文并产生一台虚拟“路由器”。一个VRRP组中只能出现一台Master路由器。 

![](../../img/VRRP基本概念1.png)

## VRRP的基本概念 (2)

* 虚拟路由器：VRRP为每一个组抽象出一台虚拟“路由器”（Virtual Router），该路由器并非真实存在的物理设备，而是由VRRP虚拟出来的逻辑设备。一个VRRP组只会产生一台虚拟路由器。
* 虚拟IP地址及虚拟MAC地址：虚拟路由器拥有自己的IP地址以及MAC地址，其中IP地址由网络管理员在配置VRRP时指定，一台虚拟路由器可以有一个或多个IP地址，通常情况下用户使用该地址作为网关地址。而虚拟MAC地址的格式是“0000-5e00-01xx”，其中xx为VRID。

![](../../img/VRRP基本概念2.png)

## VRRP的基本概念 (3)

* Master路由器：“Master路由器”在一个VRRP组中承担报文转发任务。在每一个VRRP组中，只有Master路由器才会响应针对虚拟IP地址的ARP Request。Master路由器会以一定的时间间隔周期性地发送VRRP报文，以便通知同一个VRRP组中的Backup路由器关于自己的存活情况。
* Backup路由器：也被称为备份路由器。Backup路由器将会实时侦听Master路由器发送出来的VRRP报文，它随时准备接替Master路由器的工作。
* Priority：优先级值是选举Master路由器和Backup路由器的依据，优先级取值范围0-255，值越大越优先，值相等则比较接口IP地址大小，大者优先。

![](../../img/VRRP基本概念3.png)

## VRRP报文格式

VRRP只有一种报文，即Advertisement报文，基于组播方式发送，因此只能在同一个广播域传递。 Advertisement报文的目的组播地址为224.0.0.18

![](../../img/VRRP报文格式.png)

## VRRP定时器

在VRRP协议工作过程中，VRRP定义了两个定时器：

1. ADVER_INTERVAL定时器：Master发送VRRP通告报文时间周期，缺省值为1秒。
2. MASTER_DOWN定时器：Backup设备监听该定时器超时后，会变为Master状态。MASTER_DOWN定时器计算公式如下：
$$MASTER_DOWN =（3* ADVER_INTERVAL）+ Skew_time（偏移时间）$$
其中，Skew_Time=（256–Priority）/256

## VRRP状态机

VRRP协议状态机有三种状态：Initialize（初始状态）、Master（活动状态）、Backup（备份状态）

![](../../img/VRRP状态机.png)

## VRRP协议状态

1. Master状态

* 定期（ADVER_INTERVAL）发送VRRP报文。
* 以虚拟MAC地址响应对虚拟IP地址的ARP请求。
* 转发目的MAC地址为虚拟MAC地址的IP报文。
* 默认允许ping通虚拟IP地址。
* 当多台设备同时为Master时，若设备收到与自己优先级相同的报文时，会进一步比较IP地址的大小。如果收到报文的源IP地址比自己大，则切换到Backup状态，否则保持Master状态

2. Backup状态

* 接收Master设备发送的VRRP报文，判断Master设备的状态是否正常。
* 对虚拟IP地址的ARP请求，不做响应。
* 丢弃目的MAC地址为虚拟MAC地址的IP报文。
* 丢弃目的IP地址为虚拟IP地址的IP报文。
* 如果收到优先级和自己相同或者比自己优先级大的报文时，重置MASTER_DOWN定时器，不进一步比较IP地址的大小

## VRRP主备选举 (1)

VRRP优先级不相等时主备选举过程：

1. R1与R2的GE0/0/0接口VRRP优先级都是200，两台设备完成初始化后首先切换至Backup状态。
2. 由于优先级相同，R1与R2的MASTER_DOWN定时器超时后，同时由Backup状态切换至Master状态。
3. R1与R2交换VRRP报文，优先级一样，通过比较接口IP地址选举Master路由器，由于R2的接口IP地址大于R1的接口IP地址，因此R2被选举为Master路由器。
4. R2被选举为Master路由器后，立即发送免费ARP报文将虚拟MAC地址通告给与它连接的设备和主机

![](../../img/VRRP主备选举1.png)

## VRRP主备选举 (2)

VRRP优先级相等时主备选举过程：

1. R1与R2的GE0/0/0接口VRRP优先级都是200，两台设备完成初始化后首先切换至Backup状态。
2. 由于优先级相同，R1与R2的MASTER_DOWN定时器超时后，同时由Backup状态切换至Master状态。
3. R1与R2交换VRRP报文，优先级一样，通过比较接口IP地址选举Master路由器，由于R2的接口IP地址大于R1的接口IP地址，因此R2被选举为Master路由器。
4. R2被选举为Master路由器后，立即发送免费ARP报文将虚拟MAC地址通告给与它连接的设备和主机

![](../../img/VRRP主备选举2.png)

## VRRP主备选举 (3)

当路由器接口被配置为VRRP的IP地址拥有者时（接口IP地址与Virtual IP相同），路由器无需等待任何定时器超时，可以直接切换至Master状态

配置为IP地址拥有者时主备选举过程：

1. R1与R2的GE0/0/0接口VRRP优先级都采用默认配置（默认为100），但是R1的GE0/0/0接口IP地址与Virtual IP地址相同。
2. R1的GE0/0/0接口直接切换至Master状态，R1成为Master路由器

![](../../img/VRRP主备选举3.png)

## VRRP主备切换

Master主动退出VRRP组：

![](../../img/Master主动退出VRRP组.png)

Master设备或者链路故障：

![](../../img/Master设备或者链路故障.png)

## VRRP主备回切 (1)

1. 正常情况下，由Master设备负责转发用户报文，如图所示，所有用户流量通过R1到达Internet。
2. 当R1出现故障时，网络会重新进行VRRP主备选举，如图所示，此时R2会成为新的Master设备负责转发用户报文。

![](../../img/VRRP主备回切1.png)

## VRRP主备回切 (2)

当R1从故障恢复后，网络将重新进行VRRP主备选举，由于R1的优先级大于R2，所以R1又重新成为新的Master设备负责转发用户报文

VRRP抢占模式（Preempt Mode） ：

* 抢占模式（默认激活）：如果Backup路由器激活了抢占功能，那么当它发现Master路由器的优先级比自己更低时，它将立即切换至Master状态，成为新的Master路由器
* 非抢占模式：如果Backup路由器没有激活抢占功能，那么即使它发现Master路由器的优先级比自己更低，也只能依然保持Backup状态，直到Master路由器失效。

![](../../img/VRRP主备回切2.png)

## VRRP典型应用
### VRRP负载分担

通过创建多个虚拟路由器，每个物理路由器在不同的VRRP组中扮演不同的角色，不同虚拟路由器的Virtual IP作为不同的内网网关地址可以实现流量转发负载分担

![](../../img/VRRP负载分担.png)
### VRRP监视上行端口

VRRP可监视（Track）上行端口状态，当设备感知上行端口或者链路发生故障时，可主动降低VRRP优先级，从而保证上行链路正常的Backup设备能够通过选举切换为Master状态，指导报文转发。

![](../../img/VRRP监视上行端口.png)

### VRRP与BFD联动

通过配置VRRP与BFD联动，当Backup设备通过BFD感知故障发生之后，不再等待Master_Down_Timer计时器超时而会在BFD检测周期结束后立即切换VRRP状态，此时可以实现毫秒级的主备切换

![](../../img/VRRP与BFD联动.png)

### VRRP与MSTP结合应用

![](../../img/VRRP与MSTP结合应用.png)

