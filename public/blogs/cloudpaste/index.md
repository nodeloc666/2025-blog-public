本期介绍是一个非常强大的在线剪切板项目，不仅可以部署零成本，而且支持markdown，S3存储，docker自部署，WEBDAV等等强大功能。下面让我们一起看看吧！


##### 在开始之前，首先感谢[NETJETT](https://netjett.com/aff.php?aff=72)提供的**云服务器技术支持**，还有[ling-drag0n](https://github.com/ling-drag0n)大佬对该项目的持续维护。




# 部署
## 前提条件
1. Github账号
2. Cloudflare账号


## 教程
### 1. 配置 GitHub 仓库


1. Fork 或克隆仓库 [https://github.com/ling-drag0n/CloudPaste](https://github.com/ling-drag0n/CloudPaste)
2. 进入您的 GitHub 仓库设置
3. 转到 Settings → Secrets and variables → Actions → New Repository secrets
4. 添加以下 Secrets：


| Secret 名称             | 必需 | 用途                                                  |
| ----------------------- | ---- | ----------------------------------------------------- |
| `CLOUDFLARE_API_TOKEN`  | ✅   | Cloudflare API 令牌（需要 Workers、D1 和 Pages 权限） |
| `CLOUDFLARE_ACCOUNT_ID` | ✅   | Cloudflare 账户 ID                                    |
| `ENCRYPTION_SECRET`     | ❌   | 用于加密敏感数据的密钥（如不提供，将自动生成）        |


#### 获取 Cloudflare API 令牌


1. 访问 [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. 创建新的 API 令牌
3. 选择"编辑 Cloudflare Workers"模板，并添加 **D1 数据库**编辑权限
![D1.png](https://img.104226.xyz/file/blog/1752052651192_D1.png)
#### 获取 CLOUDFLARE_ACCOUNT_ID
这个在worker界面上面有，直接复制即可


### 2. 开始自动化部署
1. Fork 仓库，填好密钥，然后Actions,依次部署后端和前端。


![image.png](https://img.104226.xyz/file/blog/1752053030159_image.png)




2. **部署成功后**，登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
3. 导航到 Pages → 您的项目（如 "cloudpaste-frontend"）
4. 点击 "Settings" → "Environment variables"
5. 添加环境变量：


   - 名称：`VITE_BACKEND_URL`
   - 值：刚刚部署的后端 Worker URL（如 `https://cloudpaste-backend.your-username.workers.dev`），末尾不带"/", 同时建议使用自定义的 worker 后端域名。


   - **<span style="color:red">一定要完整的填写后端域名,"https://xxxx.com" 格式</span>**
![test-1.png](https://img.104226.xyz/file/blog/1752056780144_test-1.png)
6. **重要步骤： 随后要再次运行一遍前端的工作流，以便完成后端域名加载！！！**


7. 结束，访问前端（**用户名: admin, 密码: admin123**），然后修改默认管理员密码。


# 使用
1. 文本内容将会直接存储在D1数据库，着我们无需关心


2. 文件，则需绑定S3存储桶，项目作者已经提供了非常详细的说明了，请参阅[CloudPaste](https://github.com/ling-drag0n/CloudPaste)，我就不再赘述了。


3. 其他扩展功能直接看大佬对项目的介绍


