# connectorConfig setNoDelay选项

## 介绍Nagle算法
[百度百科](http://baike.baidu.com/item/Nagle%E7%AE%97%E6%B3%95)

### 重点

```
从键盘输入的一个字符，占用一个字节，可能在传输上造成41字节的包，其中包括1字节的有用信息和
40字节的首部数据。这种情况转变成了4000%的消耗，这样的情况对于轻负载的网络来说还是可以接受
的，但是重负载的福特网络就受不了了，它没有必要在经过节点和网关的时候重发，导致包丢失和妨碍
传输速度.吞吐量可能会妨碍甚至在一定程度上会导致连接失败。

Nagle的算法通常会在TCP程序里添加两行代码，在 未确认 数据发送的时候让发送器把数据送到缓存里。
任何数据随后继续直到得到明显的数据确认或者直到攒到了一定数量的数据了再发包

默认情况下，发送数据采用Nagle 算法。这样虽然提高了网络吞吐量，但是实时性却降低了，在一些交互
性很强的应用程序来说是不允许的，使用TCP_NODELAY选项可以禁止Nagle 算法。
此时，应用程序向内核递交的每个数据包都会立即发送出去。需要注意的是，虽然禁止了Nagle 算法，但
网络的传输仍然受到TCP确认延迟机制的影响。


举个例子，client端调用socket的write操作将一个int型数据（称为A块）写入到网络中，由于此时
连接是空闲的（也就是说还没有未被确认的小段），因此这个int型数据会被马上发送到server端，接
着，client端又调用write操作写入‘\r\n’（简称B块），这个时候，A块的ACK没有返回，所以可以认
为已经存在了一个未被确认的小段，所以B块没有立即被发送，一直等待A块的ACK收到（大概40ms之
后），B块才被发送。

这里还隐藏了一个问题，就是A块数据的ACK为什么40ms之后才收到？这是因为TCP/IP中不仅仅有nagle
算法，还有一个TCP确认延迟机制 。当Server端收到数据之后，它并不会马上向client端发送ACK，而
是会将ACK的发送延迟一段时间（假设为t），它希望在t时间内server端会向client端发送应答数据，
这样ACK就能够和应答数据一起发送，就像是应答数据捎带着ACK过去。在我之前的时间中，t大概就是
40ms。这就解释了为什么'\r\n'（B块）总是在A块之后40ms才发出。

　　当然，TCP确认延迟40ms并不是一直不变的，TCP连接的延迟确认时间一般初始化为最小值40ms，随
　　后根据连接的重传超时时间（RTO）、上次收到数据包与本次接收数据包的时间间隔等参数进行不断调整。
　　另外可以通过设置TCP_QUICKACK选项来取消确认延迟。


```

## 测试情况
加了这个参数配置，反而比没有加的网络延迟情况更差。