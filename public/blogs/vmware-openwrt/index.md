# 声明


1. 非本人原创。本教程结合各位大佬所长和我自己部署踩的坑综合在一起写下此篇文章，基本包含全面，成功率高。,**码字不易，转载请一定保留文章来源**
2. 本教程来自 [OpenWRT在VMware Workstation上的部署（For Windows） - 开发调优 - LINUX DO](https://linux.do/t/topic/1189205)
3. 英文版教程, [OpenWRT_For_VMware](https://github.com/AASWEETBOY/OpenWRT_For_VMware),**非本人原创。**
4. 其他参考教程，[在 VMWare 中安装 OpenWrt](https://shepherd-xie.github.io/2024/09/26/deploy-openwrt-on-vmware/)


# 前言


身处在限制较多的校园网下，桥接无法使用，openwrt用不了？本教程将完整的教你在Windows平台部署openwrt，并通过openwrt访问互联网，让深受校园网其害下的“穷学生”也来感受一下软路由的快乐！！！在其他平台上可以仿照本教程迁移。


### 1. VMware 介绍


1. **VMware**是一大做虚拟机的厂商，虚拟机就是在实体机的操作系统上再利用实体机的部分CPU、内存与硬盘资源虚拟出一个完整可用的操作系统。简单来说，VMware Workstation是在Windows操作系统上再运行一个Windows系统/Linux系统（感觉到**套娃**之意味即可）。
2. 使用虚拟机有几大好处：


   * 虚拟机与实体机（主机）在文件系统上是隔离的，即虚拟机中病毒不会使实体机中毒。即使虚拟机“被玩坏了”，也不会使实体机的系统损坏。
   * 虚拟机可以使用快照（需要自己设置）快速恢复至初始状态，可折腾性很高


### 2. OpenWRT


1. **OpenWRT**是一个嵌入式Linux系统，常用来**作软路由**。软路由区别于硬路由，硬路由的固件镜像烧录进开发板，基本无法改动，且部分功能直接由各个板载芯片提供，相比于软路由无需太多代码驱动（在能效上，硬路由占优）；而软路由的固件镜像是可以自编译，可以加入自己想要的功能插件，使其功能变得强大无比。
2. 本文重点不在于编译OpenWRT系统，**重点在于使用相关资源构建出可以代理其他多个虚拟机流量的软路由系统**。所以，本教程使用的是知名大佬**eSir的固件**（稳定、原生且更新不像Windows那般频繁——仅一年2-4次，现在基本是上半年更新1次，下半年更新1次；**推荐下载**buddha分支中的“**openwrt-stable-24.10.4-buddha-version-v2\[2025]-x86-64-generic-squashfs-uefi.img**”的镜像包，因为eSir大佬主要维护的是这个**buddha分支**）


### 3. 科普一下VMware的NAT模式、Host-only模式（仅主机模式）与桥接模式


#### 1 . 桥接模式


将虚拟机的虚拟网卡直接“桥接”到宿主机的物理网卡，使虚拟机成为物理局域网中的一台独立主机，与宿主机及其他物理设备处于**同一网段**。虚拟机通过物理网络的DHCP服务器获取IP（或手动配置静态IP），网关、DNS等网络参数与物理网络一致。


#### 2. NAT 模式


虚拟机通过宿主机的“网络地址转换（NAT）”功能访问外部网络。VMware 会创建一个虚拟网卡（如VMnet8），虚拟机位于该虚拟网卡对应的**私有网段**（默认为 `192.168.x.x`），宿主机充当“网关”和“NAT路由器”，将虚拟机的网络请求转换为宿主机的物理IP发送出去，外部网络无法直接访问虚拟机。


#### 3.  Host-only模式（仅主机模式）


虚拟机与宿主机组成**完全隔离的私有网络**，仅允许宿主机与虚拟机之间通信，无法直接访问外网（默认无网关）。VMware 会创建虚拟网卡（如VMnet1），通过虚拟DHCP服务器为虚拟机分配私有IP（如`192.168.10.x`），形成独立的“小局域网”。


### 4. 本教程要实现的系统拓扑图


本项目就是“偷梁换柱”，即把 Host-only 模式（仅主机模式）的子网网络的网关强行改为 openWRT，所以 openWRT 的 LAN 口网卡必须配置为 Host-only 模式（仅主机模式）。而 openWRT 的 WAN 口网卡可配置为**NAT 模式**（**常用于校园网**，因为校园网有 AP 隔离——设备间无法直接通信，无法使用 VMware 的**桥接模式**。


![be41188ff755065571374f9684a3d11c0b86565a_2_690x320.jpeg](https://img.dlb.xx.kg/file/blog/RIw6duck.jpeg)


# 教程


安装好 `VMware Workstation`、`StarWind V 2 V Converter`（把 img 格式的 OpenWRT 镜像转换为 vmdk 格式的工具）软件与准备好 OpenWRT 镜像包（`openwrt-xxx.img`)


### 1. 转换镜像


1. 打开 `StarWind V2V Converter`
2. 在 `Select the location of the image to convert` 一栏，选择 `Local file`
3. 在 `Source image` 一栏，选择 `openwrt-xxx.img` 的存放位置
4. 在 `Select the location of the destination image` 一栏，选择 `Local file` 
5. 在 `Select destination image format` 一栏，选择 `VMDK`
6. 在 `Select option for VMDK image format` 一栏，选择 `VMware Workstation growable image`
7. 在 `Set destination file name` 一栏，自定义vmdk镜像的名称（通常使用默认名称即可)
8. 最后点击 `convert` 按钮


### 2. 创建虚拟机


1. 打开 VMware Workstation，选择“自定义（高级）”


![openwrt-2.png](https://img.dlb.xx.kg/file/blog/openwrt-2.png)


2. 默认（虚拟机硬件兼容性） → 稍后安装操作系统 


![openwrt-3.png](https://img.dlb.xx.kg/file/blog/openwrt-3.png)


4. `客户操作系统：Linux`，版本： `其他 Linux 6.x 内核 64 位`


![openwrt-5.png](https://img.dlb.xx.kg/file/blog/openwrt-5.png)


5. 虚拟机命名称：**自定义**，并自定义虚拟机文件的**存放位置** 
6. 处理器配置：建议选择 1 个处理器，1 个核心（因为只作软路由不需要太多 CPU 资源）
7. 虚拟机内存：建议选择 1 GB。
8. 网络连接：使用**仅主机模式网络。**


![openwrt-6.png](https://img.dlb.xx.kg/file/blog/openwrt-6.png)


9. I/O 控制器类型：默认。
10. 虚拟磁盘类型：默认。
11. 磁盘：**使用现有虚拟磁盘**。


![openwrt-7.png](https://img.dlb.xx.kg/file/blog/openwrt-7.png)


12. 选择之前转换为 VMDK 格式的镜像位置。
13. **磁盘格式保持不变** 。
14. 单击“完成”。


### 3. 配置虚拟机


1. 编辑虚拟机设置
2. 添加-->网络适配器-->完成


![openwrt-8.png](https://img.dlb.xx.kg/file/blog/openwrt-8.png)


3. 配置 `网络适配器` 、`网络适配器 2` 分别为 `自定义(VMnet2)`、`NAT`,  如图所示。
   **注意：**`网络适​​配器` 和 `网络适配器2` 的顺序在这里非常重要。默认情况下，网络适配器（eth0）将分配给LAN端口，而网络适配器2（eth1）将分配给WAN端口）


![openwrt-9.png](https://img.dlb.xx.kg/file/blog/openwrt-9.png)


4. 选项-->高级-->固件类型-->UEFI


![openwrt-13.png](https://img.dlb.xx.kg/file/blog/openwrt-13.png)


5. 编辑-->虚拟网络编辑器-->更改设置


![openwrt-10.png](https://img.dlb.xx.kg/file/blog/openwrt-10.png)


6. 添加网络-->添加 VMnet 2-->选中 VMnet 2-->仅主机模式-->取消 `使用本地DHCP服务将IP地址分配到虚拟机` -->子网 IP （随意，**但一定不要与其他网络冲突**）-->应用


![openwrt-12.png](https://img.dlb.xx.kg/file/blog/openwrt-12.png)


7. 虚拟机配置已完成


### 4. 配置 Openwrt


1. 在 windows 下，`WIN+R` --> `cmd` 
2. 输入 `ipconfig`，找到 `VMware Network Adapter VMnet2`,发现宿主机 IP 为 `192.168.6.1`


![openwrt-20.png](https://img.dlb.xx.kg/file/blog/hWs88joX.png)


3. 使用 `ping baidu.com` 检查网络，在以上操作完全正确的情况下，网络是可以正常访问的。如果不能正常访问，可以尝试重启网络 `/etc/init.d/network restart`


> 如果在重启网络之后依旧不能正常访问网络，请删除虚拟机 **(右键虚拟机 - 管理 - 从磁盘中删除)** ，并 **手动删除 vmdk 文件** ，重新执行前面的操作。


4. 修改网络配置文件 `vim /etc/config/network`，刚刚已经查看宿主机 IP 为 `192.168.6.1`,这里就要修改 `lan` 的 `192.168.6.1` 为 `192.168.6.2`（这个随意，但**不要与宿主机冲突**）


```
config interface 'lan'
	option type 'bridge'
	option ifname 'eth0'
	option proto 'static'
	option ipaddr '192.168.6.2' 					
	option netmask '255.255.255.0'
	option ip6assign '60'
```


![openwrt-21.png](https://img.dlb.xx.kg/file/blog/openwrt-21.png)


5. 修改完成并保存后，重启网络 `/etc/init.d/network restart`
6. 使用浏览器访问OpenWrt后台。后台地址是 `192.168.6.2`，默认用户名是 `root`，密码为 `空`。


![openwrt-22.png](https://img.dlb.xx.kg/file/blog/openwrt-22.png)


### 5. 其他虚拟机想要通过 openwrt 上网


只需将网络配置改到 `VMnet 2` 即可


![openwrt-23.png](https://img.dlb.xx.kg/file/blog/openwrt-23.png)


![openwrt-30.png](https://img.dlb.xx.kg/file/blog/openwrt-30.png)


# 总结


本教程创建了一个OpenWRT的VMware虚拟机，然后将其用作仅主机模式的虚拟局域网网关，为其他虚拟机提供网络配置服务。


# 后记


1. 此教程只是为了代理所有虚拟机的网络，不代理宿主机网络。
2. 校园网常常不支持**桥接模式**，因为每个客户端都被隔离了（存在**AP隔离**）。只要有AP隔离，就无法使用基于局域网的服务（比如：本地串流、本地文件共享……），也ping不通同一局域子网的其他客户端ip地址。如果不存在这种问题，可以参看[这个回复](https://linux.do/t/topic/1189205/20)中的文章
3. 什么时候宿主机的流量会被 openwrt代理呢？唯有**在无AP隔离的局域网**中，在以上教程的基础上同时参考[这个回复](https://linux.do/t/topic/1189205/20)中的知乎教程


- - -


 **彩蛋（附赠汉化+美化教程）**


1. 先将好不容易配置好的openwrt 虚拟机打个快照
2. 进入 ssh 工具，用 ssh 工具连接到 openwrt


![openwrt-36.png](https://img.dlb.xx.kg/file/blog/openwrt-36.png)


![openwrt-37.png](https://img.dlb.xx.kg/file/blog/openwrt-37.png)


- - -


汉化：


1. 官方版无简中，执行 `opkg update && opkg install luci-i18n-base-zh-cn` 进行汉化


- - -


美化：


2. 访问 [luci-theme-argon](https://github.com/jerrykuku/luci-theme-argon/blob/master/README_ZH.md#%E5%9C%A8%E5%AE%98%E6%96%B9%E5%92%8C-immortalwrt-%E4%B8%8A%E5%AE%89%E8%A3%85)
3. 依次执行


```
opkg install luci-compat


opkg install luci-lib-ipkg


wget --no-check-certificate https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.3.2/luci-theme-argon_2.3.2-r20250207_all.ipk


opkg install luci-theme-argon*.ipk
```


如果上面有报错，例如


```
root@OpenWrt:~# opkg install luci-theme-argon*.ipk
Collected errors:
 * wfopen: luci-theme-argon*.ipk: No such file or directory.
 * pkg_init_from_file: Failed to extract control file from luci-theme-argon*.ipk.
```


则至少保证前两个命令 `opkg install luci-compat` 和
`opkg install luci-lib-ipkg` 执行成功。


然后进入 `系统(system)` --> `Software` --> `Download and install package` 
输入 `https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.3.2/luci-theme-argon_2.3.2-r20250207_all.ipk` 也能进行主题的安装。


![openwrt-34.png](https://img.dlb.xx.kg/file/blog/openwrt-34.png)


5. 进入系统，语言与界面，切换主题，然后保存并应用即可。


![openwrt-35png.png](https://img.dlb.xx.kg/file/blog/openwrt-35png.png)


- - -


## 完结撒花


