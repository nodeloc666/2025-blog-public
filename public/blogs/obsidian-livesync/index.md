# 前言

利用服务器自部署**obsidian-livesync**来实现实时同步，基本速度很快，实现 obsidian 的优雅同步，再利用 remote 进行备份，简直完美！  
去网上寻找相关教程，没有找到较为简单易懂的教程，总是看的云里雾里的，所以我根据自己的部署过程以及踩的坑进行一次总结与留档。  
**官方介绍：**

> Self-hosted LiveSync (自搭建在线同步) 是一个社区实现的在线同步插件。使用一个自搭建的或者购买的 CouchDB 作为中转服务器。兼容所有支持 Obsidian 的平台。
> 

# 部署

## 前提

- 一台服务器(感谢由 [YVGS](https://yv.gs/aff.php?aff=72)提供的高性能服务器)
- docker
- obsidian 软件
- Nginx(反代用) 

## 部署过程
    
### 1. 安装 docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh & sudo sh get-docker.sh
```
 测试 `sudo docker run hello-world`
 
### 2. 运行 obsidian-livesync
1. 新建文件夹**obsidian-livesync**(方便管理)

```bash
mkdir obsidian-livesync && cd obsidian-livesync
```
1. 新建 `local.ini`

```bash
[couchdb]
single_node=true
max_document_size = 50000000
[chttpd]
require_valid_user = true
max_http_request_size = 4294967296
[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html
[httpd]
WWW-Authenticate = Basic realm="couchdb"
enable_cors = true
[cors]
origins = app://obsidian.md,capacitor://localhost,http://localhost
credentials = true
headers = accept, authorization, content-type, origin, referer
methods = GET, PUT, POST, HEAD, DELETE
max_age = 3600
```
官方配置地址 [obsidian-livesync](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_own_server_cn.md)，照抄就行
1. 运行

```bash
sudo docker run -d --restart always -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password -v /path/to/local.ini:/opt/couchdb/etc/local.ini -p 5984:5984 couchdb
```
|变量|值|
|---|---|
|COUCHDB_USER|admin(自定义)|
|COUCHDB_PASSWORD|password(自定义)|
|/path/to/local.ini|更改成实际地址|

1. 测试是否运行成功

```bash
sudo docker ps |grep couchdb
```
1. 反代域名并配置SSL证书,假设为 `https://a.com`
2. 访问 `https://a.com/_utils`,输入帐号密码后进入管理页面    
3. 点击 Create Database, 然后根据个人喜好创建数据库。

### 3. 客户端配置
**弄之前记得备份或开新库来一遍，别瞎点把自己数据给玩没了**


1. 安装第三方插件**Self-Hosted LiveSync**

2. 以 Win 11 配置为例
    - 先根据图片步骤一步一步来
	
![a1.jpg](https://img.uutv.dpdns.org/file/a1.jpg)

![a2.jpg](https://img.uutv.dpdns.org/file/a2.jpg)

![a3.jpg](https://img.uutv.dpdns.org/file/a3.jpg)

![a4.jpg](https://img.uutv.dpdns.org/file/a4.jpg)

![a5.jpg](https://img.uutv.dpdns.org/file/a5.jpg)

![a6.jpg](https://img.uutv.dpdns.org/file/a6.jpg)

![a7.jpg](https://img.uutv.dpdns.org/file/a7.jpg)

![a8.jpg](https://img.uutv.dpdns.org/file/a8.jpg)

### 其他端
1. 上述相同操作，重复一遍     
    1. 去最开始我们点击手动配置的地方，选择 `复制`，设置密码后得到配置链接，然后选择 `使用URI`，粘贴并输入密码，选择导入，若弹出其他选项，可根据实际情况进行选择，一般默认即可。
	1. 此时，不出意外的话，已经可以正常使用了。更高级的功能可以自己摸索。
    1. 备份正好使用 remote 传到网盘/对象存储，比 git 方便不少。
	
# 后记

##### 1. 关于我为啥不用 docker-compose.yaml 直接部署？

答：失败了，不管怎么弄都不成功，网上查询后发现也有人跟我一样，具体为啥我也不清楚

##### 2. 关于能否同步配置？

答：没研究，有需求的可自行研究，有说**可以**的，也有说**不可以**，因为我主要是手机端与 windows 端同步，两者有些配置同步会出错，故我主要是**同步文章**。

##### 3. 为啥反代并配置 SSL？

答：手机端需要**HTTPS**安全连接。