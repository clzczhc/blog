---
title: 计算机网络(四)    网络层
date: 2021-12-26 19:57:31
categories: "计算机网络"
keywords: 计算机网络
tags:
  - 计算机网络
---

# 计算机网络(四) 网络层

## 1. 网络层提供的两种服务

### 1.1 让网络负责可靠交付

- 模拟电信网络，使用**面向连接**的通信方式

- 通信之前先建立**虚电路**，保证双方通信所需的一切网络资源

  ![](https://s2.loli.net/2021/12/26/yHbS9MDL8glfnwe.jpg)

**虚电路是逻辑连接**：虚电路只是**逻辑上的连接**，分组都沿着这条逻辑连接**按照存储转发方式传送**，不是真正建立了一条物理连接

### 1.2 网络提供数据报服务

- 网络层向上只提供简单灵活的、**无连接的**、**尽最大努力交付**的**数据报服务**

- 网络在发送分组时不需要先建立连接。每一个分组(即 IP 数据报)独立发送，与其前后的分组无关

- <b style="color: red">网络层不提供端到端的可靠传输服务，由主机中的运输层负责可靠交付(包括差错处理、流量控制等)</b>

  ![](https://s2.loli.net/2021/12/26/7qcHNbFrCkymVWT.jpg)

![](https://s2.loli.net/2021/12/26/Sku6HETj5ORxvoV.jpg)

## 2. 网际协议 IP

- 网际协议 IP 是 TCP/IP 体系中两个最主要的协议之一
- 与 IP 协议配套使用的三个协议：<b style="color: red">地址解析协议 ARP</b>、<b style="color: red">网际控制报文协议 ICMP</b>、<b style="color: red">网际组管理协议 IGMP</b>

### 2.1 虚拟互连网络

虚拟互连网络：逻辑互连网络。即互连起来的各种物理网络的异构性本来是客观存在的，但是利用 IP 协议可以使这些性能各异的网络从用户看起来好像是一个统一的网络

**好处**：当互联网上的主机进行通信时，就好像在一个网络上通信一样，而看不见具体的网络异构细节

### 2.2 分类的 IP 地址

#### 2.2.1 IP 地址及其表示方法

- **IP 地址**就是给每个连接在互联网上的主机(或路由器)分配一个在全世界范围是**唯一的 32 位的标识符**

#### 2.2.2 IP 地址的编址方法

<b style="color: red">分类 IP 地址：最基本的编址方法</b>

- 将 IP 地址划分为若干个固定类(如 A 类、B 类、C 类地址)

- 每一类地址都由两个固定长度的字段组成，第一个字段是**网络号**，标志主机(或路由器)所连接到的网络，第二个字段是**主机号**，标志该主机(或路由器)

  ![](https://s2.loli.net/2021/12/26/r5ozRkJpUgHySqW.png)

- <b style="color: red">一个 IP 地址在整个互联网范围内是唯一的</b>

![](https://s2.loli.net/2021/12/26/SxtKypvMl6aX5R3.jpg)

由上图可知，

- IP 地址第一位是 0 时属于 A 类地址，IP 地址前 2 位是 10 时属于 B 类地址，IP 地址前 3 位是 110 时属于 C 类地址
- A 类地址的网络号字段为 1 字节，主机号字段为 3 字节；B 类地址的网络号字段为 2 字节，主机号字段为 2 字节; C 类地址的网络号字段为 3 字节，主机号字段为 1 字节

点分十进制记法：

1. 机器中存放的 IP 地址是 32 位二进制代码
2. 把二进制代码按每 8 位为一组分为 4 组
3. 把每一组的二进制数转换为十进制数

![](https://s2.loli.net/2021/12/26/6vjAT1skC7WURob.jpg)

<b style="color: red">IP 地址的一些重要特点：</b>

- **IP 地址是一种分等级的地址结构**

  好处：

  1. IP 地址管理机构在分配 IP 地址时只分配网络号，主机号由得到该网络号的单位自行分配，方便了 IP 地址的管理
  2. 路由器仅根据目的主机所连接的网络号来转发分组，使路由表中的项目数大幅度减少，从而减少路由表所占的存储空间

- **IP 地址是标志一个主机(或路由器)和一条链路的接口**

  - 当一个主机同时连接到两个网络上时，这个时候这台主机就必须要有两个 IP 地址，而且网络号必须是不同的。这种主机被称为**多归属主机**
  - 由上一条可以知道，**一个路由器至少要有两个不同的 IP 地址**，因为路由器需要将 IP 数据报从一个网络转发到另一个网络

- **用转发器或网桥连接起来的若干个局域网是同一个网络，所以这些局域网都具有同样的网络号**

- **所有分配到网络号的网络都是平等的**

![](https://s2.loli.net/2021/12/26/x13vZzRBYAfbdqG.jpg)

分析：

- 222 的二进制是 11011110，所以上图中的 IP 地址都属于 C 类地址
- C 类地址的网络号字段 net-id 为 3 字节，而同一个局域网上的主机(或路由器)的 IP 地址上的网络号是一样的，所以上图每一块粉色区域分别是一个局域网

### 2.3 IP 地址与硬件地址

- **IP 地址**是网络层和以上各层使用的地址，是一种逻辑地址
- **硬件地址(或物理地址)**是数据链路层和物理层使用的地址

![](https://s2.loli.net/2021/12/26/kryRIU2zwX8VdZ4.jpg)

### 2.4 地址解析协议 ARP

通信的时候使用了两个地址，**IP 地址(网络层地址)**和**MAC 地址(数据链路层地址)**

**地址解析协议**作用：通过网络层使用的 IP 地址，解析出在数据链路层使用的硬件地址

<b style="color: red">不论网络层使用的是什么协议，实际网络的链路上传送数据帧时，最终都必须使用硬件地址</b>

每一个主机都设有一个**ARP 高速缓存**，里面有它在的局域网上的所有主机和路由器的 IP 地址到硬件地址的映射表。

` <IP address; MAC address; TTL>`

**TTL(Time To Live): **地址映射有效时间

**地址解析协议 ARP**：

- 当主机 A 要向局域网中的主机 B 发送 IP 数据包时，会先在它的 ARP 高速缓存中查看有无主机 B 的 IP 地址。如果有，就将对应的硬件地址写入 MAC 帧，然后把 MAC 帧发送这个硬件地址，如果没有，则 ARP 进程在局域网中**广播发送**一个**ARP 请求分组**。收到**ARP 响应分组**后，将得到的 IP 地址到硬件地址的映射写入 ARP 高速缓存中。

![](https://s2.loli.net/2021/12/26/8lyokHMvmg5zeL4.jpg)

<b style="color: red">ARP 高速缓存的作用：存放最近获得的 IP 地址到 MAC 地址的绑定，以减少 ARP 广播的数量</b>

**要注意的问题**：

ARP 是用于解决<b style="color: red">同一个局域网</b>的主机或路由器的 IP 地址和硬件地址的映射问题的

如果要找的主机和源主机不在同一个局域网中，就要**通过 ARP 找到本局域网中的某个路由器的硬件地址**，把分组发送给这个路由器，让路由器把分组转发给下一个网络，剩下的都交给下一个网络。

**使用 ARP 的四种典型情况**：

- 发送方是主机，要把 IP 数据报发送到本网络上的另一个主机。可以用 ARP 找到目的主机的硬件地址。
- 发送方是主机，要把 IP 数据报发送到另一个网络上的一个主机。用 ARP 找到本网络上的一个路由器的硬件地址，剩下的交给这个路由器。
- 发送方是路由器，要把 IP 数据报发送到本网络上的一个主机。可以用 ARP 找到目的主机的硬件地址。
- 发送方是路由器，要把 IP 数据报发送到另一个网络上的一个主机。用 ARP 找到本网络上的另一个路由器的硬件地址，剩下的交给这个路由器。

### 2.5 IP 数据报的形式

- 一个数据报由**首部**和**数据**两部分组成
- **首部的前一部分是固定长度，一共 20 字节，是所有 IP 数据报必须具有的**
- 首部的固定部分的后面是一些可选字段，长度可变

![image-20211102224544607](https://s2.loli.net/2021/12/26/nigWU7DG9QxAe1a.png)

首部检验和的计算采用 16 位二进制反码求和算法。这个字段**只检验数据包的首部，不包括数据部分**

![image-20211102225044386](https://s2.loli.net/2021/12/26/m1C4ajFBPyuTAUR.png)

### 2.6 IP 层转发分组的流程

主要原理：**查找路由表，根据目的网络地址能确定下一跳路由器**

## 3. 划分子网和构造超网

### 3.1 划分子网

从 1985 年起，IP 地址增加一个**子网号字段**，使两级的 IP 地址变成三级的 IP 地址

![image-20211102230559834](https://s2.loli.net/2021/12/26/UEpISmNWol69icZ.png)

优点：

- 减少了 IP 地址的浪费
- 使网络的组织更加灵活
- 便于维护和管理

#### 3.1.1 子网掩码

背景：从一个 IP 数据报的首部无法判断源主机或目的主机所连接的网络是否进行了子网划分

使用**子网掩码**可以找出 IP 地址中的子网部分

规则：

- 子网掩码长度：32 位
- 子网掩码对应网络号和子网号的左边部分是一连串 1
- 子网掩码对应于主机号的右边部分是一连串 0

**IP 地址 AND 子网掩码 = 网络地址**

![image-20211102231736594](https://s2.loli.net/2021/12/26/8TYpRJomcFClSE4.png)

**默认子网地址**

![image-20211102231818881](C:\Users\CZH0318\AppData\Roaming\Typora\typora-user-images\image-20211102231818881.png)

## 4. ICMP

### 4.1 简介

**ICMP**：网际控制报文协议，是互联网的标准协议，是 IP 层的协议

作用：为了更有效地转发 IP 数据报和提高交付成功的机会

![](https://s2.loli.net/2021/12/26/p3XiEmaWMNC2k4V.png)

### 4.2 ICMP 报文的种类

- ICMP 差错报告报文(四种):

  - 终点不可达
  - 时间超过
  - 参数问题
  - 改变路由(重定向)

- ICMP 询问报文

ICMP 的一种重要应用就是分组网间探测**ping**,用来测试两台主机之间的联通性

## 5. IPv6

### 5.1 IPv6 的基本首部

IPv6 仍支持**无连接的传送**，但将协议数据单元 PDU 称为**分组**

主要变化：

- **更大的地址空间**
- **扩展的地址层次结构**
- **灵活的首部格式**
- **改进的选项**
- **允许协议继续扩充**
- **支持即插即用(即自动配置)**。IPv6 不需要使用 DHCP
- **支持资源的预分配**
- **IPv6 首部改为 8 字节对齐**。IPv4 首部是 4 字节对齐

#### 5.1.1 IPv6 数据报的一般形式

IPv6 数据报由两大部分组成。

- **基本首部(固定的 40 字节)**
- 有效载荷(净负荷)。允许有 0 个或多个扩展首部，之后才是数据部分

![](https://pic.imgdb.cn/item/6195ec492ab3f51d910b33d8.jpg)

## 6. 虚拟专用网 VPN

- **IP 地址的紧缺**：一个机构能够申请到的 IP 地址数往往小于本机构所拥有的主机数
- **互联网不是很安全**：一个机构内并不需要把所有的主机接入到外部的互联网

### 6.1 本地地址和全球地址

**本地地址**：仅在机构内部使用的 IP 地址，可以由本机构自行分配，而不需要向互联网的管理机构申请

**全球地址**：全球唯一的 IP 地址，需要向互联网的管理机构申请

RFC1918 指明了一些**专用地址**，**专用地址只能用作本地地址**，不能用于全球地址。**在互联网中的所有路由器，对目的地址是专用地址的数据报一律不进行转发**

![](https://s2.loli.net/2021/12/26/OU2Z1cNqHgrDQp5.jpg)

采用上图的专用 IP 地址的互联网称为**专用互联网**(**本地互联网、专用网**)

专用地址仅在本机构内部使用。专用 IP 地址也叫做**可重用地址**

### 6.2 虚拟专用网 VPN

利用公网的互联网作为本机构各专用网之间的通信载体，这样的专用网又称为**虚拟专用网 VPN**

![](https://s2.loli.net/2021/12/26/8QEO6ZbKVWvStjY.jpg)

## 7. 网络地址转换 NAT

- 需要在专用网连接到互联网的路由器上安装 NAT 软件。装有 NAT 软件的路由器叫做**NAT 路由器，它至少由一个有效的外部全球 IP 地址**
- 所有使用本地地址的主机在和外界通信时，都要在 NAT 路由器上**将本地地址转换成全球 IP 地址**，才能和互联网连接

![](https://s2.loli.net/2021/12/26/XBPjsEWZM6bUYJF.jpg)

在内部主机于外部主机通信时，在 NAT 路由器上发生了**两次地址转换**

- 离开专用网时：替换源地址，把内部地址替换成全球地址
- 进入专用网时：替换目的地址，将全球地址替换成内部地址

**网络与端口号转换 NAPT**

NAT 转换表把**运输层的端口号也用上**，可以使多个拥有本地地址的主机，**共用一个 NAT 路由器上的全球 IP 地址**，所以可以同时和互联网上的不同主机进行通信

使用端口号的 NAT 叫做**网络与端口号转换 NAPT**，而不使用端口号的 NAT 叫做**传统的 NAT**