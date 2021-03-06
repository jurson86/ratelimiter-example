# ratelimiter
高可用服务的三驾马车 cache 限流 降级 。这里讨论限流：基于googleapi实现一个简单的限流API 。

   一.限流算法
	常见的限流算法有：令牌桶、漏桶。计数器也可以进行粗暴限流实现。
	1. 令牌桶算法
	    令牌桶算法是一个存放固定容量令牌的桶，按照固定速率往桶里添加令牌。令牌桶算法的描述如下：
	       假设限制2r/s，则按照500毫秒的固定速率往桶中添加令牌；
	       桶中最多存放b个令牌，当桶满时，新添加的令牌被丢弃或拒绝；
	       当一个n个字节大小的数据包到达，将从桶中删除n个令牌，接着数据包被发送到网络上；
	       如果桶中的令牌不足n个，则不会删除令牌，且该数据包将被限流（要么丢弃，要么缓冲区等待）。
     
    2. 漏桶算法
    漏桶作为计量工具（The Leaky Bucket Algorithm as a Meter）时，可以用于流量整形（Traffic Shaping）和流量控制（TrafficPolicing），漏桶算法的描述如下：
        一个固定容量的漏桶，按照常量固定速率流出水滴；
        如果桶是空的，则不需流出水滴；
        可以以任意速率流入水滴到漏桶；
        如果流入水滴超出了桶的容量，则流入的水滴溢出了（被丢弃），而漏桶容量是不变的。
    
   令牌桶和漏桶对比：
    令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
    漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
    令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
    漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
    令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；
    两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。
     
    另外有时候我们还使用计数器来进行限流，主要用来限制总并发数，比如数据库连接池、线程池、秒杀的并发数；只要全局总请求数或者一定时间段的总请求数设定的阀值则进行限流，是简单粗暴的总数量限流，而不是平均速率限流。

   二.限流方式：应用级限流  分布式限流

