# n8n 部署与工作流使用指南

## 一、Skill 篇

### 1. 资源库 Skill

### 2. 导演 Skill

### 3. 分镜师 Skill

---

## 二、部署篇

### 0. 部署前说明

部署完成后，你将获得：

- 本机运行的 n8n 服务；
- 无需另外安装 Node.js、npm 或 FFmpeg；
- 工作流和凭据保存在 Docker 数据卷中；
- 关闭浏览器不会停止 n8n；
- 退出或关闭 Docker Desktop 会使 n8n 停止；
- 默认仅供本机访问，不直接暴露到公网。

> **注意：** Docker Desktop、Docker 镜像和 Docker 容器不是同一个东西。

### 1. 部署包检查

在有空余空间的磁盘中创建一个简单的**英文目录**，例如：

```text
F:\n8n-deploy
```

将百度网盘中的以下两个文件放入该目录：

- `docker-compose.yml`
- `n8n-with-ffmpeg.tar`

**部署包下载：** [百度网盘链接](https://pan.baidu.com/s/1RnmpKwzhlplP9shJ0TvH7A?pwd=gusu)（提取码：`gusu`）

### 2. 安装或更新 WSL 2

以**管理员身份**打开 PowerShell，输入：

```powershell
wsl --install
```

安装完成后，建议重启电脑，再依次执行：

```powershell
wsl --update
wsl --version
wsl --status
```

### 3. 安装 Docker Desktop

**下载地址：** [Docker Desktop 官网](https://www.docker.com/products/docker-desktop/)

安装并启动 Docker Desktop 后，在 PowerShell 中执行以下命令进行验证：

```powershell
docker version
docker compose version
docker info --format '{{.OSType}}'
```

前两条命令应返回版本信息；最后一条在 Linux 容器模式下应返回 `linux`。

### 4. 导入 n8n 自定义镜像

在 PowerShell 中进入第 1 步创建的部署目录：

```powershell
cd F:\n8n-deploy
```

导入镜像：

```powershell
docker load --input .\n8n-with-ffmpeg.tar
```

正常输出通常类似：

```text
Loaded image: n8n-with-ffmpeg:版本号
```

然后执行：

```powershell
docker image ls
docker compose config --images
```

检查已导入的镜像及 Compose 文件引用的镜像名称是否对应。

### 5. 校验 Compose 配置

在部署目录中执行：

```powershell
docker compose config
```

确认配置可以正常解析、未出现报错。

### 6. 启动 n8n，注册账号并导入工作流

在部署目录中执行：

```powershell
docker compose up -d
```

待容器启动后，在浏览器中打开 Compose 配置映射的 n8n 地址（如配置使用默认端口，可尝试 `http://localhost:5678`）。

![n8n 初次访问界面](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmU2YWFmYmVhNjUxNWRiNzU1Nzg0MjhlOWNjMDBhZjRfb3RQTHI4NkwzV0pYWUpNZ3RRUXpHazNZMFBwc2tpMkZfVG9rZW46T1NmeWJGUFd5b3RmeWZ4OEpXVGNkc3g4bnV6XzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

1. **首次进入时，先使用邮箱创建 n8n 账号。**
2. 注册完成后，进入 **Workflows** 界面。
3. 选择 **Import from file**，导入工作流文件（`.json`）。

![工作流导入界面一](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=NjZiMWNiNmRkMThlNWRjZjRiZmY0ZGRhNmNiYWQ4OTBfNlZHMVg2ckZmM2puMWxVa1E4RFBXQTVkNHg0WWEyZkdfVG9rZW46T1VYSGI2Vk1Eb3JEYlh4WUZZcWNZcjVpbnFnXzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

![工作流导入界面二](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTBkODc4Mjk0ZmU3ZThkMTlkZjg4MzJlZjM5YTE3MzRfc3JNUEFhUEtwbXhPTjVBVldUQzZNZHJ2V1BiQVFwcmhfVG9rZW46SkduQmJ5TEZKb2Y1Zkp4VmxZOWMyVFJJbkJnXzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

---

## 三、工作流篇

### 1. COSBrowser 软件下载

**下载地址：** [腾讯云 COSBrowser](https://cosbrowser.cloud.tencent.com/)

登录时，联系资源管理员获取所需的 `SecretId` 和 `SecretKey`。请通过安全渠道传递凭据，不要将其写入公开仓库或工作流分享文件中。

### 2. 工作流执行思路

#### 2.1 分镜脚本撰写（默认使用 Codex 的 Skill 撰写）

分镜脚本 Excel 的列名如下：

| 镜头序号 | 时长（秒） | 景别 | 机位 | 运镜 | 出场人物 | 物料 | 场景 | 画面内容 | 人物台词 | 环境音效 | Section | 生图提示词 | 生视频提示词 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

#### 2.2 WF00：发布资源文件

**运行前务必在 COSBrowser 中确认：** 当前人物形象及妆造是否符合剧情要求。

![WF00 发布资源文件](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=M2U5YjJkMWNmMGI2OTNiOTM0MjJjYzQyZGE2MzY5MDRfZmtEQUdEREhsV1RSSmJXWERPOEt4aThKRXNQSWl0TnpfVG9rZW46WEhMN2JqT1Vvb1lNUXl4Nmo2bGNLWFE2bjZnXzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

#### 2.3 WF29：输入分镜脚本 Excel，使用 GPT-Image-2 批量生成每集四宫格分镜图

![WF29 批量生成四宫格分镜图](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=YzM4ZTU1NDhjMWNkMDZhODdkNWEyZGE2Y2E5YjJiMDVfdXhKN3pMeTBwcUlQMnhpSHkzQVhjblNMdzBlcXN0YnBfVG9rZW46TXQ3bmJmYU1xb1BObWZ4YVRQOGNzbW5zbkFoXzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

运行完成后，分镜图将存储到 COSBrowser 所连接的对象存储中。请逐一检查是否存在画面畸变、与剧情不符等问题，并针对问题修改分镜图和视频 Prompt。

#### 2.4 WF39：输入分镜脚本 Excel，使用 Seedance 2.0 批量生成分集视频

![WF39 批量生成分集视频](https://dcnqg293zdkn.feishu.cn/space/api/box/stream/download/asynccode/?code=NTJjNzA0YjNjZTdiY2RkNmQ5OTM3MDAzZjk4ZTgxMjBfblBCdW5MVUZJZkpRRVZKU1lRV2JmMGxDODRMMTBzUDVfVG9rZW46Rjd6U2JtQk1Cb3p3MGR4UHl0RGNRMHZLbm9mXzE3OTAxNTM3NDY6MTc5MDE1NzM0Nl9WNA&add_watermark=true&scene_type=CCM)

运行完成后，视频片段将存储到 COSBrowser 所连接的对象存储中。请逐一检查是否存在画面畸变、与剧情不符等问题，并针对问题修改分镜图和视频 Prompt。

> **图片说明：** 上述图片沿用原始飞书链接；如果链接失效或需要访问权限，请将图片下载后替换为本地相对路径。
