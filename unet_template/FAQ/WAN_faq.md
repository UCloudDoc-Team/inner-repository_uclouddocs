# 外网访问相关
## 绑定在ULB上的EIP是否能重新绑定至主机?
可以的。EIP支持绑定在多种类型的云资源上，如ULB、云主机、NAT网关等。

## 我的主机只有内网IP，可以通过哪些方式访问这台机器呢？
方法一：通过云主机管理界面中应急登录功能进行登录。
方法二：通过在有外网的机器配置端口映射访问

用户必须同时在同一机房拥有两台机器,且其中一台机器有外网IP，以Centos系统为例:
假设 A机器 内网IP:1.1.1.1 外网IP:2.2.2.2     B机器 内网IP:1.1.1.2
通过ssh命令将目标B机器ip的22端口映射到A机器的外网ip的某个端口上去
命令格式:
    ssh -C -f -N -g -L 本地端口:目标IP:目标端口 用户名@目标IP
步骤:在先登录A机器，执行ssh命令:
    ssh -C -f -N -g -L 5000:1.1.1.2:22 root@1.1.1.2
之后外网即可通过以下命令访问B机器:
    ssh 2.2.2.2 -p 5000

## 为什么某地无法访问我在UCloud上的服务器?
可以通过第三方ping工具，例如ping.chinaz.com，检测服务器是否能多地可达。若都不可达，可排查服务器的服务是否正常运行。
若多地不可达，可能是骨干网络出现故障。若只是某地不可达，可能是该地本地路由的问题，可以找当地运营商申诉。

## 不同地域，例如华北一和广州，能否做到内网互通？
可以通过UCloud的高速通道(UDPN)实现，具体请咨询技术支持或客户经理。

