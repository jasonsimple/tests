

### <center>目录</center>

<!-- toc -->

---

I.`MemCache`

===



session

`MemCache`是一个自由、源码开放、高性能、分布式的分布式内存对象缓存系统，用于动态Web应用以减轻数据库的负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高了网站访问的速度。 `MemCaChe`是一个存储键值对的`HashMap`，在内存中对任意的数据（比如字符串、对象等）所使用的`key-value`存储，数据可以来自数据库调用、`API`调用，或者页面渲染的结果。

`MemCache`设计理念就是小而强大，它简单的设计促进了快速部署、易于开发并解决面对大规模的数据缓存的许多难题，而所开放的`API`使得`MemCache`能用于`Java`、`C/C++/C#`、`Perl`、`Python`、`PHP`、`Ruby`等大部分流行的程序语言。

另外，说一下为什么会有`Memcache`和`memcached`两种名称？其实`Memcache`是这个项目的名称，而`memcached`是它服务器端的主程序文件名

`MemCache`的官方网站为 `http://memcached.org/`

i.`MemCache`访问模型

---



为了加深对`memcache`的理解，以`memcache`为代表的分布式缓存，访问模型如下：

 

<center><img src="http://i.imgur.com/YEbaurX.png"></center>

 

特别澄清一个问题，`MemCache`虽然被称为”分布式缓存”，但是`MemCache`本身<font color=red>完全不具备分布式的功能</font>，`MemCache`集群之间不会相互通信（与之形成对比的，比如JBoss Cache，某台服务器有缓存数据更新时，会通知集群中其他机器更新缓存或清除缓存数据），所谓的”**分布式**”，<font color=red>完全依赖于客户端程序的实现，就像上面这张图的流程一样</font>。

同时基于这张图，理一下`MemCache`一次写缓存的流程：

- 1、应用程序输入需要写缓存的数据

- 2、API将Key输入路由算法模块，路由算法根据`Key`和`MemCache`集群服务器列表得到一台服务器编号

- 3、由服务器编号得到`MemCache`及其的`ip地址`和`端口号`

- 4、`API`调用通信模块和指定编号的服务器通信，将数据写入该服务器，完成一次分布式缓存的写操作读缓存和写缓存一样，只要使用相同的路由算法和服务器列表，只要应用程序查询的是相同的`Key`，`MemCache`客户端总是访问相同的客户端去读取数据，只要服务器中还缓存着该数据，就能保证缓存命中。

这种`MemCache`集群的方式也是从分区容错性的方面考虑的，假如`Node2`宕机了，那么`Node2`上面存储的数据都不可用了，此时由于集群中`Node0`和`Node1`还存在，下一次请求`Node2`中存储的`Key`值的时候，肯定是没有命中的，这时先从数据库中拿到要缓存的数据，然后路由算法模块根据`Key`值在`Node0`和`Node1`中选取一个节点，把对应的数据放进去，这样下一次就又可以走缓存了，这种集群的做法很好，但是缺点是成本比较大。

ii.`Hash`算法 

---



从上面的图中，可以看出一个很重要的问题，就是对服务器集群的管理，路由算法至关重要，就和负载均衡算法一样，路由算法决定着究竟该访问集群中的哪台服务器，先看一个简单的**路由算法**。

### 1.余数`Hash`



简单的路由算法可以使用`余数Hash`: 用服务器数目和缓存数据`KEY`的`hash`值相除，余数为服务器列表下标编号，假如某个`str`对应的`HashCode`是`52`、服务器的数目是`3`，取余数得到`1`，`str`对应节点`Node1`，所以路由算法把`str`路由到`Node1`服务器上。由于`HashCode`随机性比较强，所以使用`余数Hash`路由算法就可以保证缓存数据在整个`MemCache`服务器集群中有比较均衡的分布。

如果不考虑服务器集群的伸缩性，那么`余数Hash`算法几乎可以满足绝大多数的缓存路由需求，但是当分布式缓存集群需要扩容的时候，就难办了。

就假设`MemCache`服务器集群由`3`台变为4`台`吧，更改服务器列表，仍然使用`余数Hash`，`52`对`4`的余数是`0`，对应`Node0`，但是`str`原来是存在`Node1`上的，这就导致了缓存没有命中。再举个例子，原来有`HashCode`为`0~19`的`20`个数据，

那么不妨举个例子，原来有`HashCode`为`0~19`的`20`个数据，那么：

<center><img src="http://i.imgur.com/XapXKxc.png"></center>

>现在扩容到`4`台，加粗标红的表示命中： 

 

<center><img src="http://i.imgur.com/64ZTBZZ.png"></center>

 

如果扩容到`20+`的台数，只有前三个`HashCode`对应的`Key`是命中的，也就是`15%`。当然现实情况肯定比这个复杂得多，不过足以说明，使用`余数Hash`的路由算法，在扩容的时候会造成大量的数据无法正确命中（其实不仅仅是无法命中，那些大量的无法命中的数据还在原缓存中在被移除前占据着内存）。在网站业务中，大部分的业务数据度操作请求上事实上是通过缓存获取的，只有少量读操作会访问数据库，因此数据库的负载能力是以有缓存为前提而设计的。当大部分被缓存了的数据因为服务器扩容而不能正确读取时，这些数据访问的压力就落在了数据库的身上，这将大大超过数据库的负载能力，严重的可能会导致数据库宕机。

>这个问题有解决方案，解决步骤为：

- （1）在网站访问量低谷，通常是深夜，技术团队加班，扩容、重启服务器

- （2）通过模拟请求的方式逐渐预热缓存，使缓存服务器中的数据重新分布

### 2.一致性`Hash`算法 



`一致性Hash算法`通过一个叫做`一致性Hash环的数据结构`实现`Key`到缓存服务器的`Hash`映射。简单地说，一致性哈希将整个哈希值空间组织成一个虚拟的圆环（这个环被称为`一致性Hash环`），如假设某空间哈希函数H的值空间是`0~2^32-1`（即哈希值是一个`32`位无符号整形），整个哈希空间如下：

