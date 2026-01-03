# 前言


最近**uskg**域名又挂了，很多人都深受其害（**包括我**），都开始转移域名了。那么就有很多人转向了**cloudns的免费域名**，虽然域名不好看，还是**双向解析**，但是架不住它**稳定**啊，拿来当备用还是非常不错的。  
但是双向解析总是麻烦的，那有没有办法能优雅的处理双向解析的域名，能**直接**在**cloudflare**上去**解析**呢？看过不少大佬帖子和自己的实践，给出简洁易懂的两个方案。
### 这里不讲注册，大家自行解决，一般IP干净一点都是随便注册的


# 教程


## 方法一（最稳定）


### cloudflare端


1. 正常托管到cloudflare
2. 然后在CF添加DNS记录(以我的 `ok.cloud-ip.biz` 为例)


```
A	ns	8.8.8.8
```
![c1.png](https://img.cccd.cloudns.be/file/c1.png)




注意：这里的 `ns` 随意，`8.8.8.8` 也随意
3. cloudflare 结束,共1条记录


### cloudns 端
    
1. 添加第一次**NS**记录


```
ok.cloud-ip.biz	NS	a.ns.cloudflare.com
ok.cloud-ip.biz	NS	b.ns.cloudflare.com
```
注意：这里 `a.ns.cloudflare.com`, `b.ns.cloudflare.com` 对应着cloudflare给你分配的**名称服务器**


2. 添加第二次**NS**记录(证书验证)


```
acme-challenge.ok.cloud-ip.biz	NS	a.ns.cloudflare.com
acme-challenge.ok.cloud-ip.biz	NS	b.ns.cloudflare.com
```


3. 添加第三次**NS**记录


```
ns.ok.cloud-ip.biz	NS	a.ns.cloudflare.com
ns.ok.cloud-ip.biz	NS	b.ns.cloudflare.com
```


4. 添加**CNAME**记录


```
*.ok.cloud-ip.biz	CNAME	ns.ok.cloud-ip.biz
```
5. **结束**，共7条记录


![c2.png](https://img.cccd.cloudns.be/file/c2.png)


此时，你尝试去ping`*.ok.cloud-ip.biz`(如`1.ok.cloud-ip.biz`)有IP显示ping通就**大功告成**了


![c3.jpg](https://img.cccd.cloudns.be/file/c3.jpg)


# 方法二（最简洁）


## cloudflare 端


1. 正常托管cloudflare，然后拿到cloudflare分配给你的名称服务器
2. 结束


## cloudns 端


1. 添加第一次**NS**记录


```
ok.cloud-ip.biz	NS	a.ns.cloudflare.com
ok.cloud-ip.biz	NS	b.ns.cloudflare.com
```
注意：这里`a.ns.cloudflare.com`,`a.ns.cloudflare.com`对应着cloudflare给你分配的**名称服务器**


2. 添加第二次**NS**记录(证书验证)


```
acme-challenge.ok.cloud-ip.biz	NS	a.ns.cloudflare.com
acme-challenge.ok.cloud-ip.biz	NS	b.ns.cloudflare.com
```


3. 添加**CNAME**记录


```
*.ok.cloud-ip.biz	CNAME	cf.877774.xyz
```


5. **结束**,共5条记录


![c4.png](https://img.cccd.cloudns.be/file/c4.png)


此时，你尝试去ping`*.ok.cloud-ip.biz`(如`1.ok.cloud-ip.biz`)有IP显示ping通就**大功告成**了


![c5.png](https://img.cccd.cloudns.be/file/c5.png)


# 后记


1. 此方法适用于cloudns能**托管**到CF的域名
    
2. 此方法真正能直接在cloudflare**直接添加**而不用去cloudns上去添加解析的**只有**`*.ok.cloud-ip.biz`,当你使用`ok.cloud-ip.biz`直接解析时会发现不通，报错之类的，这需要到**cloudn**s上去设置解析
    
3. 这方法也只适用于开启了小黄云的解析，关了小黄云会失效，还是要到cloudns上去单独解析（理解原理就知道为啥了）
    
4. 上述的给出的所有记录都是填写完成后的显示，不是让你完整输入 `ns.ok.cloud-ip.biz` 之类，例如:
5. **方法二中**：`cf.877774.xyz`理论上说可以改成任意一个套了cloudflare的域名，比如说`www.visa.com.sg`
6. **从原理来说这一步也不是一定要CNAME，也可以是A记录到CF的IP，具体我没试过，大家不嫌麻烦可以试试**
7. 此方法只是方便解析了，不用那么麻烦了，它该有的特性还是一成不变的，比如说不能直接在cloudflare上去cname cloudflare的pages，需要去cloudns上去添加才有效，像添加**MX邮件转发**也需要到**cloudns**上去添加。


8. 所有解析在CF上都得开启小黄云
    
9. `ok.cloud-ip.biz` 才能像`*.ok.cloud-ip.biz`一样使用呢？从原理上说，只要给`ok.cloud-ip.biz`也弄上cf的ip就行，实际我没测试，原则上可以
    
10. 大家有任何问题都可以评论区留言哦，看到一定会回复的


