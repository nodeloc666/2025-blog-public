# 前言

书接上文，使用自建的**CouchDB**配合**obsidian-livesync**插件实现了**实时同步**。但是有大佬说使用**数据库**同步时间长了会出现**同步错误**，建议使用**minio S 3**进行同步，近两天研究了一下，只能说踩了不少坑(可恶的**堡塔**)，总算是能稳定同步了。  
话不多说，直接上部署教程。(由于我对很多配置与命令不熟，故很多都是**面板**进行操作的)  
根据我测试的结果：**1 panel**部署起来是最便捷的（也可能是凑巧）

# 部署

## 前提

- 服务器一台(感谢由 [YVGS](https://yv.gs/aff.php?aff=72)提供的高性能服务器)
- 已安装docker，可参考我的 [obsidian-livesync服务器部署教程](https://blog.cccd.cloudns.be/obsidian-livesync)
- nginx(反代使用)
- obsidian

## 服务端部署（默认已有 docker）
### 1. 部署 minio
```bash
docker run \
-p 9000:9000 \
-p 9090:9090 \
--name minio \
-v /data/minio/data:/data \
-v /data/minio/config:/root/.minio \
-e "MINIO_ROOT_USER=admin" \
-e "MINIO_ROOT_PASSWORD=12345678" \
minio/minio server /data --console-address ":9000" --address ":9090"
```
### 2.反代域名
反代我不多说，可参考[官网反代配置](https://min.io/docs/minio/linux/integrations/setup-nginx-proxy-with-minio.html)，后台和 API 都反代一下。可利用面板的反向代理直接进行反代。（我用的宝塔面板反代）  
**这里可能有个坑，等到后面再说。**
### 3. 访问后台

1. 访问后台：IP+端口:9000/**反代的域名**
2. 登入后，新建 buckets 和 Access Keys 并**记录**


## 客户端配置
**弄之前记得备份或开新库来一遍，别瞎点把自己数据给玩没了**

1. 安装第三方插件**Self-Hosted LiveSync**
    
2. 以 Win 11 配置为例

![a1.jpg](https://img.uutv.dpdns.org/file/a1.jpg)

![a2.jpg](https://img.uutv.dpdns.org/file/a2.jpg)

![b3.png](https://img.uutv.dpdns.org/file/b3.png)

![b4.png](https://img.uutv.dpdns.org/file/b4.png)

![b5.png](https://img.uutv.dpdns.org/file/b5.png)

![b6.png](https://img.uutv.dpdns.org/file/b6.png)
这里随便选一个就行
![b7.png](https://img.uutv.dpdns.org/file/b7.png)
然后改成图示或者按需求修改
![b10.png](https://img.uutv.dpdns.org/file/b10.png)
然后客户端就设置完成了


1. 其他端类似操作。

### 反代坑点(api 域名 403)

如果所有的都配置完并重启打开，提示如下图的情况，好恭喜你，跟我一样，踩了这个坑。
![b8.png](https://img.cccd.cloudns.be/file/b8.png)

#### 原因与解决办法
**原因**：

1. 查找日志与控制台会发现，obsidian 一直是**HEAD 请求**，而**HEAD**请求是会被**403**的。

![b11.png](https://img.uutv.dpdns.org/file/b11.png)


1. 经过多次测试，当你将存储桶改成Public，直接**[https://api+URL/桶名](https://api+url/%E6%A1%B6%E5%90%8D)** ，去访问的时候，会发现是正常的（说明桶是正常的），查看宝塔日志会发现**GET 请求**是**正常**的.
2. 当直接 URL填 [http://ip:9090](http://ip:9090/) 时，会发现能正常使用

**解决办法**： 综合来看，说明时**反代出了问题**。那知道原因就好办了。  
那既然 HEAD 请求会 403，而 GET 时 200，那就让**HEAD 变成 GET**。  
找到 nginx 反代的配置文件  
我用的是宝塔,配置文件在：`/www/server/panel/vhost/nginx`，然后找到相应的配置，在 locations 里面添加：

```
proxy_cache_convert_head off;
```
![b14.png](https://img.uutv.dpdns.org/file/b14.png)

#### 这样就解决了

# 后记

1. 经测试，**1 panel**应该直接反代后就能正常使用，不用进行额外配置**（我是这样的）**
2. 该同步方式，插件会依次读取存储桶里的文件，然后依次写入直到全部读完一遍。当你里面文件有 100 个，就会按顺序依次写入 100 次。（不是只读取最后一次）**【大概是这样，只是根据它同步方式推理出来的，不一定对】**
3. nginx 反代问题不是只有那一种方法，可以根据需求自行修改。
4. 我给出的坑点仅是我遇到的，可能还有其他导致 api 用不了的，可自行查看 log 进行排查或者问问 AI。