<center><img src="http://i.imgur.com/xYsw0Jd.png"></center>

下一步将各个服务器使用H进行一个哈希计算，具体可以使用服务器的IP地址或者主机名作为关键字，这样每台机器能确定其在上面的哈希环上的位置了，并且是按照顺时针排列，这里我们假设三台节点`memcache`经计算后位置如下

 

 <center><img src="http://i.imgur.com/NcZji96.png"></center>

 

接下来使用相同算法计算出数据的哈希值h,并由此确定数据在此哈希环上的位置

假如我们有数据`A`、`B`、`C`、`D`、`4`个对象，经过哈希计算后位置如下：

<center><img src="http://i.imgur.com/D36Po3l.png"></center>

根据一致性哈希算法，数据A就被绑定到了`server01`上，`D`被绑定到了`server02`上，`B`、`C`在`server03`上，是按照顺时针找最近服务节点方法

这样得到的哈希环调度方法，有很高的容错性和可扩展性：

假设`server03`宕机

 

<center><img src="http://i.imgur.com/m4yfxw5.png"></center>

可以看到此时`C`、`B`会受到影响，将`B`、`C`节点被重定位到`Server01`。一般的，在一致性哈希算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即<font color=red>顺着逆时针方向行走遇到的第一台服务器</font>）之间数据，其它不会受到影响。

考虑另外一种情况，如果我们在系统中增加一台服务器`Memcached Server 04`：

 

 <center><img src="http://i.imgur.com/7lkQKvA.png"></center>

 

此时`A`、`D`、`C`不受影响，只有`B`需要重定位到新的`Server04`。一般的，在一致性哈希算法中，如果增加一台服务器，则受影响的数据仅仅是新服务器到其环空间中前一台服务器（即顺着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。

综上所述，一致性哈希算法对于节点的增减都只需重定位环空间中的一小部分数据，具有较好的容错性和可扩展性。

一致性哈希的**缺点**：

在服务节点太少时，容易因为节点分部不均匀而造成数据倾斜问题。我们可以采用增加虚拟节点的方式解决。

<font color=red>更重要</font>的是，集群中缓存服务器节点越多，增加/减少节点带来的影响越小，很好理解。换句话说，随着集群规模的增大，继续命中原有缓存数据的概率会越来越大，虽然仍然有小部分数据缓存在服务器中不能被读到，但是这个比例足够小，即使访问数据库，也不会对数据库造成致命的负载压力。

iii.`MemCache`实现原理

---



首先要说明一点，`MemCache`的数据存放在内存中

- 1、访问数据的速度比传统的关系型数据库要快，因为`Oracle`、`MySQL`这些传统的关系型数据库为了保持数据的持久性，数据存放在硬盘中，`IO`操作速度慢

- 2、`MemCache`的数据存放在内存中同时意味着只要`MemCache`重启了，数据就会消失

- 3、既然`MemCache`的数据存放在内存中，那么势必受到机器位数的限制，32位机器最多只能使用`2GB`的内存空间，`64`位机器可以认为没有上限

然后我们来看一下`MemCache`的原理，`MemCache`最重要的是内存如何分配的，`MemCache`采用的内存分配方式是固定空间分配，如下图所示：

<center><img src="http://i.imgur.com/taiKhah.png"></center>

这张图片里面涉及了`slab_class`、`slab`、`page`、`chunk`四个概念，它们之间的关系是：

- 1、`MemCache`将内存空间分为一组`slab`

- 2、每个`slab`下又有若干个`page`，每个`page`默认是`1M`，如果一个`slab`占用`100M`内存的话，那么这个`slab`下应该有`100`个`page`

- 3、每个`page`里面包含一组`chunk`，`chunk`是真正存放数据的地方，同一个`slab`里面的`chunk`的大小是固定的

- 4、有相同大小`chunk`的`slab`被组织在一起，称为`slab_class`

`MemCache`内存分配的方式称为`allocator`（分配运算），`slab`的数量是有限的，几个、十几个或者几十个，这个和启动参数的配置相关。

`MemCache`中的`value`存放的地方是由`value`的大小决定的，`value`总是会被存放到与`chunk`<font color=red>大小最接近的</font>一个`slab`中，比如

- `slab[1]`的`chunk`大小为`80`字节

- `slab[2]`的`chunk`大小为`100`字节

- `slab[3]`的`chunk`大小为`125`字节（相邻`slab`内的`chunk`基本以`1.25`为比例进行增长，`MemCache`启动时可以用`-f`指定这个比例）

那么过来一个`88`字节的`value`，这个`value`将被放到`2`号`slab`中。放`slab`的时候，首先`slab`要申请内存，申请内存是以`page`为单位的，所以在放入第一个数据的时候，无论大小为多少，都会有`1M`大小的`page`被分配给该`slab`。申请到`page`后，`slab`会将这个`page`的内存按`chunk`的大小进行切分，这样就变成了一个`chunk`数组，最后从这个`chunk`数组中选择一个用于存储数据。

如果这个`slab`中没有`chunk`可以分配了怎么办，如果`MemCache`启动没有追加`-M`（禁止`LRU`，这种情况下内存不够会报`Out Of Memory`错误），那么`MemCache`会把这个`slab`中<font color=red>最近最少使用的</font>`chunk`中的数据清理掉，然后放上最新的数据。

iv.`Memcache`的工作流程

