# 职途规划系统（Career Planning System）

基于 Go（go-zero 框架）的大学生职业规划智能体，前端使用 React + TypeScript，支持 DeepSeek AI 能力。

---

# 🚀 新手部署指南（手把手）

> 本指南面向完全不会编程的新手，每一步都有详细说明，请按顺序操作。

## 目录

- [一、需要安装哪些软件？](#一需要安装哪些软件)
- [二、获取 DeepSeek API Key](#二获取-deepseek-api-key)
- [三、下载项目代码](#三下载项目代码)
- [四、配置数据库（MySQL）](#四配置数据库mysql)
- [五、配置 Redis](#五配置-redis)
- [六、创建后端配置文件](#六创建后端配置文件)
- [七、启动后端服务](#七启动后端服务)
- [八、启动前端服务](#八启动前端服务)
- [九、验证是否成功](#九验证是否成功)
- [十、生产环境部署（可选）](#十生产环境部署可选)
- [常见问题排查](#常见问题排查)

---

## 一、需要安装哪些软件？

在开始之前，你需要在电脑上安装以下软件。每个软件后面都有官方下载地址，点击链接下载安装包，一路点"下一步"即可。

| 软件 | 用途 | 下载地址 |
|------|------|----------|
| **Go 1.25+** | 运行后端代码（`go.mod` 要求 1.25.0） | https://go.dev/dl/ |
| **Node.js 20.19+** | 运行前端代码（Vite 8 需要） | https://nodejs.org/zh-cn/ |
| **MySQL 8.0** | 数据库 | https://dev.mysql.com/downloads/installer/ |
| **Redis 7.0** | 缓存服务 | https://redis.io/download/ （Windows 用 https://github.com/tporadowski/redis/releases） |
| **Git** | 下载代码 | https://git-scm.com/downloads |
| **yarn** | 前端包管理工具（安装 Node.js 后执行） | 见下方说明 |

### 安装 yarn

打开命令行（Windows 按 `Win+R` 输入 `cmd`，Mac 打开"终端"），输入：

```bash
npm install -g yarn
```

按回车，等待安装完成。

### 验证安装是否成功

逐一输入以下命令，每行都应显示版本号而不是报错：

```bash
go version
node --version
npm --version
yarn --version
git --version
mysql --version
redis-cli --version
```

---

## 二、获取 DeepSeek API Key

本项目使用 DeepSeek AI 提供智能分析功能，你需要一个 API Key（免费注册即可获得）。

1. 打开浏览器，访问：https://platform.deepseek.com/
2. 点击右上角「注册」，用手机号或邮箱注册账号
3. 登录后，点击左侧菜单「API Keys」
4. 点击「创建 API Key」，给它起个名字（如：career-api），点击确认
5. **复制生成的 API Key**（它只会显示一次，请保存好！格式类似：`sk-xxxxxxxxxxxxxxxxxxxx`）

---

## 三、下载项目代码

打开命令行，切换到你想存放项目的目录（例如桌面），然后执行：

```bash
# 进入桌面（可选，也可以换成其他目录）
cd ~/Desktop

# 下载项目代码
git clone https://github.com/ACKADE/Job-manner-review-system.git

# 进入项目根目录
cd Job-manner-review-system
```

---

## 四、配置数据库（MySQL）

### 4.1 启动 MySQL 服务

**Windows：**
- 按 `Win+R`，输入 `services.msc`，找到 `MySQL80`，右键点击「启动」

**Mac/Linux：**
```bash
# Mac (Homebrew 安装)
brew services start mysql

# Linux
sudo systemctl start mysql
```

### 4.2 登录 MySQL 并创建数据库

打开命令行，输入（将 `your_password` 替换为你安装 MySQL 时设置的密码）：

```bash
mysql -u root -p
```

回车后会提示输入密码，输入密码后按回车（输入时看不到字符，这是正常的）。

登录成功后，依次输入以下命令（每行按回车）：

```sql
-- 创建数据库（名字叫 career_db）
CREATE DATABASE career_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 选择刚创建的数据库
USE career_db;

-- 查看是否切换成功（应该显示 Database changed）
```

### 4.3 导入数据表结构

退出 MySQL（输入 `exit` 按回车），然后在命令行中执行：

```bash
# 确保你在项目根目录下
# 将 your_password 替换为你的 MySQL root 密码
mysql -u root -p career_db < sql/schema.sql
```

输入密码后按回车，没有报错即表示成功。

### 4.4 验证表是否创建成功

重新登录 MySQL 并查看表：

```bash
mysql -u root -p
```

```sql
USE career_db;
SHOW TABLES;
```

应该能看到 `users`、`jobs`、`students` 等表名。

---

## 五、配置 Redis

### 5.1 启动 Redis

**Windows：**
找到 Redis 安装目录，双击 `redis-server.exe` 启动，看到 Redis 图标即表示运行中。

**Mac/Linux：**
```bash
# Mac
brew services start redis

# Linux
sudo systemctl start redis
```

### 5.2 验证 Redis 是否运行

```bash
redis-cli ping
```

如果返回 `PONG`，说明 Redis 运行正常。

---

## 六、创建后端配置文件

在项目根目录下，创建 `etc` 文件夹和配置文件：

**Windows 命令行：**
```bash
mkdir etc
```

**Mac/Linux 终端：**
```bash
mkdir -p etc
```

然后在 `etc` 文件夹内创建一个名为 `career-api.yaml` 的文件，内容如下（注意修改其中的密码和 API Key）：

```yaml
Name: career-api
Host: 0.0.0.0
Port: 8088
Mode: dev

Timeout: 120000

Mysql:
  # 格式：用户名:密码@tcp(地址:端口)/数据库名?参数
  # 将 your_password 替换为你的 MySQL root 密码
  DataSource: root:your_password@tcp(localhost:3306)/career_db?charset=utf8mb4&parseTime=true&loc=Local
  # 必填：最大打开连接数
  MaxOpenConns: 100
  # 必填：最大空闲连接数
  MaxIdleConns: 10
  # 必填：连接最大生命周期（秒）
  ConnMaxLifetime: 3600

Redis:
  Host: localhost:6379
  Type: node

CacheRedis:
  - Host: localhost:6379
    Type: node

Auth:
  # 这是用于生成登录 Token 的密钥，可以随意填写一串字符，但要保密
  AccessSecret: change-this-to-a-random-secret-string-abcdef123456
  AccessExpire: 86400

AI:
  Provider: deepseek
  # 将下面的内容替换为你在第二步获取的 DeepSeek API Key
  ApiKey: sk-your-deepseek-api-key-here
  Model: deepseek-chat
  BaseURL: https://api.deepseek.com/v1
  Timeout: 60

RateLimit:
  TokensPerSecond: 100
  Burst: 200
```

> ⚠️ **注意事项：**
> - `your_password` → 改成你 MySQL 的 root 密码
> - `change-this-to-a-random-secret-string-abcdef123456` → 改成任意一串随机字符（越复杂越好）
> - `sk-your-deepseek-api-key-here` → 改成你的 DeepSeek API Key

---

## 七、启动后端服务

### 7.1 安装 Go 依赖

确保你在项目**根目录**下，执行：

```bash
go mod tidy
```

等待依赖下载完成（根据网速可能需要几分钟）。

> 💡 如果下载很慢，可以先设置国内镜像加速：
> ```bash
> go env -w GOPROXY=https://goproxy.cn,direct
> ```
> 然后再执行 `go mod tidy`

### 7.2 启动服务

```bash
go run career.go -f etc/career-api.yaml
```

看到类似下面的输出，表示后端启动成功：

```
Starting server at 0.0.0.0:8088...
```

> 💡 你也可以使用项目提供的脚本启动（仅限 Mac/Linux）：
> ```bash
> bash start.sh
> ```

**后端服务运行在：http://localhost:8088**

保持这个命令行窗口开着，不要关闭！

---

## 八、启动前端服务

**重新打开一个新的命令行窗口**，进入前端目录：

```bash
# 进入前端目录（从项目根目录开始）
cd high-school-worker-design-forend
```

### 8.1 安装前端依赖

```bash
yarn install
```

等待安装完成（根据网速可能需要几分钟）。

> 💡 如果 yarn 下载慢，可以先设置国内镜像：
> ```bash
> yarn config set registry https://registry.npmmirror.com
> ```
> 然后再执行 `yarn install`

### 8.2 启动前端开发服务器

```bash
yarn dev
```

看到类似下面的输出，表示前端启动成功：

```
  VITE v8.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://xxx.xxx.xxx.xxx:5173/
```

**前端界面访问地址：http://localhost:5173**

保持这个命令行窗口开着，不要关闭！

---

## 九、验证是否成功

### 9.1 测试后端接口

打开浏览器，访问：

```
http://localhost:8088/api/v1/health
```

如果看到：
```json
{"status":"ok","version":"1.0.0"}
```

说明后端运行正常 ✅

### 9.2 测试前端页面

打开浏览器，访问：

```
http://localhost:5173
```

看到登录/注册页面即表示成功 ✅

### 9.3 注册第一个账号

在前端页面点击「注册」，填写用户名、密码和邮箱，注册成功后即可登录使用系统。

---

## 十、生产环境部署（可选）

> 如果你只是想本地体验，第九步完成后即可正常使用，不需要做这一步。
> 如果你想让其他人也能通过互联网访问，才需要部署到服务器。

详细的生产环境部署步骤请参考：[docs/生产环境部署指南.md](docs/生产环境部署指南.md)

简要步骤：

1. **购买云服务器**：如阿里云、腾讯云、华为云等，选择 Ubuntu 22.04 系统
2. **在服务器上安装所有软件**：Go、Node.js、MySQL、Redis、Nginx（参考上方"安装软件"章节）
3. **将代码上传到服务器**：`git clone` 或者用 FTP 工具上传
4. **编译前端**（在服务器上，进入 `high-school-worker-design-forend` 目录）：
   ```bash
   yarn install
   yarn build
   ```
   打包后的文件在 `dist/` 目录
5. **编译后端**（在项目根目录）：
   ```bash
   go build -o career-api career.go
   ```
6. **配置 Nginx** 将前端静态文件和后端 API 反向代理到同一个域名，参考 [docs/生产环境部署指南.md](docs/生产环境部署指南.md)
7. **使用 systemd 让后端服务开机自启**，参考 [docs/生产环境部署指南.md](docs/生产环境部署指南.md)

---

## 常见问题排查

### ❓ 问题：`go mod tidy` 下载超时或失败

**解决方法**：设置国内镜像再重试
```bash
go env -w GOPROXY=https://goproxy.cn,direct
go mod tidy
```

---

### ❓ 问题：执行 `go mod tidy` 或 `go run` 提示 Go 版本过低

**原因**：项目的 `go.mod` 要求 `go 1.25.0`，本机 Go 版本太低。

**解决方法**：升级 Go 到 1.25 或更高版本后重试（可用 `go version` 检查）。

---

### ❓ 问题：启动后端时提示 `dial tcp 127.0.0.1:3306: connect: connection refused`

**原因**：MySQL 没有运行

**解决方法**：启动 MySQL 服务（参考第四步）

---

### ❓ 问题：启动后端时提示 `dial tcp 127.0.0.1:6379: connect: connection refused`

**原因**：Redis 没有运行

**解决方法**：启动 Redis 服务（参考第五步）

---

### ❓ 问题：启动后端时提示数据库密码错误

**原因**：配置文件中的 MySQL 密码不对

**解决方法**：检查并修改 `etc/career-api.yaml` 中 `DataSource` 里的密码

---

### ❓ 问题：启动后端时提示 `field "Mysql.MaxOpenConns" is not set`

**原因**：`etc/career-api.yaml` 的 `Mysql` 配置不完整，缺少必填连接池参数。

**解决方法**：在 `Mysql` 下补齐以下字段后重启：

```yaml
Mysql:
  DataSource: root:your_password@tcp(localhost:3306)/career_db?charset=utf8mb4&parseTime=true&loc=Local
  MaxOpenConns: 100
  MaxIdleConns: 10
  ConnMaxLifetime: 3600
```

---

### ❓ 问题：前端页面打开后，操作没有反应或显示"网络请求失败"

**原因**：后端服务没有启动，或者后端启动失败

**解决方法**：
1. 检查后端的命令行窗口是否有报错信息
2. 用浏览器访问 `http://localhost:8088/api/v1/health`，确认后端是否正常
3. 确保两个命令行窗口（前端和后端）都还开着

---

### ❓ 问题：端口 8088 或 5173 已被占用

**解决方法（Mac/Linux）**：
```bash
# 查找占用 8088 端口的进程
lsof -i :8088
# 杀掉该进程（将 PID 替换为上面查到的进程号）
kill -9 PID
```

**解决方法（Windows）**：
```bash
# 查找占用 8088 端口的进程
netstat -ano | findstr :8088
# 杀掉该进程（将 PID 替换为最后一列的数字）
taskkill /F /PID PID
```

---

### ❓ 问题：`yarn install` 安装依赖失败

**解决方法**：切换 npm 镜像源后重试
```bash
yarn config set registry https://registry.npmmirror.com
yarn install
```

---

## API 文档

后端启动后，主要接口如下（完整文档见 [docs/api.md](docs/api.md)）：

| 接口 | 说明 |
|------|------|
| `GET /api/v1/health` | 健康检查 |
| `POST /api/v1/user/register` | 用户注册 |
| `POST /api/v1/user/login` | 用户登录 |
| `GET /api/v1/jobs` | 获取岗位列表 |
| `POST /api/v1/jobs/generate` | AI 生成岗位画像 |
| `POST /api/v1/students/resume` | AI 解析简历 |
| `POST /api/v1/match` | 人岗匹配 |
| `POST /api/v1/reports/generate` | AI 生成职业规划报告 |

---

## 项目结构

```
Job-manner-review-system/
├── api/                          # API 定义文件
├── career.go                     # 后端程序入口
├── cmd/                          # 命令行工具
├── common/                       # 公共模块（错误码、中间件、AI Provider）
├── docs/                         # 项目文档
│   └── 生产环境部署指南.md         # 详细生产部署文档
├── etc/                          # 配置文件（需手动创建）
│   └── career-api.yaml           # 后端配置（需手动创建）
├── go.mod                        # Go 依赖清单
├── high-school-worker-design-forend/   # 前端代码（React + TypeScript）
│   ├── src/                      # 前端源码
│   ├── package.json              # 前端依赖清单
│   └── vite.config.ts            # 前端构建配置
├── internal/                     # 后端内部模块（处理器、业务逻辑等）
├── sql/                          # 数据库建表 SQL 文件
│   └── schema.sql                # 数据库表结构
└── start.sh                      # 快速启动脚本（Mac/Linux）
```

---

## 技术栈

| 层次 | 技术 |
|------|------|
| 后端框架 | Go + go-zero |
| 前端框架 | React 19 + TypeScript + Vite |
| UI 组件库 | Ant Design + Tailwind CSS |
| 数据库 | MySQL 8.0 |
| 缓存 | Redis 7 |
| AI 服务 | DeepSeek API |
