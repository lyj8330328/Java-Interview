# 一、Nginx概述

nginx是一款自由的、开源的、高性能的HTTP服务器和反向代理服务器；同时也是一个IMAP、POP3、SMTP代理服务器；nginx可以作为一个HTTP服务器进行网站的发布处理，另外nginx可以作为反向代理进行负载均衡的实现。

这里主要通过三个方面简单介绍nginx

- 反向代理
- 负载均衡
- nginx特点

# 二、反向代理

关于代理

说到代理，首先我们要明确一个概念，所谓代理就是一个代表、一个渠道；

此时就设计到两个角色，一个是被代理角色，一个是目标角色，被代理角色通过这个代理访问目标角色完成一些任务的过程称为代理操作过程；

> **正向代理**

说反向代理之前，我们先看看正向代理，正向代理也是大家最常接触的到的代理模式，我们会从两个方面来说关于正向代理的处理模式，分别从软件方面和生活方面来解释一下什么叫正向代理

在如今的网络环境下，我们如果由于技术需要要去访问国外的某些网站，此时你会发现位于国外的某网站我们通过浏览器是没有办法访问的，此时大家可能都会用一个操作FQ进行访问，FQ的方式主要是找到一个可以访问国外网站的代理服务器，我们将请求发送给代理服务器，代理服务器去访问国外的网站，然后将访问到的数据传递给我们！

**上述这样的代理模式称为正向代理，正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息。**

> **反向代理**

明白了什么是正向代理，我们继续看关于反向代理的处理方式，举例如我大天朝的某宝网站，每天同时连接到网站的访问人数已经爆表，单个服务器远远不能满足人民日益增长的购买欲望了，此时就出现了一个大家耳熟能详的名词：分布式部署；也就是通过部署多台服务器来解决访问人数限制的问题；

**反向代理，主要用于服务器集群分布式部署的情况下，反向代理隐藏了服务器的信息！**

# 三、负载均衡

负载均衡在实际项目操作过程中，有硬件负载均衡和软件负载均衡两种，硬件负载均衡也称为硬负载，如F5负载均衡，相对造价昂贵成本较高，但是数据的稳定性安全性等等有非常好的保障，如中国移动中国联通这样的公司才会选择硬负载进行操作；更多的公司考虑到成本原因，会选择使用软件负载均衡，软件负载均衡是利用现有的技术结合主机硬件实现的一种消息队列分发机制


