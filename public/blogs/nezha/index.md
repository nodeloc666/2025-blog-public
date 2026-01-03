# 前言
哪吒面板v1总是有这那的错误、不通，今天找到了一个非常不错的教程，使用1panel反代部署，单个域名（套了CF的CDN）直接解决。非常方便。帮助解决套了CDN域名不通问题。




**注意：此文章转载自[风尘落微雨](https://www.weirain.com/index.php/archives/309/)，我仅在此基础上做了优化扩充和排版整理。**
### 一、域名解析
面板域名以nezha.a.com为例
1. Cloudflare中添加A记录nezha.a.com指向Dashboard 服务器 IP，开启小黄云。
2. a.com域名设置页面——网络，选择开启**WebSockets** 和 **gRPC** 
3. SSL/TLS模式选择完全/完全（严格）


### 二、哪吒面板搭建
1. 在面板服务器中，运行以下安装脚本：
```bash
curl -L https://raw.githubusercontent.com/nezhahq/scripts/refs/heads/main/install.sh -o nezha.sh && chmod +x nezha.sh && sudo ./nezha.sh
```
如果你的服务器位于中国大陆，可以使用镜像：
```bash
curl -L https://gitee.com/naibahq/scripts/raw/main/install.sh -o nezha.sh && chmod +x nezha.sh && sudo CN=true ./nezha.sh
```


以 Docker 安装为例，安装过程按提示输入以下信息：


1. 请输入站点标题: **自定义你的站点标题（自行设置）。**


2. 请输入暴露端口: **公开访问端口（可直接默认 8008，回车就行）。**(如果不需要ip+端口直接访问，端口开不开无所谓)
3. 请指定安装命令中预设的 nezha-agent 连接地址：Agent对接地址【域名/IP:端口】:`nezha.a.com:443`
4. 是否希望通过 TLS 连接 agent : Agent 通信使用 TLS 连接:**y**
5. 请指定后台语言:  **选择语言偏好。**


安装完后，可以先用你的 ip:8008 就能访问了。如果没问题，我就进行下一步
### 三、反代设置


1. 1panel面板中新建反向代理网站，网站 ——> 创建网站 ——> 反向代理，主域名填写`nezha.a.com`，代理地址填写`127.0.0.1:8008`


2. **证书**页面自行设置Acme账户、DNS账户为nezha.a.com申请证书或者**直接去CF申请证书拿到这里来用也是一样的。** 
	***注意：证书一定申请，直接用CF的也是OK的***
3. 网站页面点击nezha.a.com进入网站设置，开启https。


4. 点击配置文件，最下方添加以下代码，保存并重载。
```
# upstream 配置
upstream dashboard {
    keepalive 512; 
    server 127.0.0.1:8008; 
}
```
![m1.png](https://img.uutv.dpdns.org/file/blog/m1.png)
![m2.png](https://img.uutv.dpdns.org/file/blog/m2.png)
5.打开网站 ——> 反向代理 ——> 源文，使用以下代码替换原内容，点击确认。至此，反向代理设置完成。
![m3.png](https://img.uutv.dpdns.org/file/blog/m3.png)
```
location ^~ / {
    proxy_pass http://127.0.0.1:8008; 
    proxy_set_header Host $host; 
    proxy_set_header X-Real-IP $remote_addr; 
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_set_header REMOTE-HOST $remote_addr; 
    proxy_set_header Upgrade $http_upgrade; 
    proxy_set_header nz-realip $http_cf_connecting_ip;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1; 
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_buffer_size 128k;
    proxy_buffers 4 128k; 
    proxy_busy_buffers_size 256k;
    proxy_max_temp_file_size 0;
    add_header X-Cache $upstream_cache_status; 
    add_header Cache-Control no-cache; 
    proxy_ssl_server_name off; 
    proxy_ssl_name $proxy_host; 
    add_header Strict-Transport-Security "max-age=31536000"; 
}
underscores_in_headers on;
set_real_ip_from 0.0.0.0/0; # CDN 回源 IP 地址段
real_ip_header CF-Connecting-IP; # CDN 私有 header，此处为 CloudFlare 默认
# gRPC 服务
location ^~ /proto.NezhaService/ {
    grpc_set_header Host $host;
    grpc_set_header nz-realip $http_CF_Connecting_IP; 
    grpc_read_timeout 600s;
    grpc_send_timeout 600s;
    grpc_socket_keepalive on;
    client_max_body_size 10m;
    grpc_buffer_size 4m;
    grpc_pass grpc://dashboard;
}
# WebSocket 服务
location ~* ^/api/v1/ws/(server|terminal|file)(.*)$ {
    proxy_set_header Host $host;
    proxy_set_header nz-realip $http_cf_connecting_ip; 
    proxy_set_header Origin https://$host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_pass http://127.0.0.1:8008;
}
```


**注意：宝塔面板类似**


1. 创建一个网站（不是直接反代，就是添加PHP站点，纯静态）
2. 打开配置文件菜单，在最后面填上**第一段代码**，然后点保存：(如果在安装哪吒监控时，你自定义了端口，把下面代码中的8008改为你自定义的端口)


**注意：如果这段代码加不上去，可以再改成`dashbord_v2`，应该就OK了，这样访问的时候，只能`nezha.a.com/dashboard/login`,直接访问`https://nezha.a.com/dashboard`可能会报错。**


![m5.png](https://img.uutv.dpdns.org/file/blog/m5.png)
3. 打开反向代理 —— 添加反向代理，`127.0.01:8008`，保存。
4. 然后点击上图中添加的反向代理目录中的配置文件，将里面的内容全选删除，并替换为**第二段代码**，然后点保存：(如果在安装哪吒监控时，你自定义了端口，把下面代码中的8008改为你自定义的端口)
![m4.png](https://img.uutv.dpdns.org/file/wuyu/m4.png)
5. **然后申请证书，HTTPS访问。**


**注意：证书一定申请，不申请可能会出现报错或者服务器不上线等问题。**


### 四、哪吒面板设置


#### 基本设置


1. 登录到 Dashboard 配置界面`https://nezha.a.com/dashboard`， 初始用户名、密码均为`admin`，登录后立即进入管理页面点击头像 ——> 个人信息 ——> 更新个人资料修改用户名和密码。


2. 点击头像 ——> 系统设置，填写站点名称、设置语言，Agent对接地址【域名/IP:端口】填写`nezha.a.com:443`，勾选Agent 使用 TLS 连接，点击确认即可。


#### 关于被控端网页SSH
禁用：
```bash
sed -i 's/disable_command_execute: false/disable_command_execute: true/' /opt/nezha/agent/config.yml && systemctl restart nezha-agent
```
开启：
```bash
sed -i 's/disable_command_execute: true/disable_command_execute: false/' /opt/nezha/agent/config.yml && systemctl restart nezha-agent
```


#### 添加 telegram 通知


1. 通知页面点击“+”，创建通知，名称自定，URL填写 `https://api.telegram.org/bot<token>/sendMessage?chat_id=<用户ID>&text= #NEZHA #`
2. 将你的机器人<Token> 和 <用户ID> 替换为实际值
3. 分组——通知，页面点击“+”，编辑通知分组，名称自定，通知勾选提前创建的 telegram 通知，确认即可。


#### 卸载
1. 面板端
```bash
curl -L https://raw.githubusercontent.com/nezhahq/scripts/refs/heads/main/install.sh -o nezha.sh && chmod +x nezha.sh && sudo ./nezha.sh
```
里面有卸载面板的命令
2. 监控端
```bash
./agent.sh uninstall
```


#### 其他更多配置参考
[哪吒官网](https://nezha.wiki/)和[在线编辑配置](https://nzinfo.lvbjj.ggff.net)




