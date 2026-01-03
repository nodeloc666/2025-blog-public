# 前言
通过简单的操作，即可通过cloudflare拥有一个自己的邮局项目，支持收发邮件，webhook通知（推荐[我的这篇博客](https://blog.02261004.xyz/posts/20250425_cloudflare%E5%AE%9E%E7%8E%B0webhook%E5%8A%9F%E8%83%BD/)）等功能，关键是完全免费。下面来看看如何部署吧！(部署教程完全根据官方文档，且为了更**简单快速**部署，会有很多删减，如有需求，可参考[官方文档](https://temp-mail-docs.awsl.uk/zh/guide/what-is-temp-mail.html)进行更多功能的添加)
# 教程（通过Github Actions进行前后端分离部署）
## 前提
1. cloudflare账号
2. 托管在cloudflare的域名
3. GitHub账号


## 步骤
### Fork 仓库并启用 Actions
- 在 GitHub fork [仓库](https://github.com/dreamhunter2333/cloudflare_temp_email)
- 打开仓库的`Actions`页面
- 找到`Deploy Backend`点击`enable workflow` 启用 workflow
- 找到`Deploy Frontend`点击`enable workflow`启用 workflow


### 配置Secrets
在仓库页面 `Settings` -> `Secrets and variables` -> `Actions` -> `Repository secrets`, 添加以下`secrets`:
#### 1. 配置基本`Secrets`
|名称|说明|
|:---|:---|
|`CLOUDFLARE_ACCOUNT_ID`|Cloudflare 账户 ID|
|`CLOUDFLARE_API_TOKEN`|Cloudflare API Token|


#### 2.worker后端`secrets`
|名称|说明|
|:---|:---|
|`BACKEND_TOML`|(必要)后端配置文件，[参考此处](https://temp-mail-docs.awsl.uk/zh/guide/cli/worker.html#%E4%BF%AE%E6%94%B9-wrangler-toml-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)|
|`DEBUG_MODE`|(可选) 是否开启调试模式，配置为`true`开启, 默认 worker 部署日志不会输出到 Github Actions 页面，开启后会输出(**如果部署有错误，可开启查看哪里出错了**)|
|`BACKEND_USE_MAIL_WASM_PARSER`|(可选) 是否使用 wasm 解析邮件，配置为`true`开启(需要webhook功能，建议开启)|
**注意**：下面是`BACKEND_TOML`最简配置内容，只有必要变量，如需更多功能，参考[官方文档](https://temp-mail-docs.awsl.uk/zh/guide/worker-vars.html),在`[vars]`下添加即可。如需更完整的配置内容，参考[官方示例](https://github.com/dreamhunter2333/cloudflare_temp_email/blob/main/worker/wrangler.toml.template)
```
name = "temp-email-backend" #对应worker项目的名称
main = "src/worker.ts"
compatibility_date = "2025-04-01"
compatibility_flags = [ "nodejs_compat" ]


# 如果你想使用自定义域名，你需要添加 routes 配置
# routes = [
#  { pattern = "temp-email-api.xxxxx.xyz", custom_domain = true },
# ]




[vars]
# 邮箱名称前缀，不需要后缀可配置为空字符串或者不配置
PREFIX = "tmp"
# 用于临时邮箱的所有域名, 支持多个域名
DOMAINS = ["xxx.xxx1" , "xxx.xxx2"]
# 用于生成 jwt 的密钥, jwt 用于给用户登录以及鉴权，随意但要复杂
JWT_SECRET = "xxx" 


# admin 控制台密码, 不配置则不允许访问控制台
# ADMIN_PASSWORDS = ["123", "456"]


# 是否允许用户创建邮件, 不配置则不允许
ENABLE_USER_CREATE_EMAIL = true
# 允许用户删除邮件, 不配置则不允许
ENABLE_USER_DELETE_EMAIL = true


# D1 数据库的名称和 ID 可以在cloudflare控制台查看
[[d1_databases]]
binding = "DB"
database_name = "xxx" # D1 数据库名称
database_id = "xxx" # D1 数据库 ID


# kv config 用于用户注册发送邮件验证码，如果不启用用户注册或不启用注册验证，可以不配置，启用webhook也需要开启
# [[kv_namespaces]]
# binding = "KV"
# id = "xxxx"




```


#### 3.pages前端`secrets`
|名称|说明|
|:---|:---|
|`FRONTEND_ENV`|参考[官方仓库](https://github.com/dreamhunter2333/cloudflare_temp_email/blob/main/frontend/.env.example)|
|`FRONTEND_NAME`|你在 Cloudflare Pages 创建的项目名称，可通过用户界面或者命令行创建(**提前创建好的**)|
|`FRONTEND_BRANCH`|(可选) pages 部署的分支，可不配置，默认 production|


```
VITE_API_BASE=https://temp-email-api.xxx.xxx #对应你的后端（即worker）的URL，只需要改这个就行
VITE_CF_WEB_ANALY_TOKEN=
VITE_IS_TELEGRAM=false
```
### 部署
- 打开仓库的`Actions`页面
- 找到`Deploy Backend`,点击`Run workflow`选择分支手动部署
- 找到`Deploy Frontend`,点击`Run workflow`选择分支手动部署


### 配置路由转发
参考[官方文档](https://temp-mail-docs.awsl.uk/zh/guide/email-routing.html),官方文档非常清楚，所以就不多介绍了


### 综上
如果中间没有错误，那部署完之后就能正常使用了。


## 如需自动更新
- 打开仓库的`Actions`页面，找到`Upstream Sync`点击`enable workflow`启用`workflow`
- 如果`Upstream Sync`运行失败，到仓库主页点击`Sync`手动同步即可


