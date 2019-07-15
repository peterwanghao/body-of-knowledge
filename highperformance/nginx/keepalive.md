# 2 Nginx+keepalive 实现高可用负载均衡方案

主nginx负载均衡器：172.26.11.99  （**通过keepalived**配置了VIP：172.26.11.101供外使用）

副nginx负载均衡器：172.26.11.100 （**通过keepalived**配置了VIP：172.26.11.101供外使用）

后端web服务器：

172.26.11.73

172.26.11.74

一、172.26.11.99 以及 172.26.11.100的关键nginx配置如下：

vim /etc/nginx/nginx.conf

```
#################
....
upstream  www.xxx.com  {
server   172.26.11.73:8080 max_fails=1;#max_fails 表示健康检查失败的次数，这里表示次数为一次，即标记该服务器down了
server   172.26.11.74:8080 max_fails=1;
}

server
{
listen  80;
server_name  www.xxx.com;

location / {
proxy_next_upstream error timeout http_500 http_502 http_504;  #这里表示健康检查涉及到的情形，有这些情形的，都切换到另外的web服务器访问
proxy_read_timeout 10s;   #这里表示程序返回的时间，请参考php.ini的max_exe_time来设置。
proxy_pass        http://www.xxx.com;
proxy_set_header   Host             $host;
proxy_set_header   X-Real-IP        $remote_addr;
proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

}

#access_log  /var/log/nginx/xxx.log;
}

##########################
```



二、安装keepalive (centos)

```
#安装 popt
yum -y install popt popt-devel

cd /data/software
wget http://www.keepalived.org/software/keepalived-1.2.8.tar.gz
cd /data/src
tar zxf ../software/keepalived-1.2.8.tar.gz
cd keepalived-1.2.8
./configure --prefix=/usr/local/keepalived --sysconf=/etc
make && make install

cp /usr/local/keepalived/sbin/keepalived  /bin/
chkconfig --add keepalived
#设置开机启动
chkconfig keepalived on
#启动keepalive服务
/etc/init.d/keepalived start
```

**如果是ubuntu 直接 apt-get install keepalived 吧….**

三、keepalive设置

cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf_bak

**MASTER**

vim /etc/keepalived/keepalived.conf

```
! Configuration File for keepalived
global_defs {
	notification_email {
		admin@test.com
	}
	notification_email_from admin@test.com
	smtp_server xxx.smtp.com
	smtp_connect_timeout 30
	router_id LVS_DEVEL
}
vrrp_script Monitor_Nginx {
	 script "/root/monitor_nginx.sh"
	 interval 2
	 weight 2
}
vrrp_instance VI_1 {
	state MASTER    #(主机为MASTER，备用机为BACKUP)
	interface eth0  #(HA监测网络接口)

	virtual_router_id 61 #(主、备机的virtual_router_id必须相同)
	#mcast_src_ip 172.26.11.99 #(多播的源IP，设置为本机外网IP，与VIP同一网卡)此项可不设置
	priority 90 #(主、备机取不同的优先级，主机值较大，备份机值较小,值越大优先级越高)
	advert_int 1 #(VRRP Multicast广播周期秒数)
	authentication {
		auth_type PASS #(VRRP认证方式)
		auth_pass 1234 #(密码)
	}

	track_script {
		Monitor_Nginx #(调用nginx进程检测脚本)
	}
	virtual_ipaddress {
.26.11.101 #(VRRP HA虚拟地址)
	}
}
```



**BACKUP方面只需要修改state为BACKUP , priority比MASTER稍低即可**

四、监控nginx进程的脚本：monitor_nginx.sh 内容如下：

vim /root/monitor_nginx.sh

**当检测到nginx进程不存在的时候，就干掉所有的keepalived，这时候，请求将会由keepalived的backup接管！！**

\#!/bin/bashif["$(ps -ef | grep "nginx: master process"| grep -v grep )"==""]then killall keepalivedfi

chmod +x /root/monitor_nginx.sh



172.26.11.99 172.26.11.100都重新启动keepalived：

service keepalived restart



这里请注意，当keepalived启动后，我们可以用命令：

ip add show eth0 来看我们的eth0网卡确实被添加了虚拟IP，如下图：

![nginx+keepalive 实现高可用负载均衡方案](./static/imgr.jpeg)

完毕，可以测试了！