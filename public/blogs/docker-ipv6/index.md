# 前言
一般默认docker容器是不支持IPv6的，想要容器支持IPv6，需要单独进行配置。


# 步骤
## 1. 开启IPv6
1. 编辑`/etc/docker/daemon.json`


```bash


{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64",
  "experimental": true,
  "ip6tables": true
}


```
- ipv6：启用 IPv6 支持。
- fixed-cidr-v6：为默认桥接网络分配一个 IPv6 子网。
- experimental：启用实验性功能（如 ip6tables）。
- ip6tables：启用 IPv6 数据包过滤规则。


2. 重启 docker


```bash


sudo systemctl restart docker
```


这样其实已经为bridge的docker网络配置了IPv6.


## 2.创建新IPv6网络


-  创建IPv6并测试网络：


```bash
docker network create --ipv6 --subnet 2001:db8::/64 ipv6net
```


   - 启动容器并测试 IPv6 连接： 


```bash
docker run --rm --network ipv6net busybox ping -6 -c1 2400:3200::1


```
- 验证输出是否显示成功的 IPv6 数据包传输。


## 3. 使用基于bridge网络（已支持ipv6）


可以参考如下进行创建：


```bash


networks:
  L-network:
    driver: bridge
    enable_ipv6: true
    ipam:
      config:
        - subnet: 2001:db8:b::/64
          gateway: 2001:db8:b::1


```
















