# 1 Nginx负载均衡策略

选择方式：

1. 随机（Random）选择
2. Hash选择
3. （Round-Robin）选择 根据地址列表按顺序选择
4. 按权重（Weight）选择
5. 按负载（Load）选择
6. 按连接（Connection）选择



为了保证访问时跳过出问题的机器，通常采用的方法是负载均衡机器定时和实际的业务处理机器进行心跳（ping、端口检测或是url侦测），发现心跳失败的机器即将其从可用地址列表中拿掉，在心跳成功后再重新加入可用地址列表。



要达到响应直接返回的效果，须要采用IP Tunneling或DR（Direct Routing，硬件负载设备中又简写为DSR：Direct Service Routing）方式



## 1.轮询（默认方式）

对于一级后端服务器群，形成一个环队列的形式，对于每个到达的请求按时间顺序顺次分配给这些后端服务器。在前端调度器与后端服务器之间采用“心跳”方式进行状态检查，如果发现后端服务器宕机，则将其删除。

​    这种方式为默认配置，优点是简洁，但缺点是无法进行最优化调度，有可能有的请求需要耗时较久，这样会带来一定的不平衡。

​    它的例子：在http区域里添加：

upstream lb {

​        server 10.10.57.122:80;

​        server 10.10.57.123:80;

}

在你的某个server里增加：

location / {

​              proxy_pass http://lb;

​        }



## 2.加权轮询

​    这是一种对上述方式的改进，引入权值的概念，能够解决后端服务器性能不均的情况。



​    例如这样一个配置：

​    upstream lb {

​         server 10.10.57.122:80 weight=5;

​         server 10.10.57.123:80 weight=10;

​    }



ps：以上轮询负载均衡策略，我个人认为对于动态网站应用，这几乎就是形同摆设，没有人会采用。但一种情况例外：服务器端的session采用共享机制，如存储在数据库或者memcached内存里等。

## 3.ip_hash(基于ip的hash分配策略)

​     这是一种非轮询式方式，对于每个到达的请求，直接通过其请求IP进行哈希的映射，通过映射结果获得那一台后端服务器要处理这个请求，这种方式有一个明显的好处是能够保证session的唯一性。

​    它的配置例子：

upstream lb {

​        ip_hash;

​        server 10.10.57.122:80;

​        server 10.10.57.123:80;

}

在你的某个server里增加：

location / {

​              proxy_pass http://lb;

​        }



   

This directive causes requests to be distributed between upstreams based on the IP-address of the client.

The key for the hash is the class-C network address of the client. This method guarantees that the client request will always be transferred to the same server. But if this server is considered inoperative, then the request of this client will be transferred to another server. This gives a high probability clients will always connect to the same server.

It is not possible to combine ip_hash and weight methods for connection distribution. If one of the servers must be removed for some time, you must mark that server as *down*.

由这段英文解说知道，**客户端只要来自同一网段的ip的request都会转发到相同的后端服务器上**。这里的所谓的class-C network这里作者并没有很详细地解释，我只能说，写这句话的人不懂网络。**我个人的理解是：以ip地址的点分十进制格式的前3个字节进行hash**。

其实这不是真正意义上的ip address hash，而只是network address hash。真正的ip address hash方式有不？其实可以通过下面介绍的url_hash来实现。关键指令：hash $remote_addr;

不过这里有个前提，$remote_addr必须是client的real ip address。

为什么这里能够实现真正意义上的ip address hash？很简单，就是这里整个ip address被当作一个字符串来对待，故只要ip地址（key）不同，hash必然也是不同的。



## 4.url_hash(基于URL的哈希方式)

​    这种方式与IP的哈希方式类似，是对客户机请求的URL进行哈希操作，这样的方式有一个明显的好处是，能够便于内容缓存的实现，对于经常性的资源访问，采用这样的方式会获得非常好的质量。它目前不是nginx自带的功能，需要安装补丁方可使用。本指令的详细说明和安装见：(文章后面有附带详细安装实例)

http://wiki.nginx.org/HttpUpstreamRequestHashModule



​    它的配置方式为：

  

upstream lb {

​         server 10.10.57.122:80;

​         server 10.10.57.123:80;

​         hash $request_uri;

​    }



如果将这里的$request_uri换成$remote_addr便可实现上面我所说的真正基于ip地址的策略。

## 5.基于服务响应式

   这种方式是根据服务器端的动态响应，对每一个请求进行分配。 这种方式能够自动根据当前的后端实际负载来优化。

   它的配置方式：

upstream lb {

​         server 10.10.57.122:80;

​         server 10.10.57.123:80;

​         fair;

​    }



这个没怎么测试，只是配起来用了下，机器差不多的话感觉就是轮询。

===============================

cd nginx-0.7.67;

patch -p0 < ../nginx_upstream_hash-0.3.1/nginx.patch

configure时：

./configure --prefix=/app/nginx -user=nobody -group=nobody \

--add-module=../nginx_upstream_hash-0.3.1/ \

--add-module=../gnosek-nginx-upstream-fair-2131c73/



负载均衡机器为避免自己成为单点，通常由两台机器构成，但只有一台处于服务状态，另一台则处于standby状态。一旦处于服务的那台机器出现问题，standby这台会自动接管。



软件负载方案中最常用的为LVS（Linux Virtual Server），多数情况下采取LVS+Keepalived来避免负载均衡机器的单点。实现负载均衡机器的自动接管。