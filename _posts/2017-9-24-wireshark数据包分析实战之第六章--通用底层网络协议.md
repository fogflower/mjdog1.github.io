#读wireshark数据包分析实战之第六章---通用底层网络协议

这一章主要是集中在OSI分层模型中从第1层到第四层的底层协议。（作为一个回顾当然也可以作为一个开始，内容较为实用）


## 地址解析协议
网络里面存在着逻辑地址和物理地址。逻辑地址用在不同的网络直接以及没有直接相连的设备之间进行通信。而物力地址则用在单一网络中交换机直接进行通信。大多数情况下都是两者共同起作用的。

TCP/IP网络（基于ipv4）中用来将IP地址解析为MAC地址的过程叫做地址解析协议（Address Resolution Protocol）。它的解析包含两个数据包：一个ARP请求与一个ARP响应。

这里有一个故事,有个机器A，他只知道小伙伴的IP地址，想通过arp来知道小伙伴的物理地址。首先他给周围的人广播，“大家好，我是A,我的IP地址是，XX.XX.XX.XX,MAC地址是XX.XX.XX.XX。我想要一个IP地址为XX.XX.XX.XX小伙伴的MAC地址，谁有这个IP地址，可否回复我MAC地址”。当不是这个IP地址的主机则丢弃这个数据包，拥有这个IP地址的设备则返回一个ARP响应。大致就是返回我是IP为XX.XX.XX.XX的，我的MAC是XX.XX.XX.XX。

### ARP头结构

1. 硬件类型：数据链路层使用的类型数据。大多数情况下，都是以太网。
2. 协议类型：ARP请求的高层协议。如IP协议
3. 硬件地址长度：正在使用的硬件地址的长度(单位为字节)（一般为6个字节）
4. 协议地址长度：对于指定协议类型使用的逻辑地址的长度。（单位字节）一般四个字节。
5. 操作(opcode)：ARP数据包的功能:0x001表示请求，0x002表示相应
6. 发送方硬件地址：发送方MAC地址
7. 发送方协议地址：发送方的高层协议地址即IP地址。
8. 目标硬件地址：目标接收方的硬件地址。（ARP请求中为0）
9. 目标协议地址：目标接收方的高层协议地址。（IP地址）
![@ARP请求数据包|center](./arp图片.png)

在图中可以看到1时，发送的是一个广播报，这个数据的源地址2就是我们的MAC地址。从3可以得知我们的操作码（opcode）为请求，即发送ARP请求。在3下面列出了所有的源IP地址与MAC地址，以及目的的IP地址以及未知的MAC地址。

![@ARP返回|center]](./arg返回.png)

图片中列举了返回的ARP文件，可以看到1发生了改变，发送方和接收方出现了交换。最重要的变化是在2中，opcode编程了0x0002表示响应包。3也发生了发送方与接收方的转换。

还有一个小点就是存在无偿的ARP转发：一般都出现在一个主机的IP地址发生改变，原先的MAC地址与IP的地址映射发生改变，需要更新这些映射的时候需要进行所谓的无偿ARP转发。

## 互联网协议
位于OSI模型中第3层的协议，网络层，负责跨网络通信在这个层上，最普遍的是互联网协议（IP）。
互联网本身是由无数路由器和局域网组成的一个集合。
### IP地址
IP地址是一个32位的地址，用来唯一标识连接到网络的设备。并且为了方便起见，把32位长度的地址分层四段。分别为0~255值域内的数据。
之所以将网络地址分为四部分，是因为每个IP地址都包含着两个部分：网络地址和主机地址。**网络地址**用来标识设备所连接到的局域网，而**主机地址**则标识网络中的设备本身。而决定这个IP地址的网络地址与主机地址的还需要一个条件，即**子网掩码**的地址信息来决定。一般会在IP地址后面加上斜杠(/)来表示网络地址与主机地址的关系。10.10.1.22/16表示网络地址为16位
### IPv4 头
![Alt text](colacs.cn/images/2017-9-26-wiresharkAnalysis/2017-9-26-ipv4.jpg)
版本号(Version): IP所使用的版本
首部长度(Header Length):IP头的长度
服务类型（Type of Service）: 优先级标志位和服务类型标志位，被路由器用来进行流量的优先排序。
总长度（Total Length）: IP头与数据包中数据的长度。
标识符（Identification）：一个唯一的标识数字，用来识别一个数据包或者被分片数据包的次序。
标记（Flags）:用来标记一个数据包是否是一组分片数据包的一部分。
分片偏移（Fragment Offset）:一个数据包是一个分片，这个域中的值就会被用来将数据包以正确的顺序重新组装。
存活时间（Time to Live）:用来定义数据包的生存周期，以经过路由器的跳数/秒数进行描述。
协议（Protocol）：用来识别在数据包序列中上层协议数据包的类型。
首部校验和（Header Checksum）:一个错误检测机制，用来确定IP头的内部没有被损坏或者篡改。
源IP地址（Source IP Address）:发出数据包的主机的IP地址。
目的IP地址（Destination IP Address）:数据包目的地的IP地址
选项（Options）:保留作额外的IP的选项。它包含着源站选路和时间戳的一些选项。
数据（Data）:使用IP传递的实际数据。
### 存活时间
存活时间（TTL）值定义了数据包被丢弃之前，所能经历的时间，或者能够经历的最大路由数目。在数据包创建的时候被定义，当经过一个路由器之后就发生了减1操作。如果在某一个路由器中，已经减到0那么就会发生丢弃的情况。
这对于类似于死循环是很有帮助的，可以检测这种TTL，这样就可以避免死循环。
给出一个请求的IP头
![Alt text](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/ipsource.png)
在编号的图中，1代表版本号为4，2代表IP的头有20个字节，3代表首部和载荷总共有60个字节，4代表TTL域的值为128。5表示源地址，6代表目标地址。这里用来了ICMP协议来验证接收主机是否发生响应。这个数据包是由发送主机创建的。
下图是接收端捕获到的数据，可以看到TTL减少了1，说明在发送端到接收端的路径中，只存在1个路由器。
![Alt text](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/ipdest.png)

