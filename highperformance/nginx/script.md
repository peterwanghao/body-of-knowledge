# 4 监控Nginx负载均衡器脚本

**监控Nginx负载均衡器脚本**

**1.编写脚本**

vim nginx_pid.sh

\#!/bin/bash

while  :

do

nginxpid=`ps -C nginx --no-header | wc -l`

if [ $nginxpid -eq 0 ];then

/usr/local/nginx/sbin/nginx

sleep 5

  if [ $nginxpid -eq 0 ];then

  /etc/init.d/keepalived stop

  fi

fi

sleep 5

done

:wq

**2.执行脚本**

sh /root/nginx_pid.sh &

nohup /bin/bash /root/nginx_pid.sh &