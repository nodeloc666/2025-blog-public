# 前言
总有很多项目支持通过webhook进行转发消息，市面常有的server酱，wxpusher等等。今天介绍的[moepush](https://github.com/beilunyang/moepush)是完全基于cloudflare，完全免费且支持多种平台，如企业微信机器人、钉钉、飞书、等等。


# 教程（Github Actions部署）


## 前提
- cloudflare账号
- 托管在cloudflare的域名
- GitHub账号


## 步骤
### Fork 仓库并启用 Actions
1. 在 GitHub fork [仓库](https://github.com/beilunyang/moepush)
2. 打开仓库的 `Actions` 页面
3. 找到`Deploy`点击`enable workflow` 启用 workflow


### 配置Secrets
在仓库页面 `Settings` -> `Secrets and variables` -> `Actions` -> `Repository secrets`, 添加以下`secrets`:


|名称|说明|
|:---|:---|
|`CLOUDFLARE_ACCOUNT_ID`|Cloudflare Account ID|
|`CLOUDFLARE_API_TOKEN`|Cloudflare API Token|
|`D1_DATABASE_NAME`|D1 数据库名称|
|`AUTH_SECRET`|加密 Session 的密钥|
|`AUTH_GITHUB_ID`|GitHub OAuth App ID|
|`AUTH_GITHUB_SECRET`|GitHub OAuth App Secret|
|`PROJECT_NAME`|(可选)项目名称,默认`moepush`|
|`DISABLE_REGISTER`|是否禁止注册，默认关闭，设置为 `true` 则禁止注册|


### Github OAuth 配置
1. 登录 [Github Developer](https://github.com/settings/developers) 创建一个新的 OAuth App
2. 生成一个新的 `Client ID` 和 `Client Secret`
3. 设置 `Application name` 为 `<your-app-name>`
4. 设置 `Homepage URL` 为 `https://<your-domain>`
5. 设置 `Authorization callback URL` 为 `https://<your-domain>/api/auth/callback/github`




### 部署


- 打开仓库的`Actions`页面
- 找到`Deploy`,点击`Run workflow`选择分支手动部署


# 总结
1. 之前有bug，即使关闭注册，依然能通过GitHub直接授权登录使用，所以如果仅个人使用，可以先**开启注册**，然后自己手动注册一个账号，然后在部署一遍**关闭注册**，这样就能仅你一个人使用了
2. 仅个人使用使用时，`GitHub OAuth`可以不设置，如果报错的话，就随便填一下就行


3. 通过Actiuons部署之后，有更新后可以直接在GitHub上同步，然后cloudflare就会同步更新，方便了很多。


