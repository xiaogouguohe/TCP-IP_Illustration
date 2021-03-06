# 第5章 Internet协议

## 5.1 引言

- IP提供了一种尽力而为、无连接的数据报交付服务
  - “尽力而为”意味着不保证IP数据报能成功到达目的地
    - 例如缓冲区满时丢弃一些数据
    - 可靠性必须由上层（例如TCP）提供
  - “无连接”意味着IP不维护任何连接状态信息，每个数据包独立于其它数据包
    - 这意味着IP数据包可不按顺序交付

## 5.2 IPv4头部和IPv6头部

- 大小
  - IPv4头部大小为20字节，除非存在选项（少见）
  - IPv6头部大小为40字节，没有选项，扩展头部提供年类似功能
- 字节传输顺序
  - 网络中传输是高位优先
  - 大多数PC使用低位字节优先

### 5.2.1 IP头部字段

- 版本字段

  - 4位
  - IPv4为4，IPv6为6

- 头部长度（IHL）

  - 4位
  - 保存IPv4头部中32位字的数量
    - 因此IPv4头部长度最大位60字节
  - 正常值为5（没有选项的时候）

- 区分服务字段（DS字段）和显式拥塞通知（ECN）字段或指示位

  - 5.2.3节

- 总长度字段

  - IPv4数据包的总长度（包括头部，单位为字节）
  - 16位，因此IPv4数据报的最大长度为65535字节

  - 一些低层协议无法精确表达自己封装的数据报大小
    - 如以太网会把短帧填充到最小长度（64字节），虽然以太网最小有效载荷为46字节，但是一个IPv4数据报可能更小（20字节），如果没有总长度字段，就无法判断一个46字节的以太网帧有没有经过填充
  - 分片
    - 大多数链路层都无法携带这么大的数据，因此需要拆分成更小的片
    - 有些主机不需要接收大于576字节的IPv4数据
      - 很多使用UDP协议（第10章）传输数据（例如DNS、DSHP）的应用程序，限制大小为512字节，以适应主机576字节的限制
    - TCP根据额外信息（第15章）选择自己的数据报大小
    - 第10章详细介绍分片

- 标识字段

  - 序列号

- 标志和分片偏移字段

  - 5.5.3

- 生存期（TTL）

  - 数据包可经过的路由器上限
  - 初始化为某个值
  - 每到达一个路由器，该值-1
  - 到0时，该数据报被丢弃，并使用一个ICMP消息通知发送方（第8章）
  - TTL防止路由环路而导致的数据报永远循环

- 协议字段和下一个头部字段

  - 协议字段是IPv4的头部字段，下一个头部字段是IPv6的头部字段
  - 标识数据报有效载荷部分的数据类型
    - 17为UDP
    - 6为TCP

- 头部校验和字段

  - 仅计算IPv4头部，这意味着IP协议不检查IPv4数据报有效载荷
  - 因此上层协议（TCP或UDP）需要自己的数据完整性机制来检查
  - IPv6没有任何校验和字段，有以下的里有
    - 更高层协议会有自己的校验和
    - IP头部的错误比较少见，因为很少发生位错误
  - 当一个IPv4数据报经过一台路由器时，TTL字段-1，因此头部校验和必须改变（5.2.2）
  - 源IP和目的IP

### 5.2.2 Internet校验和

- ###具体计算过程略

### 5.2.3 DS字段和ECN

- 区分服务
  - ###
- ECN
  - 用于位数据报标记拥塞标识符
  - 一台持续拥塞的具有ECN感知功能的路由器在转发分组时会设计这两位
  - 当一个被标记的分组被目的节点接收时，有的协议（TCP）会发现分组被标记，并通知发送方，发送方会降低速度，避免拥塞（第16章）
- 虽然DS字段和ECN字段并不密切相关，但它们被用作代替以前定义的IPv4服务类型和IPv6流量类别字段，所以常被放在一起讨论
- ###

### 5.2.4 IP选项

- IPv4选项的组成
  - 类型字段
    - 长度1字节
    - 第一位（复制）
      - 0：仅在第一个分片中复制
      - 1：复制到所有分片
    - 第二位和第三位（类别）
      - 00：数据报控制
      - 01：保留
      - 10：排错和管理
      - 11：保留
    - 第四位到第八位（编号）
      - 表5-4的“编号”列
  - 长度字段
    - 长度1字节，表示选项的总长度
  - 选项的值
    - 可变长度
