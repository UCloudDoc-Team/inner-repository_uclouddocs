# IPv6转换相关

## 关闭IPv6转换功能后，IPv6地址是否保留？

IPv6转换功能一旦关闭，EIP对应的IPv6地址将被回收，但IPv4地址不受影响。

## EIP删除后，IPv6地址是否保留？

EIP删除后，该EIP的“IPv6转换功能”将会关闭，因此EIP对应的IPv6地址将被回收。

## 对外提供IPv6服务时，外网防火墙规则和ACL规则有何特殊配置？

IPv6流量经过NAT64网关集群转换后，源IPv6地址将被转换为一个固定的IPv4段内的地址(100.96.0.0/11)，需要保证外网防火墙规则和ACL规则放行该网段。

## IPv6的访问流量，云主机中如何查看IPv6源地址？

IPv6流量经过NAT64网关集群转换后，源IPv6地址将被转换为一个固定的IPv4段内的地址，因此无法看到IPv6源地址。

## 从客户端测试IPv6地址时，为何访问不通？

请确保测试的客户端配置有IPv6地址，且有IPv6的对外访问网络环境。
