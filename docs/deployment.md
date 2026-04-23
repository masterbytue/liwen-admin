# 部署指南

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                    砺文工作室系统架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  前端网站              管理后台              后端 API        │
│  liwen-studio.        liwen-admin.         liwen-backend.   │
│  pages.dev            pages.dev            workers.dev      │
│                                                             │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐     │
│  │ 用户展示  │        │ 管理界面  │        │ 数据接口  │     │
│  │ 服务浏览  │◄──────►│ 登录认证  │◄──────►│ JWT验证   │     │
│  │ 留言提交  │        │ 内容管理  │        │ CRUD操作  │     │
│  └──────────┘        └──────────┘        └──────────┘     │
│       │                    │                    │           │
│       └────────────────────┴────────────────────┘           │
│                            │                                │
│                    ┌───────▼────────┐                       │
│                    │ Cloudflare D1  │                       │
│                    │ (SQLite数据库)  │                       │
│                    └────────────────┘                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 部署步骤

### 1. 后端 API 部署

**前提条件：**
- Cloudflare 账号
- Wrangler CLI 已安装

**部署命令：**
```bash
cd backend-worker
npm install
npm run deploy
```

**环境变量配置：**
在 Cloudflare Dashboard → Workers → liwen-backend → Settings → Variables 中设置：

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| JWT_SECRET | JWT签名密钥 | 随机字符串 |
| ADMIN_USERNAME | 管理员用户名 | admin |
| ADMIN_PASSWORD_HASH | 密码哈希值 | sha256(salt+password) |

### 2. 管理后台部署

**方式一：通过 Git 部署（推荐）**

1. 在 GitHub 创建仓库 `liwen-admin`
2. 推送代码：
```bash
cd admin-panel
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/YOUR_USERNAME/liwen-admin.git
git push -u origin main
```

3. 在 Cloudflare Dashboard：
   - Pages → Create a project
   - Connect to Git → 选择 `liwen-admin`
   - Framework preset: **None**
   - Build command: 留空
   - Build output directory: `/`
   - 点击 Save and Deploy

**方式二：直接上传**

1. Pages → Create a project → Upload assets
2. 拖拽 `admin-panel` 文件夹内容
3. 点击 Deploy

### 3. 前端网站部署

与管理后台相同步骤，仓库名为 `liwen-studio`。

## 配置检查清单

### 后端 API
- [ ] D1 数据库已创建并绑定
- [ ] 环境变量已设置
- [ ] CORS 配置包含前端域名
- [ ] JWT 密钥已更新（生产环境）

### 管理后台
- [ ] API_BASE 指向正确的后端地址
- [ ] 已部署到 Cloudflare Pages
- [ ] 自定义域名（可选）

### 前端网站
- [ ] API 端点指向正确的后端地址
- [ ] 已部署到 Cloudflare Pages
- [ ] 自定义域名（可选）

## 域名配置（可选）

在 Cloudflare Dashboard → Pages → 项目 → Custom domains 中添加：

| 项目 | 建议域名 |
|------|----------|
| 前端网站 | www.liwen-studio.com |
| 管理后台 | admin.liwen-studio.com |

## 安全建议

1. **修改默认密码**
   - 首次登录后立即修改管理员密码
   - 使用强密码（12位以上，包含大小写+数字+符号）

2. **更新 JWT 密钥**
   - 生产环境使用随机生成的复杂字符串
   - 定期更换密钥

3. **启用 HTTPS**
   - Cloudflare Pages 默认启用
   - 确保所有通信通过 HTTPS

4. **访问控制**
   - 管理后台不要公开传播地址
   - 考虑添加 IP 白名单（企业版功能）

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 登录失败 | 密码错误 | 检查 ADMIN_PASSWORD_HASH |
| 接口 401 | Token 过期 | 重新登录 |
| 接口 403 | CORS 问题 | 检查后端 CORS 配置 |
| 数据不显示 | 数据库连接失败 | 检查 D1 绑定 |
| 页面空白 | JS 错误 | 检查浏览器控制台 |

## 更新部署

**后端更新：**
```bash
cd backend-worker
npm run deploy
```

**管理后台更新：**
```bash
cd admin-panel
git add .
git commit -m "Update"
git push
```

Cloudflare Pages 会自动重新部署。
