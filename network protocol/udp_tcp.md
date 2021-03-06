### UDP

UDP：用户数据报协议。

#### 特点 

面向无连接，在发送数据之前不需要像TCP一样建立连接再发送，要发数据的话直接发就可以了。

数据报文搬运工，只对数据转发，不会有拼接拆分等操作。

传输方式，有一对一，有多对多，有多对一。

不可靠，没有拥塞机制，以恒定的速度来发送数据，即使网络不好，也不会对发送数据进行调整。

看起来好像缺点很多，但是也看应用场景，在对实时性要求比较高的场景中，反而需要UDP而不是TCP，比如电话/视频会议，丢点包，问题也不大。



#### 头部构成

1. 端口号，包括源端口和目标端口
2. 整个数据报文长度
3. 整个数据报文的**检验**和（IPv4 可选 字段），该字段用于发现头部信息和数据中的错误



因为头部构成比TCP少，所以传输的时候相对来说是比较高效的。



### TCP

TCP：传输控制协议

#### 特点

面向连接的，可靠的（因为只有确认连接之后才会开始发送数据）的传输层通信协议。



为什么 TCP 建立连接需要三次握手，而不是两次？这是为了防止出现**失效的连接请求报文段被服务端接收**的情况，从而产生错误。

❓ 具体怎么理解这个失效的连接请求被服务端接受？这句话主要出自计算机网络那本教材。

目的应该是确认状态吧，三次是在最少的情况下确认双方都知道，对方已经知道并且建立连接。



看一下TCP的状态位和重要概念，主要是状态码：

SYN 是发起一个连接，ACK 是回复，RST 是重新连接，FIN 是结束连接。TCP 是面向连接的，因而双方要维护连接的状态，这些带状态位的包的发送，会引起双方的状态变更。



#### 流量控制

还有一个重要的就是**窗口大小**。TCP 要做流量控制，通信双方各声明一个窗口，标识自己当前能够的处理能力，别发送的太快，撑死我，也别发的太慢，饿死我。

发送端发送的每一个数据包，接收端都要给一个确认包（ACK），确认它收到了。 接收端给发送端发送的确认包（ACK包）中，同时会携带一个窗口的大小。 **这个窗口的大小就代表目前接收端的处理能力**（接收端最大缓存量-接收已确认但还未被应用层读取的部分）。 

如果接受端确认接受数据却一直不处理数据，导致窗口越来越小，发送给发送端的确认包中的携带的窗口大小也越来越小，最后小的0，发送端就停止发送。

如果这样的话，发送端会定时发送**窗口探测数据包**，看是否有机会调整窗口的大小。当接收方比较慢的时候，要防止低能窗口综合征，别空出一个字节来就赶快告诉发送方，然后马上又填满了，可以当窗口太小的时候，不更新窗口，直到达到一定大小，或者缓冲区一半为空，才更新窗口。

#### 拥塞阻塞

作为老司机，做事情要有分寸，待人要把握尺度，既能适当提出自己的要求，又不强人所难。除了做流量控制以外，TCP 还会做**拥塞控制**，对于真正的通路堵车不堵车，它无能为力，唯一能做的就是控制自己，也即控制发送的速度。不能改变世界，就改变自己嘛。



拥塞控制的问题，也是通过窗口的大小来控制的，前面的滑动窗口 rwnd 是怕发送方把接收方缓存塞满，而拥塞窗口 cwnd，是怕把网络塞满。

这里有一个公式 LastByteSent - LastByteAcked <= min {cwnd, rwnd} ，是拥塞窗口和滑动窗口共同控制发送的速度。

TCP 发送包常被比喻为往一个水管里面灌水，而 TCP 的拥塞控制就是在不堵塞，不丢包的情况下，尽量发挥带宽。**TCP 的拥塞控制主要来避免两种现象，包丢失和超时重传**。