### IP分片
数据包分片是将一个数据流分成若干个更小的片段，是IP用于解决跨越不同类型网络时可靠传输的一个特性。

一个数据包的分片是基于第2层数据链路层协议里所使用的最大传输单元（Maximum Transmission Unit, MTU）的大小，以及使用这些第2层协议的设备配置情况。这个MTU默认是1500，但可以修改。

当一个设配需要进行传输IP数据包的时候，它将会比较这个数据包的大小，以及将要把这个数据包发送出去的网络接口MTU，用于决定是否需要将这个数据包分片。主要步骤如下：
1. 设备将数据分为若干个可成功进行传输的数据包
2. 每个IP头的总长度域会被设置为每个分片的片段长度。
3. 更多分片标志将会在数据流的所有数据包中设置为1，除了最后一个数据包。
4.  IP头中分片部分的分片偏移将会被设置。
5.  数据包被发送出去

接下来举一个列子：
我们以一个ICMP数据包来分析，它分为三块，首先给出整体图形：
![image](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/ipfenpian.png)
第一段的数据包为
![Alt text](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/ipfenpian-frag1.png)
可以看到这里的Flags被设置成：0x01表示还存在更多的数据包，并不是只有这一个。Fragment offset: 0 表示当前的偏移为0，即可以表示分片数据包的开始位置。

![Alt text](https://raw.githubusercontent.com/mjdog1/mjdog1.github.io/master/images/2017-9-26-wiresharkAnalysis/ipfenpian-frag2.png)
同样的，可以看到和上面除了Fragment offset 不同为1480，其他都相同，由于MTU默认为1500,减去IP头大小默认为20，所以剩下的数据为1480。且Flags为：0x01还存在分片。

![Alt text](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/ipfenpian-frag3.png)
图中，Flags为0x00表示已经是最后一个了。同样的offset为2960表示前面两个存放的数据为2960.这里面还有一个重要的点就是这三个分片为同一个数据，因此他们的标识（Identification）都是一样的。

## 传输控制协议
传输控制协议（Transmission Control Protocol,TCP）的最终目的是为数据提供可靠的端到端的传输。工作在OSI模型中第4层。 

### TCP头
![Alt text](https://github.com/mjdog1/mjdog1.github.io/blob/master/images/2017-9-26-wiresharkAnalysis/tcp_head.png)

源端口（source port）:用来传输数据包的端口
目的端口（Destination port）：数据包将要被发送的端口。
序号（Sequence Number ）: 这个数字用来 表示一个TCP片段，这个域用来保证数据流中的部分没有流失。
确认号（Acknowledgment Number）:这个数字是通信中希望从另一个设备得到的下一个数据包的序号。
标记(Flags): URG、ACK、PSH、RST、SYN和FIN标记都是用来表示所传输的TCP数据包的类型。
窗口大小（Window Size）:TCP接收者缓冲的字节大小
校验和（Checknum）:用来保证TCP头和数据的内容在抵达目的地时的完整性。
紧急指针（Urgent Pointer）：如果设置了URG位，这个域被用来检查作为额外的指令，告诉CPU从数据包的哪里开始读取数据。
选项（Options）：各种可选域，可以在TCP数据包中进行指定。

### TCP端口
使用TCP进行通信的时候，有65535个端口可以使用，一般分为两个部分：
1. 1~1023 是标准端口组，特定的服务会用到，
2. 1024~65535 是零时端口组，当一个服务想在任意时间进行通信的时候，现代操作系统都会随机选择一个源端口，让这个同喜使用唯一源端口。

## 用户数据报协议

## 互联网控制协议
|                           大家好我是狗哥              |
| Col1      |     Col2 |   Col3   | dajia |
| :-------- | --------:| ------:    | --		 |
| field1    |   field2 |  field3  |大家好  |