---



 

 ![10](http://i.imgur.com/oruN6fL.png)

 

- 1、检查客户端的请求数据是否在`memcached`中，如果有，直接把请求数据返回，不再对数据库进行任何操作，路径操作为①②③⑦。

- 2、如果请求的数据不在`memcached`中，就去查数据库，把从数据库中获取的数据返回给客户端，同时把数据缓存一份到`memcached`中（`memcached`客户端不负责，需要程序明确实现），路径操作为①②④⑤⑦⑥。

- 3、每次更新数据库的同时更新`memcached`中的数据，保证一致性。

- 4、当分配给`memcached`内存空间用完之后，会使用`LRU`（`Least Recently Used`，最近最少使用）策略加上到期失效策略，失效数据首先被替换，然后再替换掉最近未使用的数据。

v.`Memcached`特征

---



>协议简单:

   它是基于文本行的协议，直接通过`telnet`在`memcached`服务器上可进行存取数据操作

注：文本行的协议：指的是信息以文本传送，一个信息单元传递完毕后要传送换行。比如对于`HTTP`的`GET`请求来说，`GET /index.html HTTP/1.1`是一行，接下去每个头部信息各占一行。一个空行表示整个请求结束

>基于libevent事件处理:

` Libevent`是一套利用`C`开发的程序库，它将`BSD`系统的`kqueue`,`Linux`系统的`epoll`等事件处理功能封装成一个接口，与传统的`select`相比，提高了性能。

 

>内置的内存管理方式:

所有数据都保存在内存中，存取数据比硬盘快，当内存满后，通过LRU算法自动删除不使用的缓存，但没有考虑数据的容灾问题，重启服务，所有数据会丢失。

    

>分布式

   各个`memcached`服务器之间互不通信，各自独立存取数据，不共享任何信息。服务器并不具有分布式功能，分布式部署取决于`memcache`客户端。

   

>Memcache的安装

分为两个过程：

- `memcache`服务器端的安装

- `memcached`客户端的安装。

所谓服务器端的安装就是在服务器（一般都是linux系统）上安装`Memcache`实现数据的存储。

所谓客户端的安装就是指`php`（或者其他程序，`Memcache`还有其他不错的`api`接口提供）去使用服务器端的`Memcache`提供的函数，需要`php`添加扩展。

`PHP`的`Memcache`

II.`centos7.2`+`nginx`+`php`+`memcache`+`mysql`

===



环境描述

---



>OS

```bash

[root@www ~]# cat /etc/redhat-release 

CentOS Linux release 7.2.1511 (Core)

```

>nginx和php

- `nginx-1.10.2.tar.gz`

- `php-5.6.27.tar.gz`

- `ip地址`：`192.168.31.141/24`

>memcache

- `memcached-1.4.33.tar.gz`

- `ip地址`：`192.168.31.250/24`

>mysql

- `mysql-5.7.13.tar.gz`

- `ip地址`：`192.168.31.225/24`

i.安装`nginx`(在`192.168.31.141`主机操作)

---



### 1.解压`zlib`



	[root@www ~]# tar zxf zlib-1.2.8.tar.gz

说明：不需要编译，只需要解压就行。

### 2.解压`pcre`



	[root@www ~]# tar zxf pcre-8.39.tar.gz

说明：不需要编译，只需要解压就行。

	[root@www ~]# yum -y install gcc gcc-c++ make libtool openssl openssl-devel

下载`nginx`的源码包：http://nginx.org/download

### 3.解压源码包



```bash

[root@www ~]# tar zxf nginx-1.10.2.tar.gz

[root@www ~]# cd nginx-1.10.2/

 [root@www ~]# groupadd www   #添加www组

[root@www ~]# useradd -g www www -s /sbin/nologin  #创建nginx运行账户www并加入到www组，不允许www用户直接登录系统

[root@www nginx-1.10.2]# ./configure --prefix=/usr/local/nginx1.10 --with-http_dav_module --with-http_stub_status_module --with-http_addition_module --with-http_sub_module  --with-http_flv_module --with-http_mp4_module --with-pcre=/root/pcre-8.39 --with-zlib=/root/zlib-1.2.8 --with-http_ssl_module --with-http_gzip_static_module --user=www --group=www

[root@www nginx-1.10.2]# make&& make install

```

注：

- `--with-pcre`：用来设置`pcre`的源码目录。

- ` --with-zlib`：用来设置`zlib`的源码目录。

 

 因为编译`nginx`需要用到这两个库的源码。

 

```bash

[root@www nginx-1.10.2]# ln -s /usr/local/nginx1.10/sbin/nginx /usr/local/sbin/

[root@www nginx-1.10.2]# nginx -t

```

### 4.启动`nginx`



```bash

[root@www nginx-1.10.2]# nginx

[root@www nginx-1.10.2]# netstat -anpt | grep nginx

tcp   0   0 0.0.0.0:80     0.0.0.0:*    LISTEN      9834/nginx: master

[root@www nginx-1.10.2]# firewall-cmd --permanent --add-port=80/tcp

success

[root@www nginx-1.10.2]# firewall-cmd --reload 

success

```

启动后可以再浏览器中打开页面，会显示`nginx`默认页面。

![11](http://i.imgur.com/MPbU2o2.png)

ii.安装php

---



### 1.安装`libmcrypt`



```bash

[root@www ~]# tar zxf libmcrypt-2.5.7.tar.gz 

[root@www ~]# cd libmcrypt-2.5.7/

[root@www libmcrypt-2.5.7]# ./configure --prefix=/usr/local/libmcrypt && make && make install

```

	[root@www ~]# yum -y install libxml2-devel libcurl-devel openssl-devel bzip2-devel

```bash

[root@www ~]# tar zxf php-5.6.27.tar.gz 

[root@www ~]# cd php-5.6.27/

[root@www php-5.6.27]#./configure --prefix=/usr/local/php5.6 --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-openssl --enable-fpm --enable-sockets --enable-sysvshm --enable-mbstring --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --with-mhash --with-mcrypt=/usr/local/libmcrypt --with-config-file-path=/etc --with-config-file-scan-dir=/etc/php.d --with-bz2 --enable-maintainer-zts

[root@www php-5.6.27]# make&& make install

```

```bash

[root@www php-5.6.27]# cp php.ini-production /etc/php.ini

修改/etc/php.ini文件，将short_open_tag修改为on，修改后的内容如下：

short_open_tag = On //支持php短标签

```

### 2.创建`php-fpm`服务启动脚本



```bash

[root@www php-5.6.27]# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm

[root@www php-5.6.27]# chmod +x /etc/init.d/php-fpm 

[root@www php-5.6.27]# chkconfig --add php-fpm

[root@www php-5.6.27]# chkconfig php-fpm on

```

提供`php-fpm`配置文件并编辑

	# cp /usr/local/php5.6/etc/php-fpm.conf.default /usr/local/php5.6/etc/php-fpm.conf

	[root@www php-5.6.27]# vi /usr/local/php5.6/etc/php-fpm.conf

修改内容如下

```bash

pid = run/php-fpm.pid

listen =127.0.0.1:9000

pm.max_children = 300

pm.start_servers = 10

pm.min_spare_servers = 10

pm.max_spare_servers =50

```

启动`php-fpm`服务

```bash

[root@phpserver ~]# service  php-fpm start

Starting php-fpm  done

[root@www php-5.6.27]# netstat -anpt | grep php-fpm

tcp   0   0 127.0.0.1:9000    0.0.0.0:*     LISTEN      10937/php-fpm: mast

```

iii.安装`mysql`（在`192.168.31.225`主机操作）

---



因为`centos7.2`默认安装了`mariadb-libs`，所以先要卸载掉

### 1.查看是否安装`mariadb`



	#rpm -qa | grep mariadb

卸载`mariadb`

	rpm -e --nodeps mariadb-libs

 

<center><img src="http://i.imgur.com/c7KLpjE.png"></center>

 

### 2.安装依赖包



注： 相关依赖包的作用

- `cmake`：由于从`MySQL5.5`版本开始弃用了常规的`configure`编译方法，所以需要`CMake`编译器，用于设置`mysql`的编译参数。如：安装目录、数据存放目录、字符编码、排序规则等。

- `Boost`  #从`MySQL 5.7.5`开始`Boost`库是必需的，`mysql`源码中用到了`C++`的`Boost`库，要求必须安装`boost1.59.0`或以上版本

- `GCC`是Linux下的`C语言`编译工具，`mysql`源码编译完全由`C`和`C++`编写，要求必须安装`GCC`

- `bison`：Linux下`C/C++`语法分析器

- `ncurses`：字符终端处理库

>1）安装文件准备

- 下载`cmake-3.5.tar.gz`    http://www.cmake.org/download/

- 下载`ncurses-5.9.tar.gz` ftp://ftp.gnu.org/gnu/ncurses/

- 下载`bison-3.0.4.tar.gz` http://ftp.gnu.org/gnu/bison/

- 下载`mysql-5.7.13.tar.gz`

wget http://cdn.mysql.com/Downloads/MySQL-5.7/mysql-5.7.13.tar.gz

- 下载`Boost_1_59_0.tar.gz`

wget http://nchc.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz

>2）安装`CMAKE`及必要的软件

安装`cmake`

 

 ![13](http://i.imgur.com/gmCwJEl.png)

 

![14](http://i.imgur.com/3vdA8gS.png)

`cmake –version`  ---查看`cmake`版本

![15](http://i.imgur.com/HkFEuj3.png)

安装`ncurses`

 

 ![16](http://i.imgur.com/nmeNxo4.png)

 

安装`bison`

 

安装`bootst`

	tar zxf  boost_1_59_0.tar.gz 

	mv boost_1_59_0 /usr/local/boost

>3）创建mysql用户和用户组及目录

	# groupadd -r mysql && useradd -r -g mysql -s /bin/false -M mysql---新建msyql组和msyql用户禁止登录shell

	# mkdir /usr/local/mysql        ---创建目录

	# mkdir /usr/local/mysql/data    ---数据库目录

### 3.编译安装`mysql`



解压`mysql`源码包

![18](http://i.imgur.com/wqmwGk4.png)

执行`cmake`命令进行编译前的配置

 

 ![19](http://i.imgur.com/dg7hV9W.png)

 

开始编译、编译安装：

![20](http://i.imgur.com/gjFlZw5.png)

注1：配置解释：

- `- DCMAKE_INSTALL_PREFIX=/usr/local/mysql`[`MySQL`安装的根目录]

- `- DMYSQL_DATADIR=/usr/local/mysql /data`[`MySQL`数据库文件存放目录]

- `- DSYSCONFDIR=/etc` [`MySQL`配置文件所在目录]

- `- DWITH_MYISAM_STORAGE_ENGINE=1` [添加`MYISAM`引擎支持]

- `- DWITH_INNOBASE_STORAGE_ENGINE=1`[添加`InnoDB`引擎支持]

- `- DWITH_ARCHIVE_STORAGE_ENGINE=1 ` [添加`ARCHIVE`引擎支持]

- `- DMYSQL_UNIX_ADDR=/usr/local/mysql /mysql.sock`[指定`mysql.sock`位置]

- `- DWITH_PARTITION_STORAGE_ENGINE=1`[安装支持数据库分区]

- `- DEXTRA_CHARSETS=all` [使`MySQL`支持所有的扩展字符]

- `- DDEFAULT_CHARSET=utf8`[设置`MySQL`的默认字符集为`utf8`]

- `- DDEFAULT_COLLATION=utf8_general_ci `[设置默认字符集校对规则]

- `- DWITH-SYSTEMD=1`  [可以使用`systemd`控制`mysql`服务]

- `- DWITH_BOOST=/usr/local/boost`  [指向`boost`库所在目录]

更多参数执行

	[root@localhost mysql-5.7.13]# cmake . –LH

注2：为了加快编译速度可以按下面的方式编译安装

 

![21](http://i.imgur.com/Jf4OgLV.png)

	make -j $(grep processor /proc/cpuinfo | wc –l)

`-j`参数表示根据`CPU`核数指定编译时的线程数，可以加快编译速度。默认为`1`个线程编译。

注3：若要重新运行`cmake`配置，需要删除`CMakeCache.txt`文件

	# make clean

	# rm -f CMakeCache.txt

优化`Mysql`的执行路径

 

![22](http://i.imgur.com/IBSuwef.png)

### 4.设置权限并初始化`MySQL`系统授权表



	# cd/usr/local/mysql

	# chown -R mysql:mysql  .       ---更改所有者,属组，注意是mysql .

	#bin/mysqld --initialize--user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data  

注1：以root初始化操作时要加`--user=mysql`参数，生成一个随机密码（注意保存登录时用）

注2：`MySQL 5.7.6`之前的版本执行这个脚本初始化系统数据库

	/usr/local/mysql/bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

`5.7.6`之后版本初始系统数据库脚本（本文使用此方式初始化）

	#/usr/local/mysql/bin/mysqld --initialize-insecure--user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

 

![23](http://i.imgur.com/z2erI5t.png)

注意：如果使用`–initialize`参数初始化系统数据库之后，会生成`root`用户的一个临时密码，如上图高亮中所示。

	# chown -Rmysql:mysql .       ---改所有者，注意是root .

### 5.创建配置文件



	# cd/usr/local/mysql/support-files     ---进入MySQL安装目录支持文件目录

	# cp my-default.cnf /etc/my.cnf    ---复制模板为新的配置文件，

 

 ![24](http://i.imgur.com/dPCO4vt.png)

 

修改文件中配置选项，如下图所示，添加如下配置项

	#vi  /etc/my.cnf

![25](http://i.imgur.com/AMBQTE7.png)

### 6.配置`mysql`自动启动

 

 

 

![26](http://i.imgur.com/VLT4fiG.png)

![27](http://i.imgur.com/WLxOo89.png)

服务启动失败，查看错误日志文件

 

![28](http://i.imgur.com/ezVnQSi.png)

在`mysqld.service`，把默认的`pid`文件指定到了`/var/run/mysqld/`目录，而并没有事先建立该目录，因此要手动建立该目录并把权限赋给`mysql`用户。

 

 ![29](http://i.imgur.com/ky9NHld.png)

 

或者修改`/usr/lib/system/system/mysqld.service`，修改内容如下：

![30](http://i.imgur.com/2Xxz3rc.png)

	#systemctl  daemon-reload

再次启动`mysql`服务

 

 ![31](http://i.imgur.com/keURMxO.png)

查看端口号

 

![32](http://i.imgur.com/tJtnv2g.png)

服务启动成功

访问`MySQL`数据库

	# mysql -u root -h 127.0.0.1 -p     ---连接mysql，输入初始化时生成的随机密码

 

 ![33](http://i.imgur.com/YVWVZI2.png)

设置数据库管理员用户`root`的密码

 

![34](http://i.imgur.com/vBiQ6ZG.png)

iv.安装`memcached`服务端（在`192.168.31.250`主机操作）

---



`memcached`是基于`libevent`的事件处理。`libevent`是个程序库，它将`Linux`的`epoll`、`BSD`类操作系统的`kqueue`等事件处理功能封装成统一的接口。即使对服务器的连接数增加，也能发挥`I/0`的性能。 `memcached`使用这个`libevent`库，因此能在`Linux`、`BSD`、`Solaris`等操作系统上发挥其高性能。

### 1.首先先安装`memcached`依赖库`libevent`



```bash

[root@memcache ~]# tar zxf libevent-2.0.22-stable.tar.gz 

[root@memcache ~]# cd libevent-2.0.22-stable/

[root@memcache libevent-2.0.22-stable]# ./configure

[root@memcache libevent-2.0.22-stable]# make&& make install

```

编译安装的时候没有指定路径，那么默认路径就是`/usr/local`

### 2.安装`memcached`



```bash

[root@memcache ~]# tar zxf memcached-1.4.33.tar.gz 

[root@memcache ~]# cd memcached-1.4.33/

[root@memcache memcached-1.4.33]# ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local

[root@memcache memcached-1.4.33]# make&& make install

```

`libevent`默认的安装路径是在`/usr/local`，所以指定此路径

### 3.检测是否成功安装



```bash

[root@memcache ~]# ls /usr/local/memcached/bin/memcached 

/usr/local/memcached/bin/memcached

```

通过以上操作就很简单的把`memcached`服务端编译好了。这时候就可以打开服务端进行工作了。

配置环境变量:

进入用户宿主目录，编辑`.bash_profile`，为系统环境变量`LD_LIBRARY_PATH`增加新的目录，需要增加的内容如下：

```bash

[root@memcache ~]# vi ~/.bash_profile

MEMCACHED_HOME=/usr/local/memcached

LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MEMCACHED_HOME/lib

[root@memcache ~]# /usr/local/memcached/bin/memcached -d -m 2048 -l 192.168.31.250 -p 11211 -u root -c 10240 -P /usr/local/memcached /memcached.pid

```

>启动参数说明

|参数|作用|

|:-----|:------|

|`-d`　|　选项是启动一个守护进程。|

|`-m`　|　分配给`Memcache`使用的内存数量，单位是`MB`，默认`64MB`。|

|`-l　`|　监听的`IP地址`。（默认：`INADDR_ANY`，所有地址）|

|`-p`　　|设置`Memcache`的`TCP`监听的端口，最好是`1024`以上的端口。|

|`-u`　　|运行`Memcache`的用户，如果当前为`root`的话，需要使用此参数指定用户。|

|`-c`　|　选项是最大运行的并发连接数，默认是`1024`。|

|`-P`　　|设置保存`Memcache`的`pid`文件。|

|`-M`| 内存耗尽时返回错误，而不是删除项|

|`-f `|块大小增长因子，默认是`1.25`|

|`-n` |最小分配空间，`key`+`value`+`flags`默认是`48`|

|`-h` |显示帮助|

```bash

[root@memcache ~]# netstat -anpt |grep memcached

tcp   0  0 192.168.31.250:11211    0.0.0.0:*  LISTEN      12840/memcached 

```

### 4.设置防火墙



```bash

[root@memcache ~]# firewall-cmd --permanent --add-port=11211/tcp

success

[root@memcache ~]# firewall-cmd --reload 

success

```

### 5.刷新用户环境变量



		[root@memcache ~]# source ~/.bash_profile

### 6.编写`memcached`服务启停脚本



	[root@memcache ~]# vi /etc/init.d/memcached

>脚本内容如下

 

```bash

[root@memcache ~]# cat /etc/init.d/memcached 

#!/bin/sh

#

# pidfile: /usr/local/memcached/memcached.pid

# memcached_home: /usr/local/memcached

# chkconfig: 35 21 79

# description: Start and stop memcached Service

# Source function library

. /etc/rc.d/init.d/functions

RETVAL=0

prog="memcached"

basedir=/usr/local/memcached

cmd=${basedir}/bin/memcached

pidfile="$basedir/${prog}.pid"

#interface to listen on (default: INADDR_ANY, all addresses)

ipaddr="192.168.31.250"

#listen port

port=11211

#username for memcached

username="root"

#max memory for memcached,default is 64M

max_memory=2048

#max connections for memcached

max_simul_conn=10240

start() {

echo -n $"Starting service: $prog"

$cmd -d -m $max_memory -u $username -l $ipaddr -p $port -c $max_simul_conn -P $pidfile

RETVAL=$?

echo

[ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog

}

stop() {

echo -n $"Stopping service: $prog  "

run_user=$(whoami)

pidlist=$(ps -ef | grep $run_user | grep memcached | grep -v grep | awk '{print($2)}')

for pid in $pidlist

do

kill -9 $pid

if [ $? -ne 0 ]; then

return 1

fi

done

RETVAL=$?

echo

[ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog

}

# See how we were called.

case "$1" in

start)

start

;;

stop)

stop

;;

restart)

stop

start

;;

*)

echo "Usage: $0 {start|stop|restart|status}"

exit 1

esac

exit $RETVAL

```

>设置脚本可被执行

```bash

[root@memcache ~]# chmod +x /etc/init.d/memcached 

[root@memcache ~]# chkconfig --add memcached

[root@memcache ~]# chkconfig memcached on

```

说明：

>`shell`脚本中`return`的作用

- 1) 终止一个函数. 

- 2) `return`命令允许带一个整型参数, 这个整数将作为函数的"退出状态码"返回给调用这个函数的脚本, 并且这个整数也被赋值给变量`$?`.

- 3) 命令格式：`return value`

### 7.配置`nginx.conf`文件（在`nginx`主机操作）



配置内容如下：

```bash

user www www;

worker_processes  4;

worker_cpu_affinity 0001 0010 0100 1000;

error_log  logs/error.log;

#error_log  logs/error.log  notice;

#error_log  logs/error.log  info;

pid        logs/nginx.pid;

events {

    use epoll;

    worker_connections  65535;

    multi_accept on;

}

http {

    include       mime.types;

    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '

    #                  '$status $body_bytes_sent "$http_referer" '

    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;

    tcp_nopush     on;

    keepalive_timeout  65;

    tcp_nodelay on;

    client_header_buffer_size 4k;

    open_file_cache max=102400 inactive=20s;

    open_file_cache_valid 30s;

    open_file_cache_min_uses 1;

    client_header_timeout 15;

    client_body_timeout 15;

    reset_timedout_connection on;

    send_timeout 15;

    server_tokens off;

    client_max_body_size 10m;

    fastcgi_connect_timeout     600;

    fastcgi_send_timeout 600;

    fastcgi_read_timeout 600;

    fastcgi_buffer_size 64k;

    fastcgi_buffers     4 64k;

    fastcgi_busy_buffers_size 128k;

    fastcgi_temp_file_write_size 128k;

    fastcgi_temp_path /usr/local/nginx1.10/nginx_tmp;

    fastcgi_intercept_errors on;

    fastcgi_cache_path /usr/local/nginx1.10/fastcgi_cache levels=1:2       

    keys_zone=cache_fastcgi:128m inactive=1d max_size=10g;

    gzip on;

    gzip_min_length  2k;

    gzip_buffers     4 32k;

    gzip_http_version 1.1;

    gzip_comp_level 6;

    gzip_types  text/plain text/css text/javascript application/json 

    application/javascript application/x-javascript application/xml;

    gzip_vary on;

    gzip_proxied any;

server {

    listen       80;

    server_name  www.benet.com;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

location ~* ^.+\.(jpg|gif|png|swf|flv|wma|wmv|asf|mp3|mmf|zip|rar)$ {

valid_referers none blocked  www.benet.com benet.com;

if ($invalid_referer) {

    #return 302  http://www.benet.com/img/nolink.jpg;

    return 404;

    break;

  }

    access_log off;

}

location / {

    root   html;

    index  index.php index.html index.htm;

        }

location ~* \.(ico|jpe?g|gif|png|bmp|swf|flv)$ {

    expires 30d;

    #log_not_found off;

    access_log off;

        }

location ~* \.(js|css)$ {

    expires 7d;

    log_not_found off;

    access_log off;

        }      

location = /(favicon.ico|roboots.txt) {

    access_log off;

    log_not_found off;

        }

location /status {

    stub_status on;

        }

location ~ .*\.(php|php5)?$ {

            root html;

            fastcgi_pass 127.0.0.1:9000;

            fastcgi_index index.php;

            include fastcgi.conf;

            fastcgi_cache cache_fastcgi;

            fastcgi_cache_valid 200 302 1h;

            fastcgi_cache_valid 301 1d;

            fastcgi_cache_valid any 1m;

            fastcgi_cache_min_uses 1;

            fastcgi_cache_use_stale error timeout invalid_header http_500;

            fastcgi_cache_key http://$host$request_uri;

        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html

        #

        error_page   500 502 503 504  /50x.html;

location = /50x.html {

    root   html;

        }

   }

}

```

重启`nginx`服务

生成一个`php`测试页

```bash

[root@www memcache-3.0.8]# cat /usr/local/nginx1.10/html/test1.php 

<?php

phpinfo();

?>

```

使用浏览器访问`test1.php`测试页

 

![35](http://i.imgur.com/vXcQ3h0.png)

v.`memcache`客户端（在`php`服务器操作）:

---



`memcache`分为服务端和客户端。服务端用来存放缓存，客户端用来操作缓存。

安装php扩展库（`phpmemcache`）。

### 1.安装`PHP`  `Memcache`扩展



可以使用`php`自带的`pecl`安装程序

	# /usr/local/servers/php/bin/pecl install memcache

也可以从源码安装,他是生成`php`的扩展库文件`memcache.so`。

### 2.安装`memcache`扩展库



```bash

[root@www ~]# tar zxf memcache-3.0.8.tgz 

[root@www ~]# cd memcache-3.0.8/

[root@www memcache-3.0.8]# /usr/local/php5.6/bin/phpize

[root@wwwmemcache-3.0.8]#./configure --enable-memcache --with-php-config=/usr/local/php5.6/bin/php-config

[root@www memcache-3.0.8]# make&& make install

```

安装完后会有类似这样的提示：

	Installing shared extensions:     /usr/local/php5.6/lib/php/extensions/no-debug-zts-20131226/

把这个记住，然后修改`php.ini`

添加一行

	extension=/usr/local/php5.6/lib/php/extensions/no-debug-zts-20131226/memcache.so

>重启`php-fpm`服务

```bash

[root@www memcache-3.0.8]# service  php-fpm restart

Gracefully shutting down php-fpm .done

Starting php-fpm  done

```

测试：

>检查`php`扩展是否正确安装

1、	`[root@www html]# /usr/local/php5.6/bin/php -m`

命令行执行`php -m` 查询结果中是否有`memcache`项

2、创建`phpinfo()`页面，查询`session`项下面的`Registered save handlers`值中是否有`memcache`项

>浏览器访问`test1.php`

 

 

![36](http://i.imgur.com/bcnvFf2.png)

![37](http://i.imgur.com/XS6gv83.png)

>测试代码

```bash

[root@www ~]# cat /usr/local/nginx1.10/html/test2.php 

<?php

$memcache = new Memcache;

$memcache->connect('192.168.31.250', 11211) or die ("Could not connect");

$version = $memcache->getVersion();

echo "Server's version: ".$version."<br/>";

$tmp_object = new stdClass;

$tmp_object->str_attr = 'test';

$tmp_object->int_attr = 123;

$memcache->set('key', $tmp_object, false, 10) or die ("Failed to save data at the server");

echo "Store data in the cache (data will expire in 10 seconds)<br/>";

$get_result = $memcache->get('key');

echo "Data from the cache:<br/>";

var_dump($get_result);

?>

```

>浏览器访问`test2.php`

 

![38](http://i.imgur.com/VolnR2d.png)

使用`memcache`实现`session`共享

配置`php.ini`中的`Session`为`memcache`方式。

	session.save_handler = memcache

		session.save_path = "tcp://192.168.31.250:11211?persistent=1&weight=1&timeout=1&retry_interval=15"

注：

- `session.save_handler`：设置`session`的储存方式为`memcache` 。默认以文件方式存取session数据，如果想要使用自定义的处理来存取session数据，比如memcache方式则修为`session.save_handler` = `memcache`

- `session.save_path`：设置`session`储存的位置，多台`memcache`用逗号隔开

使用多个 `memcached server` 时用逗号`,`隔开，可以带额外的参数`persistent`、`weight`、`timeout`、`retry_interval`等等，

类似这样的：`tcp://host:port?persistent=1&weight=2,tcp://host2:port2`。

>memcache实现session共享也可以在某个一个应用中设置： 

`ini_set`("session.save_handler", "memcache"); 

`ini_set`("session.save_path", "tcp://192.168.0.9:11211"); 

`ini_set()`只对当前php页面有效，并且不会去修改`php.ini`文件本身，也不会影响其他php页面。

### 3.测试`memcache`可用性



>在web服务器上新建`/usr/local/nginx1.10/html/memcache.php`文件。

内容如

```bash

<?php

session_start();

if (!isset($_SESSION['session_time']))

{

 $_SESSION['session_time'] = time();

}

echo "session_time:".$_SESSION['session_time']."<br />";

echo "now_time:".time()."<br />"; 

echo "session_id:".session_id()."<br />";

?>

```

访问网址`http://192.168.31.141/memcache.php`可以查看`session_time`是否都是为`memcache`中的`Session`，同时可以在不同的服务器上修改不同的标识查看是否为不同的服务器上的。

 

<center><img src="http://i.imgur.com/OopJsyC.png"></center>

>可以直接用`sessionid` 去` memcached` 里查询一下

```bash

[root@www html]# telnet 192.168.31.250 11211

Trying 192.168.31.250...

Connected to 192.168.31.250.

Escape character is '^]'.

get ffaqe5b1oar311n3cn5q9co5g6

VALUE ffaqe5b1oar311n3cn5q9co5g6 0 26

session_time|i:1479134997;

```

得到`session_time|i`:`1479134997`;这样的结果，说明`session` 正常工作 

默认`memcache`会监听`11221`端口，如果想清空服务器上`memecache`的缓存，一般使用的是：

```bash

[root@memcache ~]# telnet 192.168.31.250 11211

Trying 192.168.31.250...

Connected to 192.168.31.250.

Escape character is '^]'.

flush_all

OK

```

同样也可以使用

```bash

[root@memcache ~]# echo "flush_all" | nc 192.168.31.250 11211

OK

```

使用`flush_all` 后并不是删除`memcache`上的`key`，而是置为过期`memcache`安全配置

因为`memcache`不进行权限控制，因此需要通过`iptables`将`memcache`仅开放个`web`服务器。

vi.测试`memcache`缓存数据库数据

---



### 1.在`Mysql`服务器上创建测试表



```bash

mysql> create database testdb1;

Query OK, 1 row affected (0.00 sec)

mysql> use testdb1;

Database changed

mysql> create table test1(id int not null auto_increment,name varchar(20) default null,primary key (id)) engine=innodb auto_increment=1 default charset=utf8;

Query OK, 0 rows affected (0.03 sec)

mysql> insert into test1(name) values ('tom1'),('tom2'),('tom3'),('tom4'),('tom5');

Query OK, 5 rows affected (0.01 sec)

Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from test1;

+----+------+

| id | name |

+----+------+

|  1 | tom1 |

|  2 | tom2 |

|  3 | tom3 |

|  4 | tom4 |

|  5 | tom5 |

+----+------+

5 rows in set (0.00 sec)

```

测试

下面就是测试的工作了，这里有个`php`脚本，用于测试`memcache`是否缓存数据成功

需要为这个脚本添加一个只读的数据库用户，命令格式

```bash

mysql> grant select on testdb1.* to user@'%' identified by '123456';

Query OK, 0 rows affected, 1 warning (0.00 sec)

```

### 2.在web服务器上创建测试脚本内容如下



```bash

[root@www html]# cat /usr/local/nginx1.10/html/test_db.php 

<?php

$memcachehost = '192.168.31.250';

$memcacheport = 11211;

$memcachelife = 60;

$memcache = new Memcache;

$memcache->connect($memcachehost,$memcacheport) or die ("Could not connect");

$query="select * from test1 limit 10";

$key=md5($query);

if(!$memcache->get($key))

{

                $conn=mysql_connect("192.168.31.225","user","123456");

                mysql_select_db(testdb1);

                $result=mysql_query($query);

while ($row=mysql_fetch_assoc($result))

                {

                        $arr[]=$row;

                }

                $f = 'mysql';

                $memcache->add($key,serialize($arr),0,30);

                $data = $arr ;

}

else{

        $f = 'memcache';

        $data_mem=$memcache->get($key);

        $data = unserialize($data_mem);

}

echo $f;

echo "<br>";

echo "$key";

echo "<br>";

//print_r($data);

foreach($data as $a)

{

echo "number is <b><font color=#FF0000>$a[id]</font></b>";

echo "<br>";

echo "name is <b><font color=#FF0000>$a[name]</font></b>";

echo "<br>";

}

?>

```

### 3.访问页面测试

 

 

<center><img src="http://i.imgur.com/pDqk9tU.png"></center>

如果出现`mysql`表示`memcached`中没有内容，需要`memcached`从数据库中取得

再刷新页面，如果有`memcache`标志表示这次的数据是从`memcached`中取得的。

`memcached`有个缓存时间默认是1分钟，过了一分钟后，`memcached`需要重新从数据库中取得数据

 

<center><img src="http://i.imgur.com/8YVNlVC.png"></center>

### 4.查看 `Memcached `缓存情况



>需要使用` telnet `命令查看

```bash

[root@memcache ~]# telnet 192.168.31.250 11211

Trying 192.168.31.250...

Connected to 192.168.31.250.

Escape character is '^]'.

stats

STAT pid 1681                    //Memcached 进程的ID

STAT uptime 8429                //进程运行时间

STAT time 1479142306             //当前时间

STAT version 1.4.33                // Memcached 版本

STAT libevent 2.0.22-stable

STAT pointer_size 64

STAT rusage_user 1.218430

STAT rusage_system 1.449512

STAT curr_connections 5

STAT total_connections 32

STAT connection_structures 10

STAT reserved_fds 20

STAT cmd_get 25//总共获取数据的次数（等于 get_hits + get_misses ）

STAT cmd_set 19 //总共设置数据的次数

STAT cmd_flush 4

STAT cmd_touch 0

STAT get_hits 15//命中了多少次数据，也就是从 Memcached 缓存中成功获取数据的次数

STAT get_misses 10//没有命中的次数

STAT get_expired 3

STAT get_flushed 1

STAT delete_misses 0

STAT delete_hits 0

STAT incr_misses 2

STAT incr_hits 2

STAT decr_misses 0

STAT decr_hits 0

STAT cas_misses 0

STAT cas_hits 0

STAT cas_badval 0

STAT touch_hits 0

STAT touch_misses 0

STAT auth_cmds 0

STAT auth_errors 0

STAT bytes_read 3370

STAT bytes_written 15710

STAT limit_maxbytes 2147483648//总的存储大小，默认为 64M

STAT accepting_conns 1

STAT listen_disabled_num 0

STAT time_in_listen_disabled_us 0

STAT threads 4

STAT conn_yields 0

STAT hash_power_level 16

STAT hash_bytes 524288

STAT hash_is_expanding 0

STAT malloc_fails 0

STAT log_worker_dropped 0

STAT log_worker_written 0

STAT log_watcher_skipped 0

STAT log_watcher_sent 0

STAT bytes 584//当前所用存储大小

STAT curr_items 3

STAT total_items 17

STAT expired_unfetched 2

STAT evicted_unfetched 0

STAT evictions 0

STAT reclaimed 4

STAT crawler_reclaimed 0

STAT crawler_items_checked 0

STAT lrutail_reflocked 0

END

```

命中率= `get_hits/ cmd_get`



---

