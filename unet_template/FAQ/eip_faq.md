# 绑定外网弹性 IP
## 外网弹性IP有哪些计费方式，具体的计费规则是什么？
1.常规带宽模式下（非共享带宽），可申请流量计费IP，或将存量IP切换为流量计费，共享带宽模式不支持流量计费的外网IP；

2.流量计费下，流量费用和IP地址费用分开付费：

- 外网IP地址（无带宽）费用：可按需. 按月、按年支付(裸IP)；
- 流量费用：不同地域价格不同，每天凌晨根据每个IP所使用流量进行结算。

> （1）流量计费的IP，不收取带宽费用；  
（2）流量统计仅统计出口流量，不统计入口流量；  
（3）流量计费的IP，试用规则普通IP相同，试用IP的流量不进行扣费；  
（4）将存量IP切换为流量计费IP，会对已购带宽进行自动退费；  
（5）流量计费IP，不支持绑定带宽包；  

3.若当日扣费失败，则会在下一天再次进行扣费。


## 绑定外网弹性IP后多久生效?
一般情况下, 5秒内生效.

## 一台云主机是否可以绑定多个外网弹性IP?
可以绑定任意多个.

## 一个弹性IP是否支持绑定多个云主机 
单个外网弹性IP只能绑在一个云主机上, 但可以通过解绑绑定的方式转给另一台云主机. 绑定多个主机的需求, 可考虑用NAT网关解决.

## 是否可以更换IP？
待添加
