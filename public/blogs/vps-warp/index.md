# 📌 前言
在 Linux VPS 上以 Proxy 模式 运行 WARP-CLI，使其仅在本地创建一个 Socks5 代理端口，将需要的应用流量通过该端口走 WARP 网络。应用官方程序不容易掉线且恢复容易。


注意：**占内存较大，内存较小记得提前设置好swap，防止内存溢出导致服务器崩溃**


本文章完全参考于[在 Linux VPS 上安装与配置 WARP-CLI 并通过 Socks5 代理接入 WARP 网络](https://googhub.nyc.mn/posts/2025/%E5%9C%A8-linux-vps-%E4%B8%8A%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE-warp-cli-%E5%B9%B6%E9%80%9A%E8%BF%87-socks5-%E4%BB%A3%E7%90%86%E6%8E%A5%E5%85%A5-warp-%E7%BD%91%E7%BB%9C)和[Nodeseek](https://www.nodeseek.com/post-429175-1),本文章主要用于个人备份。
# 🛠 教程


## 🚀 安装 WARP-CLI
### 步骤 1：更新软件源


```bash
sudo apt update
```


> ⚠️ 如果提示 sudo: command not found，请先安装 sudo：


```bash
apt install sudo -y


```
### 步骤 2：安装 gnupg（必需）
WARP 软件仓库启用了 GPG 数字签名，系统需要 `gnupg` 校验签名。


```bash
sudo apt install gnupg -y
```


### 步骤 3：导入 Cloudflare GPG 公钥


```bash
curl https://pkg.cloudflareclient.com/pubkey.gpg \
| sudo gpg --yes --dearmor \
--output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
```


### 步骤 4：添加 Cloudflare 软件仓库


```bash
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" \
| sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```


说明：
- `$(lsb_release -cs)` 会自动获取当前系统代号，如 `focal`、`jammy`、`bullseye` 等。




### 步骤 5：再次更新软件源
```bash
sudo apt update
```


### 步骤 6：安装 WARP-CLI
```bash
sudo apt install cloudflare-warp -y
```
安装完成后，`warp-cli` 命令即可使用。




## ⚙️ 配置与运行 WARP-CLI


### 步骤 1：注册 WARP 免费账户
```bash


warp-cli registration new
```


首次运行会提示是否同意用户协议，输入：
```
y
```
并回车确认。




### 步骤 2：设置为 Proxy 模式
默认模式会接管全局流量，风险是 VPS 可能失联。
必须切换到代理模式：
```bash
warp-cli mode proxy
```
### 步骤 3：设置 Socks5 代理端口
例如设置为 `50000`：


```bash
warp-cli proxy port 50000
```
说明：
- 代理仅监听本地 `127.0.0.1:30000`
- 之后需要通过此端口访问 WARP 网络


### 步骤 4：（可选）切换协议为 MASQUE


WARP 默认使用 WireGuard 协议，你可以切换到 MASQUE 协议：
```bash
warp-cli tunnel protocol set MASQUE
```
查看是否切换成功：
```bash
warp-cli settings | grep protocol


```
### 步骤 5：连接 WARP
```bash
warp-cli connect
```


如果成功，会输出：
```
Success
```


## 🔍 验证 WARP 代理是否正常工作
使用 curl 通过 Socks5 代理访问：


```bash
curl ifconfig.me --proxy socks5://127.0.0.1:30000
```


- 如果输出的是 WARP 分配的 IPv4/IPv6 地址 → 代理工作正常


- 如果报错，请确认端口号与 `warp-cli proxy port` 设置一致


## 📊 （可选）检测 WARP 分配的 IP 质量
你可以使用第三方脚本检测 IP 可用性：


bash <(curl -Ls IP.Check.Place) -x socks5://127.0.0.1:30000
说明：


- 该脚本由社区维护，可能会失效或地址变更


- 输出结果主要用于参考（例如判断能否解锁某些服务）


## 🌐 （可选）为 IPv6 Only VPS 添加 WARP IPv4 出口


通过 `onekey-tun2socks` 工具，可以让 IPv6 Only VPS 拥有 IPv4 出口。


### 步骤 1：下载脚本并赋予权限


```bash
curl -L https://raw.githubusercontent.com/hkfires/onekey-tun2socks/main/onekey-tun2socks.sh -o onekey-tun2socks.sh
chmod +x onekey-tun2socks.sh
sudo ./onekey-tun2socks.sh -i custom


```
> ⚠️ 如果 VPS 无法访问 GitHub，请使用代理或反代替换下载链接。


### 步骤 2：输入脚本参数
运行脚本后按提示输入：


1. Socks5 服务器地址


```
127.0.0.1
```


2. Socks5 服务器端口（需与 warp-cli 设置一致，如 30000）


```
30000
```


3. 用户名
直接回车（留空）


4. 密码
直接回车（留空）


### 步骤 3：等待脚本自动配置


- 脚本会自动安装依赖、配置 TUN 虚拟网卡


- 设置完成后，IPv6 Only VPS 将获得 WARP 提供的 IPv4 出口


# ✅ 总结
- warp-cli 默认接管全局流量，务必使用 Proxy 模式 避免 VPS 失联


- 完成配置后，可以通过本地 Socks5 代理访问 WARP 网络


- 若 VPS 仅支持 IPv6，可结合 onekey-tun2socks 提供 IPv4 出口


- IP 检测脚本等可选步骤仅供参考，非必需