一旦出现了这些现象就说明，发送速度太快了，要慢一点。但是一开始我怎么知道速度多快呢，我怎么知道应该把窗口调整到多大呢？如果我们通过漏斗往瓶子里灌水，我们就知道，不能一桶水一下子倒进去，肯定会溅出来，要一开始慢慢的倒，然后发现总能够倒进去，就可以越倒越快。这叫作**慢启动**。

一条 TCP 连接开始，cwnd 设置为一个报文段，一次只能发送一个；当收到这一个确认的时候，cwnd 加一，于是一次能够发送两个；当这两个的确认到来的时候，每个确认 cwnd 加一，两个确认 cwnd 加二，于是一次能够发送四个；当这四个的确认到来的时候，每个确认 cwnd 加一，四个确认 cwnd 加四，于是一次能够发送八个。可以看出这是**指数性的增长**。

涨到什么时候是个头呢？有一个值 ssthresh 为 65535 个字节，当超过这个值的时候，就要小心一点了，不能倒这么快了，可能快满了，再慢下来。每收到一个确认后，cwnd 增加 1/cwnd，我们接着上面的过程来，一次发送八个，当八个确认到来的时候，每个确认增加 1/8，八个确认一共 cwnd 增加 1，于是一次能够发送九个，变成了线性增长。

但是线性增长还是增长，还是越来越多，直到有一天，水满则溢，出现了拥塞，这时候一般就会一下子降低倒水的速度，等待溢出的水慢慢渗下去。



**快速重传算法**：当接收端发现丢了一个中间包的时候，发送三次前一个包的 ACK，于是发送端就会快速地重传，不必等待超时再重传。TCP 认为这种情况不严重，因为大部分没丢，只丢了一小部分，cwnd 减半为 cwnd/2，然后 sshthresh = cwnd，当三个包返回的时候，cwnd = sshthresh + 3，也就是没有一夜回到解放前，而是还在比较高的值，呈线性增长。



**但是**！！

TCP 的拥塞控制主要来避免的两个现象都是有问题的。

**第一个问题**是丢包并不代表着通道满了，也可能是管子本来就漏水。例如公网上带宽不满也会丢包，这个时候就认为拥塞了，退缩了，其实是不对的。

**第二个问题**是 TCP 的拥塞控制要等到将中间设备都填充满了，才发生丢包，从而降低速度，这时候已经晚了。其实 TCP 只要填满管道就可以了，**不应该接着填，直到连缓存也填满**。

为了优化这两个问题，后来有了 TCP BBR 拥塞算法。它企图找到一个平衡点，就是通过不断地加快发送速度，将管道填满，但是不要填满中间设备的缓存，因为这样时延会增加，在这个平衡点可以很好的达到高带宽和低时延的平衡。

总结：填满管道，但不填满缓存，如果缓存满了会增加时延。

（这个缓存是哪里的缓存？个人理解是接受方的吧，那么传输过程中服务器上缓存算吗？🤔）



#### 传输中的丢包

万一丢包了，TCP会进行什么样的处理？

1. 超时重传，对发送了却没有ACK的包设置一个定时器，如果超过这个时间就重新发送。那么这个时间是怎么确定的？简单来说是对多次往返的时间进行加权处理，并且这个值是会根据网络状态发生变化的。
   以及，TCP 的策略是**超时间隔加倍**。每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送。
2. 快速重新传：
   当接收方收到一个序号大于下一个所期望的报文段时，就会检测到数据流中的一个间隔，于是它就会发送冗余的 ACK，仍然 ACK 的是期望接收的报文段。而当客户端收到三个冗余的 ACK 后，就会在定时器过期之前，重传丢失的报文段。

3. 还有一种方式称为 Selective Acknowledgment （SACK）。这种方式需要在 TCP 头里加一个 SACK 的东西，可以将**缓存的地图**发送给发送方。例如可以发送 ACK6、SACK8、SACK9，有了地图，发送方一下子就能看出来是 7 丢了。