- 多数IP选项很少使用
  - IP头部空间有限
  - 选项主要用于诊断目的，为防火墙的构建带来风险（第7章）

## 5.3 IPv6扩展头部

- 在IPv6中，由IPv4选项提供的特殊功能，通过在IPv6之后增加扩展头部来实现
- IPv6头部的下一个头部字段
  - 标识紧跟着的头部的类型
  - 头部的类型，头部链的顺序和下一个头部字段的值见表5-5
  - ###

- 下面介绍一些扩展头部的类型

### 5.3.1 IPv6选项

- IPv6选项携带在逐跳（H）选项或目的地（D）选项扩展头部中（见表5-5的扩展头部类型），也就是这是一种头部
- IPv6选项格式见图5-7
  - 第一二位
    - 动作字段，如果这个IPv6选项没被某个节点识别，这个节点如何处理这个数据报，见表5-6
    - 如果一个组播数据报中包含未知选项（或者说节点可能识别不了的选项），可以把动作位设置为11来避免
  - 第三位
    - 当宣咸数据可能在转发过程改变时，该字段设为1
  - 表5-7？？？
  - 第四到八位
    - 类型字段
  - 第2个字节
    - 选项数据长度（整个选项的长度）

#### 5.3.1.1 填充1和填充N

- IPv6选项要对齐8字节，因此用0填充
- 两种填充方式
  - 填充1选项是唯一缺少长度字段和值字段的选项，只有1字节长，全0
  - 填充N选项格式如5-7

#### 5.3.1.2 IPv6超大有效载荷

- IP数据报大小限制为64KB
- IPv6超大有效载荷选项指定了有效载荷大于65535字节的IPv6数据报，称为超大报文
- 该选项长度为32位
- 当一个超大报文形成时，IPv6头部的负载长度字段被设置为0，真正的的长度来自选项的长度值，选项的哪一部分？？？

#### 5.3.1.3 隧道封装限制

###

#### 5.3.1.4 路由器警告

- 与IPv4的路由器警告选项目的相同

#### 5.3.1.5 快速启动

- 适用于IPv4和IPv6，但建议用于专用网络而非全球性Internet
- 用于调整发送速率###

#### 5.3.1.6 CALIPSO

###

#### 5.3.1.7 家乡地址

- 用于移动IP，见5.5

### 5.3.2 路由头部

- 控制数据报通过网络的路径
- 路由扩展头部由两个不同版本，类型0（RH0）和类型2（RH2）

#### 5.3.2.1 RH0

- 出于安全方面的考虑被否决
- 头部结构见图5-8
  - 路由类型标识符
    - RH0：值为0
    - RH2：值为2
  - 剩余部分字段
    - 到达最终目的地址前仍需访问的目的IP地址中的节点数
  - 目的IP地址字段
    - 可能有一个或多个，是转发路径上的IPv6地址
- 具体路由过程见图5-9

#### 5.3.2.2 RH2

- RH2取代RH0的优势
  - RH0的目的IP地址字段中，可能会有重复的IP，导致重复转发
  - RH2的目的地址只有一个，这和IPv6头部的目的地址有什么不同，是只能控制一个中间节点吗？？？

### 5.3.3 分片头部

- 分片是为了发送一个大于路径MTU的数据报
  - 如何确定路径MTU见第13章
  - 1280字节是针对IPv6定义的链路层最小MTU
- 分片的时机
  - IPv4中，任何主机或路由器可将该数据报分片
    - 第二个32位字段表示分片信息，见5.2.1
  - IPv6中，仅数据报的发送者可以执行分片操作
