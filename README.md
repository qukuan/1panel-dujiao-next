# 1Panel 面板部署 Dujiao-Next 教程

## 📝 前言
- **旧版独角数卡**已经停止更新维护，作者旧仓库：`https://github.com/assimon/dujiaoka`
- 作者推出了**新版 Dujiao-Next**，新仓库：`https://github.com/dujiao-next`
- 新版独角由三个核心部分组成：
`user 用户前端`
`admin 管理前端` 
`api后端`

#### 注意⚠️
本篇是1Panel面板部署独角Dujiao-Next教程，和宝塔不一样，不要弄混了！

宝塔Docker部署教程移步到点击移步到仓库：[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&style=for-the-badge)](https://github.com/qukuan/1panel-dujiao-next)

---

#### 📷 视频教程（保姆级）一看就会

[![Bilibili](https://img.shields.io/badge/Bilibili-国内网点击观看blibli视频教程-blue?style=for-the-badge&logo=bilibili)](https://www.bilibili.com/video/BV1dMP3znE1i/)

[![YouTube Video](https://img.youtube.com/vi/Mv946RQJmrY/maxresdefault.jpg)](https://www.youtube.com/watch?v=Mv946RQJmrY)

---

## ❓ 常见问题 (FAQ)

- **问**：旧版独角数卡能直接升级到新版本吗？
- **答**：不能。旧版独角是基于 PHP 语言开发的，而新版 `Dujiao-Next` 是使用 Go 语言完全重构的。

- **问**：那我旧版独角的数据、卡密可以迁移到新版独角吗？
- **答**：可以。社区提供了一个数据迁移插件，具体操作请参考仓库地址：`https://github.com/luoyanglang/dujiao-migrate`，请自行阅读文档完成数据迁移。

---

## 🛠️ 准备环境

> 本教程以 **Linux 服务器 Ubuntu 24.04** 操作系统为例。

1. **准备 1Panel 面板**：提前安装好 1Panel 面板，或者使用您已经在使用的 1Panel 环境。
2. **安装数据库与缓存引擎**：新版推荐的生产环境为 `Redis` + `PostgreSQL`。我这里早早安装好了这两个组件。
   - 如果您是新安装的 1Panel 面板，请前往 **菜单栏 👉 应用商店 👉 搜索 Redis 和 PostgreSQL** 进行安装。
   - **注意：** 生产环境安装时，**不要**点击高级设置，**不要**打开端口的外部访问权限。只需将默认密码修改为您自己的密码即可。
3. **记录关键信息**：
   - 记录好您的 `Redis 密码`，稍后配置时需要使用。
   - 记录容器名称：如果不知道刚刚安装的 `Redis` 和 `PostgreSQL` 的容器名称，可以打开菜单栏点击 **容器** 查看，顺手复制下来备用。

---

## 🌐 添加三个站点 (反向代理配置)

我们需要分别为 `user前端`、`admin前端` 和 `API后端` 添加站点并设置反向代理。**个人建议先添加站点设置反代，然后再补充配置文件。**

### 1. 添加 User 前端站点
- **操作路径**：菜单栏 → 网站 → 网站 → 创建 → 反向代理
- **主域名**：填写您的 user 域名（例如 `user.faka.com`，不要带 https）
- **代理地址**：输入 `127.0.0.1:9005`（`9005` 为我想要的 user 前端监听端口号，可自行修改）
- **其他设置**：保持默认或不填 → 点击 **确认**

### 2. 添加 Admin 管理前端站点
- **操作路径**：菜单栏 → 网站 → 网站 → 创建 → 反向代理
- **主域名**：填写您的 admin 域名（例如 `admin.faka.com`，不要带 https）
- **代理地址**：输入 `127.0.0.1:9006`（`9006` 为我想要的 admin 管理前端监听端口号，可自行修改）
- **其他设置**：保持默认或不填 → 点击 **确认**

### 3. 添加 API 后端站点
- **操作路径**：菜单栏 → 网站 → 网站 → 创建 → 反向代理
- **主域名**：填写您的 API 域名（例如 `api.faka.com`，不要带 https）
- **代理地址**：输入 `127.0.0.1:9007`（`9007` 为我想要的 API 后端监听端口号，可自行修改）
- **其他设置**：保持默认或不填 → 点击 **确认**

> **配置信息补充：**
> 三个站点在添加完反向代理后，再分别去点击一下域名，点击：**反向代理 → 配置信息**。
> 参考相对应的配置信息复制粘贴补充进去（nginx文件夹里面的 `user.conf`  和  `admin.conf`），这两个文件里面都有注解，请自行参考。

---

## 📁 准备部署目录

您可以选择使用 SSH 连接终端执行命令，或者直接在 1Panel 面板中可视化创建。

### 方式一：终端命令创建
依次复制以下命令并回车执行：

```bash
mkdir -p /opt/dujiao-next/{config,data/db,data/uploads,data/logs,data/redis,data/postgres}
```
```bash
cd /opt/dujiao-next
```
```bash
chmod -R 0777 ./data/logs ./data/db ./data/uploads ./data/redis ./data/postgres 
```

### 方式二：在 1Panel 面板直接创建
1. 进入 `opt` 文件夹 → 创建文件夹 `dujiao-next`
2. 在 `dujiao-next` 文件夹内 继续创建以下文件夹：`config` 和 `data`
3. 进入 `data` 文件夹内，创建以下子目录：`logs`、`postgres`、`redis`、`uploads`
4. 返回根目录（即 `dujiao-next`），全选 `config` 和 `data` 文件夹 → 点击 **权限** → 设置为 `777`

---

## 🗄️ 创建 PostgreSQL 数据库

- **操作路径**：面板菜单栏 → 数据库 → 点击 PostgreSQL 切换 → 添加数据库
- 设置好**数据库名称**、**数据库用户**、**数据库密码**，设置好并记下来一会要用。

---

## ⚙️ 填写配置信息

按照前面复制记下来的：Redis 密码、Redis 容器名称、PostgreSQL 容器名称、数据库名称、数据库用户、数据库密码。

正确按照你自己配置信息来填写 `.env` 文件和 `config.yml` 文件（可以看文件内的注解）。填写完成后，上传到相对应的目录下：

- **根目录存放文件**：`.env` 文件 和 `docker-compose.postgres.yml` 文件。这两个放在根目录下（也就是 `dujiao-next` 文件夹内）。
- **config目录存放文件**：`config.yml` 文件。点击进入 `dujiao-next/config` 子目录，上传 `config.yml`。

---

## 🚀 启动与运行

确保当前在 `/opt/dujiao-next` 目录下，执行启动命令：

```bash
docker compose --env-file .env -f docker-compose.postgres.yml up -d
```

### 查看容器是否启动成功：
```bash
docker compose --env-file .env -f docker-compose.postgres.yml ps
```

### 查看 API 服务端日志（重要）：
很多时候前端成功了，但 API 服务端没有启动成功（通常是 Redis 或者 PostgreSQL 配置不对，导致 API 服务没有连接上）。继续查看一下 API 服务端日志：
```bash
docker compose --env-file .env -f docker-compose.postgres.yml logs -f api
```

**⚠️ API 服务端没有启动成功你可能会遇到：**
- 后台输入账号密码登录时提示：`限流服务不可用`
- 前端页面都能打开，登录却有等等问题，基本上都是 API 服务端没启动成功。

### 配置不对导致启动失败怎么办？
清理已产生的旧容器：
```bash
docker compose --env-file .env -f docker-compose.postgres.yml down
```
重新修改好配置文件后，再次执行启动命令即可！
