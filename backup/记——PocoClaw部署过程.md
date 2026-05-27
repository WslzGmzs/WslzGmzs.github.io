### 花了半天时间部署，把坑都踩了一遍

---

### 简单运行

```
git clone https://github.com/poco-ai/poco-agent.git  
cd poco-agent
```

```
./scripts/quickstart.sh
```

脚本会以交互式向导引导你完成配置，主要询问：

1. **语言选择**（中文/英文）
2. **部署模式**：本地开发 or 云端部署
3. **模型 API 类型**：Anthropic 官方 or 兼容 API（代理/第三方）
4. **API Key**：填入你的 `ANTHROPIC_API_KEY`
5. **API Base URL**：官方默认 `https://api.anthropic.com`，第三方填对应地址
6. **默认模型 ID**：如 `claude-sonnet-4-20250514`
7. **S3 公共端点**（本地开发可直接跳过）

脚本会自动：
- 从 `.env.example` 生成 `.env` 配置文件
- 拉取所需 Docker 镜像
- 启动所有服务 poco-agent:73-81 

**访问界面**

启动完成后，打开浏览器访问：

```
http://localhost:3000
```

---

### 这时候你可能遇到的问题

1. Linux 下 RustFS 权限问题：如果报 Permission denied，需要执行：
```
mkdir -p oss_data  
sudo chown -R 10001:10001 oss_data
```

2. 端口占用，修改.env文件带PORT的环境变量，比如
```
# -----------------------------------------------------------------------------
# Service Ports (Optional)
# -----------------------------------------------------------------------------
# FRONTEND_PORT=3000
# BACKEND_PORT=8000
# EXECUTOR_MANAGER_PORT=8001
# POSTGRES_PORT=5432
# S3_PORT=9000
# S3_CONSOLE_PORT=9001
```

3. backend 容器启动失败，使用下面命令查看报错
```
docker compose logs backend
```
- 可以发给AI看看，大概率是S3 bucket 未创建，backend 启动时会尝试连接 S3，如果 bucket 不存在会报错。需要先创建 bucket：

```
docker compose --profile init up rustfs-init
```
- 小概率`tmp_workspace`目录不存在，backend 挂载了 ./tmp_workspace，如果目录不存在会导致启动失败：

```
mkdir -p tmp_workspace oss_data  
sudo chown -R 10001:10001 oss_data
```

修好之后重新执行脚本即可，如果之前成功启动了部分组件，使用下面命令停止在重新执行脚本

```
docker compose down
```

4. 访问前端`http://localhost:3000`提示未设置OAuth登录

#### 两种处理方式
**方式一：跳过 OAuth（个人/内网单用户使用）**
在 .env 中添加：
```
AUTH_MODE=single_user
```
然后重启服务：
```
docker compose down && docker compose up -d
```
`single_user`模式下所有请求都会自动映射到 `SINGLE_USER_ID`（默认 `default`），无需登录，该用户自动拥有 admin 权限。`disabled`是 `single_user`的旧别名，效果相同。

四个 `AUTH_MODE` 值的对比：

| 值 | 行为 |
|---|---|
| `oauth_required` | 必须配置 OAuth provider 才能使用（默认值） |
| `oauth_optional` | 有 provider 则要求登录，没有则单用户模式 |
| `single_user` | 始终单用户模式，无需登录 |
| `disabled` | 同 `single_user`（旧别名） |

**方式二：配置 OAuth Provider（多用户/生产环境）**
系统支持三个 provider：**Google、GitHub、飞书/Lark**，至少配置一个即可。

**GitHub OAuth（最简单）**：
- 打开 https://github.com/settings/developers → New OAuth App
- Homepage URL 填： `http://<你的IP>:3000或者反代地址`
- Authorization callback URL填： `http://<你的IP或域名>:3000/api/v1/auth/github/callback`
- 点击 Register application，然后生成 Client Secret
- 在 .env 中填入：
```
GITHUB_CLIENT_ID=<your_client_id>  
GITHUB_CLIENT_SECRET=<your_client_secret>
```

**Google OAuth：**
- 打开 https://console.cloud.google.com/ → API & Services → Credentials → Create OAuth 2.0 Client
- 授权回调 URL 填： `http://<你的IP或域名>:3000/api/v1/auth/google/callback`
- 在 .env 中填入：
```
GOOGLE_CLIENT_ID=<your_client_id>  
GOOGLE_CLIENT_SECRET=<your_client_secret>
```

