Remote access VPN

VPN server 端

**第一阶段**

1.  **策略**

-   **Pre-share (不是单纯的key，而是group+key)**

-   **DH GROUP 2**

1.  **AM（3个包）密码的安全性由HASH来保障**

**第1.5阶段**

1.  **xauth（1.安全2.用户认证3.AAA）**

2.  **mode-config（推送策略1.ip 2.DNS 3.tunnel split 4.dns split等等）**

**第2阶段QM （策略）**

**Remote access vpn的策略怎么样构建？**

**Remote access vpn第一阶段的认证策略我们使用pre-share
key，当然我们可以使用证书来认证。远程访问VPN第一阶段内建的策略都是DH
GROUP 2的，group 1是不行的。因为客户端内建的策略都是group
2的，所以这一点一定要注意。在站点到站点的VPN当中，如果我们选择了预共享密钥方式来验证的时候，我们只需敲上一个crypto
isakmp
key，然后写上对方的IP地址就可以了，通过一个单纯的KEY来验证对方。但是远程访问VPN中并没有通过一个key来验证对方，而是通过一个group+key，两者的组合作为一个预共享密钥来验证的；为什么要有这样一个设计呢？因为在实际的应用环境会存在很多的部门，我们可以为每个部门创建一个GROUP，为每一个group创建一个key,我们可以基于group的不同，部门的不同推送不同的策略，做不同的认证方法。这就是远程访问VPN第一阶段的认证不使用单纯的key，而要使用group+key的原因。所以我们可以使用组的方法，把用户分开，基于不同的组做相应的策略。**

远程访问vpn的预共享密钥认证方法的第一阶段采用的是积极模式（AM），只有3个包，而站点到站点的VPN第一阶段采用的是主模式（MM）共6个包，主模式的6个包中的第5，6个包是在加密的环境下进行的密钥的交换，但是积极模式只有3个包，密钥并没有采用加密方式，当然肯定不是明文传送的，密钥的安全性是有HASH来保障的。其实就是将密钥通过HASH做相应扰乱发送给对方的。当然这样的方式是不够安全的，因为他的安全性不够，所以提供了增强型的认证--1.5阶段，第一阶段的group+key认证完了之后，在做一个Xauth，&lt;做一个用户名和密码的认证&gt;。

**做一次增强型的认证有以下好处：首先是会比以前更安全，因为1.5阶段是加密的，其次提供了一个用户的认证。再者，Xauth引入了AAA机制，我们可以通过AAA服务器而不是用本地来做用户名和密码的认证，这是xauth的作用。**

**在1.5阶段，还有一个重要的东西就是mode
confige，它的作用就是为我们的客户推送策略，比如为客户端推送一个内网IP地址，这个是必须推送的策略，如果内网有dns服务器，同样可以推送DNS地址给我的客户，还有tunnel的分割技术。**

**最后就是QM阶段，第二阶段，跟site-to-site的QM是一样的，三个包的交互。**

**下面我们来看一下远程访问VPN的配置过程：**

**1**<img src="http://funkyimg.com/i/2kd8p.png" />

GW(config)\#interface f0/0

GW(config-if)\#ip add 10.0.0.1 255.255.255.0

GW(config-if)\#no shutdown

GW(config-if)\#exit

GW(config)\#int f1/0

GW(config-if)\#ip add 192.168.1.1 255.255.255.0

GW(config-if)\#no shutdown

GW(config-if)\#exit

GW(config)\#ip route 0.0.0.0 0.0.0.0 10.0.0.2

**第一阶段策略**

GW(config)\#crypto isakmp policy 1

GW(config-isakmp)\#encryption 3des

GW(config-isakmp)\#group 2

GW(config-isakmp)\#hash md5

GW(config-isakmp)\#authentication pre-share

GW(config-isakmp)\#exit

**第一阶段group+key认证
此阶段采用HASH算法保障key的安全，显然安全性不够。**

GW(config)\#crypto isakmp client configuration group vpngroup

GW(config-isakmp-group)\#key cisco

GW(config-isakmp-group)\#exit

**因为第一阶段group+key的认证不够安全，所以引入xauth增强型认证，来认证用户名，密码。xauth特性需要引入AAA模式，这个阶段称之为1.5阶段，在1.5阶段的认证方式采用加密方法**

**启用AAA服务**

GW(config)\#aaa new-model

**做xauth的认证策略名字叫benet，客户端从本地（local&lt;GW&gt;）来拿用户名和密码**

GW(config)\#aaa authentication login benet local

**GW端指定的用户名为cisco 密码是ccnp**

GW(config)\#username cisco password ccnp

**xauth认证成功后为客户端分配的IP地址池，地址池的名字是vpnpool
,地址段是123.1.1.100到123.1.1.200**

GW(config)\#ip local pool vpnpool 123.1.1.100 123.1.1.200

**策略的授权需要在组中进行，如地址池，隧道分割，dns分割等等。**

GW(config)\#crypto isakmp client configuration group vpngroup

GW(config-isakmp-group)\#pool vpnpool

**通过网络访问方式在路由器本地实现策略的授权,策略的名字accp**

GW(config)\#aaa authorization network accp local

**第二阶段，定义如何保护感兴趣流，创建传输集**

