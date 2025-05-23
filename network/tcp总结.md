#####定义
TCP（Transmission Control Protocol，传输控制协议）提供的是面向连接，可靠的字节流服务。
即客户和服务器交换数据前，必须现在双方之间建立一个TCP连接，之后才能传输数据。并且提供超时重发，检验数据，流量控制，拥塞控制等功能，保证数据能从一端传到另一端。  
#####主要关注五个问题
顺序问题，丢包问题，连接维护，流量控制，拥塞控制

- 什么是流量控制  
防止发送方发的太快，耗尽接收方的资源，从而使接收方来不及处理。
- 如何实现流量控制  
使用滑动窗口机制，窗口指的是一次批量的发送多少数据。
在确认应答策略中，对每一个发送的数据段，都要给一个ACK确认应答，收到ACK后再发送下一个数据段，这样做有一个比较大的缺点，就是性能比较差，尤其是数据往返的时间长的时候，
使用滑动窗口，就可以一次发送多条数据，从而就提高了性能
- 什么是拥塞控制  
防止发送方发的太快，使得网络来不及处理，从而导致网络拥塞。  
拥塞控制使用的机制：AIMD\slow start  
slow start: 慢启动
A: additive（加法的）
I: increase（增加）
M: multiplicative（乘法的）
D: decrease（减少）
即就是加法增加，乘法减少---->加增乘减
因特网建议标准RFC2581定义了进行拥塞控制的四种算法，
即慢开始（Slow-start)、拥塞避免（Congestion Avoidance)、快重传（Fast Restrangsmit)和快回复（Fast Recovery）。

- Address already in use产生的原因和解决方法  
你只要记住一句话，在所有 TCP 服务器程序中，调用 bind 之前请设置 SO_REUSEADDR 套接字选项。
这不会产生危害，相反，它会帮助我们在很快时间内重启服务端程序，而这一点恰恰是很多场景所需要的。