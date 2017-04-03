针对nginx的优化我们可以从两方面入手： 
====


从运维的角度：   
====

```
从运维的角度来进行系统优化一般是我们对系统优化的首选方案。主要是由于风险小、成本低、见效快；

* 首选是内核配置的优化： 
 * 1. net.ipv4.tcp_syncookies = 1    防止syn flood攻击   
 * 2. net.core.somaxconn             已连接队列大小  
 * 3. net.ipv4.tcp_max_syn_backlog   半连接队列大小            
 * 4. net.ipv4.tcp_tw_recycle = 0    不开启tw状态的fd循环使用，线上环境 tw_recycle 不要打开    
 * 5. net.ipv4.tcp_tw_reuse = 1      开启tw状态的fd重复使用，tw_reuse 只对客户端起作用，开启后客户端在1s内回收    
 * 6. net.ipv4.tcp_timestamps = 0    关闭tcp时间戳   
 
 注意： 注意观察4、5、6三个配置；
    
* 然后是nginx配置的优化：   
 * 1. worker_processes    auto;   cpu个数自动适配；
 * 2. worker_cpu_affinity  auto;  cpu亲和性绑定、避免cpu cache失效、上下文切换等等；
 * 3. tcp_nodelay on;             减少数据延迟；
 * 4. tcp_nopush on;              在sendfile开启的时候生效，避免频繁net操作；
 * 5. pcre_jit on;                正则处理优化，使用jit；
 * 6. client_body_buffer_size     避免body写磁盘
 * 7. proxy_buffer_size           避免body写磁盘
 * 8. lua_code_cache on;          缓存lua 代码；
 * 9. [ssl_stapling](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling)、[ssl_session_tickets](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_tickets) 、[ssl_session_cache](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache)  
 * 10. postpone_output 0          减少数据延迟；
 * 11. sendfile on;
 * 12. reuseport                  [允许套接字端口共享](http://io.upyun.com/2015/07/20/nginx-socket-sharding/)

```


从开发的角度：
======

```  

从开发的角度进行优化，那就要找到自己系统的瓶颈在哪(cpu、mem、net、io、disk等)，cpu:找到最吃cpu的函数进行优化(高效算法替换、空间置换时间、硬件加速)、mem: 找到吃内存的地方（初始值是否合理、引用计算、结构体优化、使用共享内存）、net(同机房优先调用、大小包调整)、io(使用buffer cache异步刷盘、page cache设置)、disk(数据压缩存盘)；  

* 下面简单介绍一个列子： 
 * 1. 第一步火焰图绘制，可以参考这个[文章](https://moonbingbing.gitbooks.io/openresty-best-practices/content/flame_graph/install.html)，注意最终的火焰图的高度表示其函数的调用深度、其长度表示占用cpu的比例；    
 
 * 2. 根据火焰图找到相关瓶颈函数，然后在对症下药；


关于nginx lua中常遇到的问题：

* 1. 详细可见如下文章：
   * 1.  https://groups.google.com/forum/#!msg/openresty/x0EkaEj85ho/-ugGeYhrQ_EJ;context-place=searchin/openresty/nginx$20%E4%BC%98%E5%8C%96%7Csort:relevance       
   * 2.  https://groups.google.com/forum/#!msg/openresty/o67EjkImXyE/dASOW7x_argJ      
   * 3.  https://groups.google.com/forum/#!searchin/openresty/require$20resty.core/openresty/psPBSG4KcSU/Xp1QTx7tCAAJ     

主要涉及到：

 * 1.尽量让luajit进行编译更多的代码；
 * 2. --with-luajit-xcflags="-DLUAJIT_NUMMODE=2" 避免浮点数转化为字符串的昂贵代价；
 * 3.在init_by_lua阶段调用了require "resty.core"，尽量使用lua ffi接口避免lua Cfunction；
 
 ```   
 

Communite
====
 
在使用中有任何问题，欢迎反馈给我，可以用以下联系方式跟我交流

* 邮件(1031379296#qq.com, 把#换成@)
* QQ: 1031379296
* weibo: [@王发康](http://weibo.com/u/2786211992/home)




Thx
====

* chunshengsterATgmail.com



Author
====
* Linux\nginx\golang\c\c++爱好者
* 欢迎一起交流  一起学习# 
* Others say good and Others good
	