GW(config)\#crypto ipsec transform-set benetset esp-3des esp-md5-hmac

GW(cfg-crypto-trans)\#exit

**因为远程访问VPN无法确定对端的ip地址，也就无从知道相应的感兴趣流，所以要创建动态MAP,在动态map中调用传输集**

GW(config)\#crypto dynamic-map dymap 1

GW(config-crypto-map)\#set transform-set benetset

GW(config-crypto-map)\#reverse-route //向网关注入反向路由。

GW(config-crypto-map)\#exit

**创建静态map,在静态MAP中调用动态map,**

GW(config)\#crypto map vpnmap 1 ipsec-isakmp dynamic dymap

**在静态MAP中匹配认证策略，授权策略，向VPN客户端推送IP地址的方法是respond**

GW(config)\#crypto map vpnmap client authentication list benet

GW(config)\#crypto map vpnmapisakmp authorization list accp

GW(config)\#crypto map vpnmap client configuration address respond

**在VPN网关接口上调用静态map**

GW(config)\#int f0/0

GW(config-if)\#crypto map vpnmap

INSIDE

INSIDE(config)\#int f0/0

INSIDE(config-if)\#ip add 192.168.1.2 255.255.255.0

INSIDE(config-if)\#no shutdown

INSIDE(config-if)\#exit

INSIDE(config)\#no ip routing

INSIDE(config)\#ip default-gateway 192.168.1.1

INSIDE(config)\#exit

**ISP**

ISP(config)\#int f0/0

ISP(config-if)\#ip add 10.0.0.2 255.255.255.0

ISP(config-if)\#no shutdown

ISP(config-if)\#exit

ISP(config)\#int f1/0

ISP(config-if)\#ip add 12.1.1.1 255.255.255.0

ISP(config-if)\#no shutdown

客户端验证

第一步：Group+key验证: 组名vpngroup密码：cisco

<img src="http://funkyimg.com/i/2kd8q.png" width="553" height="431" />

第二步：xauth 1.5阶段验证：用户名cisco，密码：ccnp

<img src="http://funkyimg.com/i/2kd8r.png" width="553" height="324" />

第三步：验证成功后给VPN客户端从VPN池中分配IP地址，获得了123.1.1.102

<img src="http://funkyimg.com/i/2kd8s.png" width="553" height="292" />

**自动向网关注入一个反向路由,下一跳地址指向12.1.1.2**

GW\#show ip route

Gateway of last resort is 10.0.0.2 to network 0.0.0.0

10.0.0.0/24 is subnetted, 1 subnets

C 10.0.0.0 is directly connected, FastEthernet0/0

123.0.0.0/32 is subnetted, 1 subnets

S 123.1.1.102 \[1/0\] via 12.1.1.2

C 192.168.1.0/24 is directly connected, FastEthernet1/0

S\* 0.0.0.0/0 \[1/0\] via 10.0.0.2

此时VPN客户端的路由表中可以发现去往192.168.1.0网段是从123.1.1.102出去的路由条目。

查看客户端路由表使用route print 命令即可。

<img src="http://funkyimg.com/i/2kd8t.png" width="553" height="370" />

可以实现与内网inside主机的通信了。

<img src="http://funkyimg.com/i/2kd8u.png" width="553" height="214" />

在inside主机上启用HTTP服务，测试VPN

INSIE(config)\#ip http server

INSIE(config)\#ip http authentication local

INSIE(config)\#username huarong privilege 15 password ccnp

INSIE(config)\#

<img src="http://funkyimg.com/i/2kd8v.png" width="553" height="430" />

<img src="http://funkyimg.com/i/2kd8x.png" width="553" height="366" />

<img src="http://funkyimg.com/i/2kd8z.png" width="553" height="438" />

**vpn client 的状态**

<img src="http://funkyimg.com/i/2kd8A.png" width="523" height="374" />

因为没有在Vpn网关上做隧道分离，所以默认所有的流量都要加密，即0.0.0.0-0.0.0.0的流量都走VPN通道

<img src="http://funkyimg.com/i/2kd9u.png" width="553" height="378" />

做隧道分离，任何VPN客户端访问192.168.1.0网段的主机的流量走vpn通道并加密，其余的不加密，正常访问。

GW(config)\#access-list 102 permit ip 192.168.1.0 0.0.0.255 any

GW(config)\#crypto isakmp client configuration group vpngroup

GW(config-isakmp-group)\#acl 102

GW(config-isakmp-group)\#exit

<img src="http://funkyimg.com/i/2kd9v.png" width="513" height="338" />

**关于easy VPN反向路由注入的解释：**

VPN网关向客户端推送完策略后，会注入反向路由，原因是pc访问inside主机时，网关肯定有inside主机的路由，流量是可以过去的，但是返回的包vpn网关没有办法发给pc，如果要让返回的包来到pc，必须要有路由，所以需要注入反向路由，会根据分配给PC的ip地址如123.1.1.102/32位的路由，下一跳指向pc的公网地址12.1.1.2.

**关于easy VPN地址池是否可以分配与内网地址段相同的解释**

其实分配同样网段的ip地址在不冲突的情况下也是可以的，因为路由器是最长匹配，反向路由是32位的，池是可以分配同一个网段，但IP地址不能冲突。