- IPv6分片头部，见图5-11
  - 分片偏移字段
    - 和IPv4一样，分片偏移字段给住了有效载荷在原始数据报中以8字节为单位的偏移量（相对于原始数据报的可分篇部分）
  - 可分片部分和不可分片部分
    - 每个分片都会包含一个原始数据报中不可分片部分的副本
    - 不可分片的头部包括IPv6头部和任何在到达目的地之前需由中间节点处理的扩展头部（如果只在目的节点处理，那么只需要组合起来分片，就可以得到一个头部了，不需要每个分片携带一个头部）
      - 路由头部以及之前的所有头部
      - 如果有逐跳选项，则是它之前的所有头部
    - 每个分片的IPv6头部的负载长度字段需要改成这个分片的大小
    - 需要处理分片偏移字段
    - 标识符字段，也就是序列号，所有分片都一样
    - 最后一个分片的M（更多分片）位字段设置位0
  - 分片演示过程见图5-12

## 5.4 IP转发

- 是否使用路由器，取决于目的地是否是直接相连的主机（例如点到点链接）或共享网络（以太网），是则直接发送到目的地，否则发送到一台路由器，由该路由器将数据报交付到目的地
- 主机和路由器处理IP数据报的区别在于，主机不转发不是由它生成的数据报，但是路由器会这样做

### 5.4.1 转发表（路由表）

- 路由表条目应当包含的信息
  - 目的地
    - 对于涵盖所有目的地的“默认路由”，目的地可设为0
    - 对于仅描述一个目的地的“主机路由”，目的地设为具体的IP地址
  - 掩码
    - 前面若干高位为1，其余为0
    - 和目的IP地址做按位与操作，找到目的IP地址所在的子网
  - 下一跳
    - 下一个IP实体的IP地址
  - 接口
    - 和下一跳有什么区别？？？
- 有多种路由协议来确保路由表正确，如RIP、OSPF、BGP和IS-IS

### 5.4.2 IP转发行动

- 最长前缀匹配算法
  - 目的地址与路由表的每个条目进行按位与，并将得到的结果和该条目的目的IP比较，则每个条目会得到一个比较后的结果，匹配位数最多的结果，转发到它的下一跳
  - 如果没有匹配的条目，则会给发送方一个“主机不可达“的错误，通常是ICMP消息

### 5.4.3 例子

- 相同的网络前缀（子网内部）使用直接交付

#### 5.4.3.1 直接交付

- 交付过程见图5-16
- 转发表见表5-8
  - 与第二个条目匹配得更好（见5.4.2），因此用这个条目的下一跳

#### 5.4.3.2 间接交付

###

### 5.4.4 讨论

###

## 5.5 移动IP

###

## 5.6 IP数据报的主机处理

- 主机必须考虑把哪个IP地址放在分组的源IP地址和目的IP地址字段中（路由器可以不考虑），因为发送方和接收方都可能有多个IP地址（5.6.2）
- 如果数据报到达一个错误的接口，或者说这个接口配置的IP地址不在数据报的目的地址里，那么是否接收并处理数据报？（图5-20）
  - 由接收方的主机模式，以及它是否为最相关的多宿主主机决定（5.6.1）

### 5.6.1 主机模式

- 决定一个单播数据报是否匹配一台主机的IP地址
  - 强主机模式
    - 数据报携带的目的地址和数据报到达的接口配置的IP地址匹配时，才交付
  - 弱主机模式
    - 数据报携带的目的地址可以和它到达的任何接口的任何本地地址匹配
- 主机模式也适用于发送行为
  - 在强主机模式下，当某个接口，它配置的地址与发送数据报的源IP地址匹配时这台主机才可以从这个接口发送数据报
- 强主机模式的问题
  - 过于“严格”，见图5-20的例子，A是强主机模式，B发送一个目的地址为203.0.113.1的数据报给A，如果是从本地网络发送，则会被A拒绝，但是从用户角度来看，不应该被拒绝
- 强主机模式的优势
  - 安全问题，例如假设B是弱主机模式，那么网络中生成一个目的地址为203.0.113.2，源地址为203.0.113.1的分组（伪造源IP），并且发送给B，那么B就会接受这个分组，相信这个分组来源于A

### 5.6.2 地址选择

- 发送方和接收方都可能有多个IP地址
- 如何用TCP管理地址（第13章）

- 源地址选择程序和目的地址选择程序
  - ###

## 5.7 与IP相关的攻击

- 针对IP选项的攻击，或者无效的头部字段
- 伪造源IP地址
  - 通过入口过滤检查流量的源地址，以确保数据报包含一个指定的IP前缀
- IPv6和移动IP相对较新，很多漏洞尚未发现

