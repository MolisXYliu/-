# NGINX
## Nginx 的简介
1. 什么是 nginx
     - Nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的， 能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。
2. 正向代理
     1. 需要在客户端配置代理服务器进行指定网站访问
3. 反向代理
     - 暴露的是代理服务器地址，隐藏了真实服务器 IP 地址
4. 负载均衡
     - 增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡
5. 动静分离
   
## Nginx 的原理
1. mater 和 worker
2. worker 如何进行工作的(争抢机制)
3. 一个 master 和多个 woker 有好处
    1.  可以使用 nginx –s reload 热部署，利用 nginx 进行热部署操作
    2. 每个 woker 是独立的进程，如果有其中的一个 woker 出现问题，其他 woker 独立的，继续进行争抢，实现请求过程，不会造成服务中断
4. 设置多少个 woker 合适
     - **worker 数和服务器的 cpu 数相等是最为适宜的**
5. 连接数 worker_connection
     1. 第一个：发送请求，占用了 woker 的几个连接数？
         - 2 或者 4 个
     2.  nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的最大并发数是多少？
         - 普通的静态访问最大并发数是： worker_connections * worker_processes /2
         - 而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections *worker_processes/4。








## Nginx 安装
1. 安装 pcre 依赖
2. 安装 openssl 、 zlib 、 gcc 依赖
     - yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
3. 安装 nginx
    * 使用命令解压
    * ./configure
    * make && make install
4. 进入目录 /usr/local/nginx/sbin/nginx 启动服务


- 在 windows 系统中访问 linux 中 nginx，默认不能访问的，因为防火墙问题
    1. 关闭防火墙
    2. 开放访问的端口号， 80 端口

````java
查看开放的端口号
firewall-cmd --list-all
设置开放的端口号
firewall-cmd --add-service=http –permanent
firewall-cmd --add-port=80/tcp --permanent
重启防火墙
firewall-cmd –reload
````

## Nginx 的常用的命令
1. 进入 nginx 目录中
   cd /usr/local/nginx/sbin
2. 查看 nginx 版本号
   ./nginx -v
3. 、启动 nginx
    ./nginx
4. 停止 nginx
    ./nginx -s stop
5. 重新加载 nginx
    ./nginx -s reload

## Nginx 的配置文件
1. nginx 配置文件位置
     - cd /usr/local/nginx/conf/nginx.conf
2. 配置文件中的内容包含三部分内容
     1. 全局块：配置服务器整体运行的配置指令比如 worker_processes 1;处理并发数的配置
     2. events 块：影响 Nginx 服务器与用户的网络连接比如 worker_connections 1024; 支持的最大连接数为 1024
     3. http 块还包含两部分：http 全局块server 块

## Nginx 配置实例-反向代理实例 1
1. 实现效果
     - 打开浏览器，在浏览器地址栏输入地址 www.123.com，跳转到 liunx 系统 tomcat 主页面中
2. 准备工作
     1. 在 liunx 系统安装 tomcat， 使用默认端口 8080tomcat 安装文件放到 liunx 系统中，解压进入 tomcat 的 bin 目录中， ./startup.sh 启动 tomcat 服务器
     2. 对外开放访问的端口firewall-cmd --add-port=8080/tcp --permanent firewall-cmd –reload
     3. 在 windows 系统中通过浏览器访问 tomcat 服务器
3. 具体配置
     1. 第一步 在 windows 系统的 host 文件进行域名和 ip 对应关系的配置
     2. 第二步 在 nginx 进行请求转发的配置（反向代理配置）
          - server_name ;
          - proxy_pass http://127.0.0.1

## Nginx 配置实例-反向代理实例 2
1. 实现效果
     - 使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中nginx 监听端口为 9001，
     - 访问 http://192.168.17.129:9001/edu/ 直接跳转到 127.0.0.1:8080
     - 访问 http:// 192.168.17.129:9001/vod/ 直接跳转到 127.0.0.1:8081
2. 具体配置
     1. 找到 nginx 配置文件，进行反向代理配置
        - listen 9001
        - server_name ;
        - location ~/edu/ {proxy_pass http://127.0.0.1}
     2. 开放对外访问的端口号 9001 8080 8081

## Nginx 配置实例-负载均衡
1. 实现效果
     - 浏览器地址栏输入地址 http://192.168.17.129/edu/a.html，负载均衡效果，平均 8080和 8081 端口中
2. 在 nginx 的配置文件中进行负载均衡的配置
     - upstream myserver{server ip:8080;ip:8081;}
     - proxy_pass http://myserver;
3. nginx 分配服务器策略
1. 轮询（默认）
     - 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
2. weight
     - weight 代表权重默认为 1,权重越高被分配的客户端越多
3. ip_hash
    - 每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器
4. fair（第三方）
     - 按后端服务器的响应时间来分配请求，响应时间短的优先分配。

## Nginx 配置实例-动静分离
1. 什么是动静分离
     - 通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏
览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源
设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，
所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，
不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一
个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码 304，
如果有修改，则直接从服务器重新下载，返回状态码 200。
2. 具体配置
     1. 在 nginx 配置文件中进行配置
           - location /www/{root /data/;index index.html index.html:}


## Nginx 配置高可用的集群
1. 什么是 nginx 高可用
     1. 需要两台 nginx 服务器
     2. 需要 keepalived
     3. 需要虚拟 ip

2. 在两台服务器安装 keepalived
     1. 使用 yum 命令进行安装yum install keepalived –y
     2. 安装之后，在 etc 里面生成目录 keepalived，有文件 keepalived.conf
3. 完成高可用配置（主从配置）
     1. 修改/etc/keepalived/keepalivec.conf 配置文件 
````java
global_defs {
notification_email {
acassen@firewall.loc
failover@firewall.loc
sysadmin@firewall.loc
}
notification_email_from Alexandre.Cassen@firewall.loc
smtp_server 192.168.17.129
smtp_connect_timeout 30
router_id LVS_DEVEL
}
vrrp_script chk_http_port {
script "/usr/local/src/nginx_check.sh"
interval 2 #（检测脚本执行的间隔）
weight 2
}
vrrp_instance VI_1 {
state BACKUP # 备份服务器上将 MASTER 改为 BACKUP
interface ens33 //网卡
virtual_router_id 51 # 主、备机的 virtual_router_id 必须相同
priority 90 # 主、备机取不同的优先级，主机值较大，备份机值较小
advert_int 1
authentication {
auth_type PASS
auth_pass 1111
}
virtual_ipaddress {
192.168.17.50 // VRRP H 虚拟地址
}
}
````

     2. 在/usr/local/src 添加检测脚本
 ````java
 #!/bin/bash
 A=`ps -C nginx – no-header |wc -l`
 if [ $A -eq 0 ];then
 /usr/local/nginx/sbin/nginx
 sleep 2
 if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
 killall keepalived
 fi
 fi
````


     3. 把两台服务器上 nginx 和 keepalived 启动 
         - 启动 nginx： ./nginx 
         - 启动 keepalived： systemctl start keepalived.service

