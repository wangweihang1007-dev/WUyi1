# 校园答疑助手 RAG 系统部署指南

## 一、项目说明

系统由两个独立端口组成：

| 端口 | 地址 | 用途 |
|---|---|---|
| 3000 | `http://localhost:3000/` | 用户端智能问答 |
| 3001 | `http://localhost:3001/` | 管理员后台 |

用户端不显示知识库管理入口，管理员需要通过独立端口进入后台。

## 二、运行环境

- Node.js 18 或更高版本
- 智谱开放平台 API Key
- Windows、Linux 或 macOS 均可运行

检查 Node.js：

```bash
node -v
npm -v
```

## 三、本地启动

进入项目目录：

```bash
cd WUyi
```

安装项目依赖：

```bash
npm install
```

配置智谱 API Key。

Windows PowerShell：

```powershell
$env:ZHIPU_API_KEY="你的智谱API Key"
npm start
```

Windows CMD：

```cmd
set ZHIPU_API_KEY=你的智谱API Key
npm start
```

Linux / macOS：

```bash
export ZHIPU_API_KEY="你的智谱API Key"
npm start
```

启动成功后会看到：

```text
User app running at http://localhost:3000
Admin app running at http://localhost:3001
```

## 四、访问系统

用户端：

```text
http://localhost:3000/
```

管理员端：

```text
http://localhost:3001/
```

默认管理员账号：

```text
账号：admin
密码：wwhyyds
```

首次登录后，可以在管理后台配置：

- 学校名称
- 助手名称
- 助手副标题
- 咨询电话
- 首页简介
- 智谱 API Key
- 使用模型
- 学长微信二维码地址

## 五、知识库使用

1. 打开管理员端。
2. 使用默认账号登录。
3. 在“知识库资料”区域选择文件。
4. 点击“上传并索引”。
5. 上传成功后，用户端提问会自动进行知识库检索。

当前可自动解析并建立索引的格式包括：

- TXT
- Markdown
- JSON
- CSV
- HTML
- XML
- YAML
- JavaScript / TypeScript
- CSS
- SQL
- 日志文件

其他格式会保存原文件，但如果没有对应解析器，不会自动生成文本索引。

## 六、数据目录

运行后会生成：

```text
data/
├─ knowledge.json     # 文档索引和文本分块
└─ uploads/           # 上传的原始文件
```

系统配置保存在：

```text
config.json
```

生产环境部署时，需要备份以下内容：

- `config.json`
- `data/knowledge.json`
- `data/uploads/`

## 七、自定义端口

默认端口是用户端 3000、管理员端 3001。

PowerShell：

```powershell
$env:PORT="8080"
$env:ADMIN_PORT="8081"
$env:ZHIPU_API_KEY="你的智谱API Key"
npm start
```

访问地址变为：

```text
用户端：http://localhost:8080/
管理员端：http://localhost:8081/
```

## 八、生产环境运行

建议使用 PM2 管理进程：

```bash
npm install -g pm2
pm2 start server.js --name campus-rag
pm2 save
pm2 startup
```

启动前先配置环境变量：

```bash
export ZHIPU_API_KEY="你的智谱API Key"
export PORT=3000
export ADMIN_PORT=3001
```

查看运行状态：

```bash
pm2 status
pm2 logs campus-rag
```

## 九、域名和反向代理

建议将用户端和管理员端使用不同域名：

```text
用户端：https://assistant.example.com
管理员端：https://admin-assistant.example.com
```

Nginx 示例：

```nginx
server {
    listen 80;
    server_name assistant.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 80;
    server_name admin-assistant.example.com;

    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## 十、安全建议

生产环境必须完成以下事项：

- 修改默认管理员账号和密码
- 使用 HTTPS
- 不要把 API Key 提交到 Git 仓库
- 限制管理员端 IP 或增加 VPN 访问控制
- 限制上传文件大小和文件类型
- 定期备份 `config.json` 和 `data/`
- 使用更复杂的管理员认证方式
- 定期更换智谱 API Key

当前默认管理员账号是演示配置，不建议直接用于公网生产环境。

## 十一、健康检查

用户端服务健康检查：

```text
http://localhost:3000/api/health
```

返回示例：

```json
{
  "ok": true,
  "model": "glm-4-flash",
  "configured": true,
  "documents": 3,
  "chunks": 28
}
```

如果 `configured` 为 `false`，说明服务端还没有配置智谱 API Key。

## 十二、常见问题

### 1. 用户端无法回答

检查：

- `ZHIPU_API_KEY` 是否正确
- 服务是否正在运行
- `/api/health` 的 `configured` 是否为 `true`
- 服务器是否可以访问智谱 API

### 2. 管理员端打不开

检查 3001 端口是否被占用，并确认启动日志中存在：

```text
Admin app running at http://localhost:3001
```

### 3. 上传后没有检索结果

确认：

- 文件格式属于可解析格式
- 文档内容不是空文件
- 管理后台显示“已索引”
- 用户问题和资料内容存在相关关键词

### 4. 二维码不显示

检查二维码地址是否是浏览器可访问的图片 URL。建议使用 HTTPS 图片地址，或将图片放在项目可访问目录中。