![](http://mycsdnblog.work/201919211614-Y.png)

nginx支持的负载均衡调度算法方式如下：

weight轮询（默认）：接收到的请求按照顺序逐一分配到不同的后端服务器，即使在使用过程中，某一台后端服务器宕机，nginx会自动将该服务器剔除出队列，请求受理情况不会受到任何影响。 这种方式下，可以给不同的后端服务器设置一个权重值（weight），用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的。

ip_hash：**每个请求按照发起客户端的ip的hash结果进行匹配**，这样的算法下一个固定ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题。

fair：**智能调整调度算法**，动态的根据后端服务器的请求处理到响应的时间进行均衡分配，响应时间短处理效率高的服务器分配到请求的概率高，响应时间长处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是nginx默认不支持fair算法，如果要使用这种调度算法，请安装upstream_fair模块

url_hash：**按照访问的url的hash结果分配请求**，每个请求的url会指向后端固定的某个服务器，可以在nginx作为静态服务器的情况下提高缓存效率。同样要注意nginx默认不支持这种调度算法，要使用的话需要安装nginx的hash软件包

# 四、Nginx配置

nginx是一个功能非常强大的web服务器加反向代理服务器，同时又是邮件服务器等等

在项目使用中，使用最多的三个核心功能是反向代理、负载均衡和静态服务器

这三个不同的功能的使用，都跟nginx的配置密切相关，nginx服务器的配置信息主要集中在nginx.conf这个配置文件中，并且所有的可配置选项大致分为以下几个部分

```nginx
main                                # 全局配置
 
events {                            # nginx工作模式配置
 
}
 
http {                                # http设置
    ....
 
    server {                        # 服务器主机配置
        ....
        location {                    # 路由配置
            ....
        }
 
        location path {
            ....
        }
 
        location otherpath {
            ....
        }
    }
 
    server {
        ....
 
        location {
            ....
        }
    }
 
    upstream name {                    # 负载均衡配置
        ....
    }
}
```

如上述配置文件所示，主要由6个部分组成：

1. main：用于进行nginx全局信息的配置
2. events：用于nginx工作模式的配置
3. http：用于进行http协议信息的一些配置
4. server：用于进行服务器访问信息的配置
5. location：用于进行访问路由的配置
6. upstream：用于进行负载均衡的配置

## 4.1 main模块

```nginx
# user nobody nobody;
worker_processes 2;
# error_log logs/error.log
# error_log logs/error.log notice
# error_log logs/error.log info
# pid logs/nginx.pid
worker_rlimit_nofile 1024;
```

上述配置都是存放在main全局配置模块中的配置项

- user用来指定nginx worker进程运行用户以及用户组，默认nobody账号运行
- worker_processes指定nginx要开启的子进程数量，运行过程中监控每个进程消耗内存(一般几M~几十M不等)根据实际情况进行调整，通常数量是CPU内核数量的整数倍
- error_log定义错误日志文件的位置及输出级别【debug / info / notice / warn / error / crit】
- pid用来指定进程id的存储文件的位置
- worker_rlimit_nofile用于指定一个进程可以打开最多文件数量的描述

## 4.2 event模块

```nginx
event {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}
```

上述配置是针对nginx服务器的工作模式的一些操作配置

worker_connections 指定最大可以同时接收的连接数量，这里一定要注意，最大连接数量是和worker processes共同决定的。

multi_accept 配置指定nginx在收到一个新连接通知后尽可能多的接受更多的连接

use epoll 配置指定了线程轮询的方法，如果是linux2.6+，使用epoll，如果是BSD如Mac请使用Kqueue

## 4.3 http模块

作为web服务器，http模块是nginx最核心的一个模块，配置项也是比较多的，项目中会设置到很多的实际业务场景，需要根据硬件信息进行适当的配置，常规情况下，使用默认配置即可！

```nginx
http {
    ##
    # 基础配置
    ##
 
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;
 
    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;
 
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
 
    ##
    # SSL证书配置
    ##
 
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;
 
    ##
    # 日志配置
    ##
 
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
 
    ##
    # Gzip 压缩配置
    ##
 
    gzip on;
    gzip_disable "msie6";
 
    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript
 text/xml application/xml application/xml+rss text/javascript;
 
    ##
    # 虚拟主机配置
    ##
 
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

```

### 4.3.1 基础配置

```nginx
sendfile on：配置on让sendfile发挥作用，将文件的回写过程交给数据缓冲去去完成，而不是放在应用中完成，这样的话在性能提升有有好处
tc_nopush on：让nginx在一个数据包中发送所有的头文件，而不是一个一个单独发
tcp_nodelay on：让nginx不要缓存数据，而是一段一段发送，如果数据的传输有实时性的要求的话可以配置它，发送完一小段数据就立刻能得到返回值，但是不要滥用哦
 
keepalive_timeout 10：给客户端分配连接超时时间，服务器会在这个时间过后关闭连接。一般设置时间较短，可以让nginx工作持续性更好
client_header_timeout 10：设置请求头的超时时间
client_body_timeout 10:设置请求体的超时时间
send_timeout 10：指定客户端响应超时时间，如果客户端两次操作间隔超过这个时间，服务器就会关闭这个链接
 
limit_conn_zone $binary_remote_addr zone=addr:5m ：设置用于保存各种key的共享内存的参数，
limit_conn addr 100: 给定的key设置最大连接数
 
server_tokens：虽然不会让nginx执行速度更快，但是可以在错误页面关闭nginx版本提示，对于网站安全性的提升有好处哦
include /etc/nginx/mime.types：指定在当前文件中包含另一个文件的指令
default_type application/octet-stream：指定默认处理的文件类型可以是二进制
type_hash_max_size 2048：混淆数据，影响三列冲突率，值越大消耗内存越多，散列key冲突率会降低，检索速度更快；值越小key，占用内存较少，冲突率越高，检索速度变慢
```

### 4.3.2 日志配置

```nginx
access_log logs/access.log：设置存储访问记录的日志
error_log logs/error.log：设置存储记录错误发生的日志
```

### 4.3.3 SSL证书加密

```nginx
ssl_protocols：指令用于启动特定的加密协议，nginx在1.1.13和1.0.12版本后默认是ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2，TLSv1.1与TLSv1.2要确保OpenSSL >= 1.0.1 ，SSLv3 现在还有很多地方在用但有不少被攻击的漏洞。
ssl prefer server ciphers：设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件
```

### 4.3.4 压缩配置

```nginx
gzip 是告诉nginx采用gzip压缩的形式发送数据。这将会减少我们发送的数据量。
gzip_disable 为指定的客户端禁用gzip功能。我们设置成IE6或者更低版本以使我们的方案能够广泛兼容。
gzip_static 告诉nginx在压缩资源之前，先查找是否有预先gzip处理过的资源。这要求你预先压缩你的文件（在这个例子中被注释掉了），从而允许你使用最高压缩比，这样nginx就不用再压缩这些文件了（想要更详尽的gzip_static的信息，请点击这里）。
gzip_proxied 允许或者禁止压缩基于请求和响应的响应流。我们设置为any，意味着将会压缩所有的请求。
gzip_min_length 设置对数据启用压缩的最少字节数。如果一个请求小于1000字节，我们最好不要压缩它，因为压缩这些小的数据会降低处理此请求的所有进程的速度。
gzip_comp_level 设置数据的压缩等级。这个等级可以是1-9之间的任意数值，9是最慢但是压缩比最大的。我们设置为4，这是一个比较折中的设置。
gzip_type 设置需要压缩的数据格式。上面例子中已经有一些了，你也可以再添加更多的格式。
```

### 4.3.5 文件缓存配置

```nginx
open_file_cache 打开缓存的同时也指定了缓存最大数目，以及缓存的时间。我们可以设置一个相对高的最大时间，这样我们可以在它们不活动超过20秒后清除掉。
open_file_cache_valid 在open_file_cache中指定检测正确信息的间隔时间。
open_file_cache_min_uses 定义了open_file_cache中指令参数不活动时间期间里最小的文件数。
open_file_cache_errors 指定了当搜索一个文件时是否缓存错误信息，也包括再次给配置中添加文件。我们也包括了服务器模块，这些是在不同文件中定义的。如果你的服务器模块不在这些位置，你就得修改这一行来指定正确的位置。
```

## 4.4 server模块

srever模块配置是http模块中的一个子模块，用来定义一个虚拟访问主机，也就是一个虚拟服务器的配置信息

```nginx
server {
    listen        80;
    server_name localhost    192.168.1.100;
    root        /nginx/www;
    index        index.php index.html index.html;
    charset        utf-8;
    access_log    logs/access.log;
    error_log    logs/error.log;
    ......
}
```

核心配置信息如下：

server：一个虚拟主机的配置，一个http中可以配置多个server

server_name：用力啊指定ip地址或者域名，多个配置之间用空格分隔

root：表示整个server虚拟主机内的根目录，所有当前主机中web项目的根目录

index：用户访问web网站时的全局首页

charset：用于设置www/路径中配置的网页的默认编码格式

access_log：用于指定该虚拟主机服务器中的访问记录日志存放路径

error_log：用于指定该虚拟主机服务器中访问错误日志的存放路径

## 4.5 location模块

location模块是nginx配置中出现最多的一个配置，主要用于配置路由访问信息

在路由访问信息配置中关联到反向代理、负载均衡等等各项功能，所以location模块也是一个非常重要的配置模块

基本配置

```nginx
location / {
    root    /nginx/www;
    index    index.php index.html index.htm;
}
```

location /：表示匹配访问根目录

root：用于指定访问根目录时，访问虚拟主机的web目录

index：在不指定访问具体资源时，默认展示的资源文件列表

反向代理配置方式

通过反向代理代理服务器访问模式，通过proxy_set配置让客户端访问透明化

```nginx
location / {
    proxy_pass http://localhost:8888;
    proxy_set_header X-real-ip $remote_addr;
    proxy_set_header Host $http_host;
}
```

## 4.6 upstream模块

upstream模块主要负责负载均衡的配置，通过默认的轮询调度方式来分发请求到后端服务器

简单的配置方式如下

```nginx
upstream name {
    ip_hash;
    server 192.168.1.100:8000;
    server 192.168.1.100:8001 down;
    server 192.168.1.100:8002 max_fails=3;
    server 192.168.1.100:8003 fail_timeout=20s;
    server 192.168.1.100:8004 max_fails=3 fail_timeout=20s;
}
```

核心配置信息如下

ip_hash：指定请求调度算法，默认是weight权重轮询调度，可以指定

server host:port：分发服务器的列表配置

- down：表示该主机暂停服务

- max_fails：表示失败最大次数，超过失败最大次数暂停服务

- fail_timeout：表示如果请求受理失败，暂停指定的时间之后重新发起请求