**飞书/Lark OAuth：**
回调地址填： `http://<你的IP或域名>:3000/api/v1/auth/feishu/callback`
```
FEISHU_OAUTH_CLIENT_ID=<cli_xxx>  
FEISHU_OAUTH_CLIENT_SECRET=<your_secret>  
FEISHU_OAUTH_REGION=cn   # 国内飞书用 cn，Lark 用 global
```

**值得注意的是回调地址需要填写的是`http://<你的IP或域名>:3000`即前端地址而不是后端地址，项目的流程是`浏览器 → Frontend (3000) → 代理 /api/v1/* → Backend (8000)`，`_frontend_url `用的是 `FRONTEND_PUBLIC_URL`（默认 `http://localhost:3000`）作为基底。
也就是说如果你使用了反向代理或者域名你必须修改`.env `中的 `FRONTEND_PUBLIC_URL `与你填入 GitHub 的 callback URL 的域名/端口一致，否则 `build_redirect_uri `生成的地址会对不上，导致 GitHub 拒绝回调。**

**这种情况下你需要修改的需要修改添加的环境变量**
假设你的域名是 `https://poco.example.com`，在 .env 中修改以下项：
```
FRONTEND_PUBLIC_URL=https://poco.example.com
CORS_ORIGINS=["https://poco.example.com"]
AUTH_COOKIE_SECURE=true
OAUTH_SESSION_COOKIE_SECURE=true
```

| 变量 | 为什么必须改 |
|---|---|
| `FRONTEND_PUBLIC_URL` | 后端用它拼接 OAuth 回调地址。不改的话，GitHub 授权后会把你重定向到 `http://localhost:3000/api/v1/auth/github/callback`，浏览器直接访问不通（见 [auth_service.py](backend/app/services/auth_service.py#L236-L237)） |
| `CORS_ORIGINS` | 后端校验跨域请求来源。不改的话，前端从 `https://poco.example.com` 发起的 API 请求会被后端拒绝（默认只允许 `http://localhost:3000`） |
| `AUTH_COOKIE_SECURE` | 用了 HTTPS 就必须设为 `true`，否则浏览器不会发送 Secure Cookie，登录态无效 |
| `OAUTH_SESSION_COOKIE_SECURE` | 同理，OAuth 流程中的状态 Cookie 也需要 Secure 标记 |

ps：不需要开放后端或者使用域名代理后端服务

#### 配置管理员账号
使用 OAuth 登录时，通过 SYSTEM_ADMIN_EMAILS 指定哪些邮箱登录后自动获得 admin 权限：
···
SYSTEM_ADMIN_EMAILS=you@example.com
···

5. 生产环境变量需要修改的设置
```
BACKEND_SECRET_KEY=change-this-secret-key-in-production
INTERNAL_API_TOKEN=change-this-token-in-production
CALLBACK_TOKEN=change-this-token-in-production
```

使用下面命令生成即可
```
openssl rand -hex 32
```

`POSTGRES_PASSWORD`没有开放公网酌情修改，脚本部署的时候把数据库端口映射到了宿主机，需要注意docker穿透ufw的风险，有这方面问题的可以考虑使用[ufw-docker](https://github.com/chaifeng/ufw-docker#%E8%A7%A3%E5%86%B3-ufw-%E5%92%8C-docker-%E7%9A%84%E9%97%AE%E9%A2%98)解决

因为没有热更新，任何修改了.env需要生效都需要在项目执行下面命令
```
docker compose down && docker compose up -d
```

---

### 开启mem0
```
# 1. 在 .env 中设置（更多的也在这里配置）
MEM0_ENABLED=true
 
# 2. 用 mem0 profile 启动（部署过了记得先down）
docker compose --profile mem0 up -d
```

---

### 更新镜像

1. 拉取最新镜像  

```
docker compose pull  
```

2. 重启服务（滚动更新：直接替换正在运行的容器为新镜像） 

```
docker compose up -d
```

---

详细的部署文档和问题排查，请参考 [部署指南](https://docs.poco-ai.com/zh/deployment)。