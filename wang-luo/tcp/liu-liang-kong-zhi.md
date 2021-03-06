# 流量控制

​     流量控制方面主要有两个要点需要掌握。一是TCP利用滑动窗口实现流量控制的机制；二是如何考虑流量控制中的传输效率。

1. 流量控制
     所谓流量控制，主要是接收方传递信息给发送方，使其不要发送数据太快，是一种端到端的控制。主要的方式就是返回的ACK中会包含自己的接收窗口的大小，并且利用大小来控制发送方的数据发送：
![img](http://blog.chinaunix.net/attachment/201402/17/26275986_1392627535jeG5.png)
     这里面涉及到一种情况，如果B已经告诉A自己的缓冲区已满，于是A停止发送数据；等待一段时间后，B的缓冲区出现了富余，于是给A发送报文告诉A我的rwnd大小为400，但是这个报文不幸丢失了，于是就出现A等待B的通知||B等待A发送数据的死锁状态。为了处理这种问题，TCP引入了持续计时器（Persistence timer），当A收到对方的零窗口通知时，就启用该计时器，时间到则发送一个1字节的探测报文，对方会在此时回应自身的接收窗口大小，如果结果仍未0，则重设持续计时器，继续等待。
2. 传递效率
     一个显而易见的问题是：单个发送字节单个确认，和窗口有一个空余即通知发送方发送一个字节，无疑增加了网络中的许多不必要的报文（请想想为了一个字节数据而添加的40字节头部吧！），所以我们的原则是尽可能一次多发送几个字节，或者窗口空余较多的时候通知发送方一次发送多个字节。对于前者我们广泛使用Nagle算法，即：
*1. 若发送应用进程要把发送的数据逐个字节地送到TCP的发送缓存，则发送方就把第一个数据字节先发送出去，把后面的字节先缓存起来；
*2. 当发送方收到第一个字节的确认后（也得到了网络情况和对方的接收窗口大小），再把缓冲区的剩余字节组成合适大小的报文发送出去；
*3. 当到达的数据已达到发送窗口大小的一半或以达到报文段的最大长度时，就立即发送一个报文段；
     对于后者我们往往的做法是让接收方等待一段时间，或者接收方获得足够的空间容纳一个报文段或者等到接受缓存有一半空闲的时候，再通知发送方发送数据。