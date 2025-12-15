# 🐳 TrendRadar Docker 部署指南

## 目录
1. [快速开始](#快速开始)
2. [文件说明](#文件说明)
3. [容器启动](#容器启动)
4. [查看运行结果](#查看运行结果)
5. [常用命令](#常用命令)
6. [后续启用推送渠道](#后续启用推送渠道)

---

## 快速开始

### 前置条件
- ✅ 已安装 Docker 和 Docker Compose
- ✅ 项目目录结构完整

### 1️⃣ 启动容器

```bash
# 进入项目目录
cd /Users/runysun/program/TrendRadar

# 方式一：使用简易配置（推荐新手）
docker-compose -f docker-compose.simple.yml up -d

# 方式二：使用原始配置 + .env 文件
# 先复制 .env 文件：
# cp .env.docker .env
# 然后：
# docker-compose -f docker/docker-compose.yml up -d
```

### 2️⃣ 验证容器运行状态

```bash
# 查看容器状态
docker ps | grep trend-radar

# 查看容器日志
docker logs -f trend-radar
```

---

## 文件说明

### 📄 `.env.docker`
包含所有可配置的环境变量，可作为 `.env` 参考模板使用

**关键配置项：**
| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `ENABLE_NOTIFICATION` | 是否启用推送 | `false` |
| `ENABLE_CRAWLER` | 是否启用爬虫 | `true` |
| `RUN_MODE` | 运行模式 | `cron` |
| `CRON_SCHEDULE` | 定时执行周期 | `*/30 * * * *` |
| `REPORT_MODE` | 报告模式 | `daily` |

### 📄 `docker-compose.simple.yml`
**完全无推送渠道的最小化配置文件**

特点：
- ✅ 开箱即用，无需修改
- ✅ 已禁用所有推送渠道
- ✅ 每30分钟自动执行一次爬虫
- ✅ 容器启动时立即执行一次

### 📄 `docker/docker-compose.yml`
原始配置文件，包含所有推送渠道选项

---

## 容器启动

### 启动容器
```bash
# 后台启动
docker-compose -f docker-compose.simple.yml up -d

# 或前台启动（查看日志）
docker-compose -f docker-compose.simple.yml up
```

### 停止容器
```bash
docker-compose -f docker-compose.simple.yml down
```

### 重启容器
```bash
docker-compose -f docker-compose.simple.yml restart trend-radar
```

---

## 查看运行结果

### 📊 实时查看日志
```bash
# 查看最近日志
docker logs -f trend-radar

# 查看最后 100 行日志
docker logs --tail 100 trend-radar
```

### 📁 查看输出文件

新闻数据存储在 `./output` 目录，按日期组织：

```
output/
├── 2025年12月11日/
│   └── txt/
│       ├── 09时30分.txt
│       ├── 10时00分.txt
│       └── ...
├── 2025年12月12日/
│   └── txt/
│       └── ...
```

**查看最新输出：**
```bash
# 查看最新的爬虫结果
cat output/$(ls -t output | head -1)/txt/$(ls -t output/$(ls -t output | head -1)/txt | head -1)

# 或使用简便方法
find output -name "*.txt" -type f | xargs ls -t | head -1 | xargs cat
```

### 📝 输出文件说明

每个 `.txt` 文件包含：
1. **热点词汇统计** - 各平台热点词出现频率
2. **新增热点新闻** - 本次执行抓取到的新增新闻
3. **关键词匹配** - 与 `config.yaml` 中配置的关键词匹配结果

---

## 常用命令

### 🔍 查看容器状态
```bash
# 列出正在运行的容器
docker ps | grep trend-radar

# 查看容器详细信息
docker inspect trend-radar
```

### 📊 查看容器资源使用
```bash
docker stats trend-radar
```

### 🔧 进入容器调试
```bash
docker exec -it trend-radar bash
```

### 🗑️ 清理容器和数据
```bash
# 停止并删除容器（保留数据）
docker-compose -f docker-compose.simple.yml down

# 删除容器和所有数据
docker-compose -f docker-compose.simple.yml down -v

# 删除镜像
docker rmi wantcat/trendradar:latest
```

### 🔄 更新镜像版本
```bash
# 拉取最新镜像
docker pull wantcat/trendradar:latest

# 重建容器
docker-compose -f docker-compose.simple.yml up -d --force-recreate
```

---

## 日志说明

### 正常启动日志示例
```
✅ 配置文件检查通过
📅 生成的crontab内容:
*/30 * * * * cd /app && /usr/local/bin/python main.py
▶️ 立即执行一次
🌐 启动 Web 服务器...
⏰ 启动supercronic: */30 * * * *
```

### 常见日志和含义

| 日志 | 含义 | 解决方案 |
|------|------|---------|
| `❌ 配置文件缺失` | 缺少 `config.yaml` 或 `frequency_words.txt` | 检查 `config/` 目录 |
| `crontab格式验证失败` | CRON_SCHEDULE 格式错误 | 检查 cron 表达式格式 |
| `✅ 推送成功` | 推送成功（仅在配置了推送渠道时出现） | 正常 |
| `⏭️ 跳过推送` | 未启用推送功能 | 正常（当 ENABLE_NOTIFICATION=false 时） |

---

## 配置关键词和平台

编辑 `config/config.yaml` 文件：

### 1. 监听的平台
```yaml
platforms:
  - id: "toutiao"
    name: "今日头条"
  - id: "baidu"
    name: "百度热搜"
  # ... 更多平台
```

### 2. 关键词配置（添加自己关心的词汇）
编辑 `config/frequency_words.txt`，每行一个关键词：

```
AI
ChatGPT
特斯拉
```

### 3. 报告模式选择
```yaml
report:
  mode: "daily"  # daily(日报) / current(当前榜单) / incremental(增量)
```

---

## 后续启用推送渠道

当需要启用推送时，只需添加环境变量：

### 方式一：修改 `.env` 文件
```bash
# 复制配置模板
cp .env.docker .env

# 编辑 .env，添加推送渠道配置
nano .env

# 重启容器
docker-compose up -d
```

### 方式二：修改 docker-compose.yml
```yaml
environment:
  - ENABLE_NOTIFICATION=true      # 启用推送
  - FEISHU_WEBHOOK_URL=https://...  # 填入飞书 URL
  - TELEGRAM_BOT_TOKEN=xxx        # 填入 Telegram Token
  # ... 其他推送渠道
```

### 支持的推送渠道
- 🐦 **Telegram** - 需要 `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID`
- 📱 **飞书** - 需要 `FEISHU_WEBHOOK_URL`
- 🔔 **钉钉** - 需要 `DINGTALK_WEBHOOK_URL`
- 🏢 **企业微信** - 需要 `WEWORK_WEBHOOK_URL`
- 📧 **邮件** - 需要 `EMAIL_FROM` + `EMAIL_PASSWORD` + `EMAIL_TO`
- 🔊 **Slack** - 需要 `SLACK_WEBHOOK_URL`
- 🔔 **Bark** - 需要 `BARK_URL`
- 🌐 **ntfy** - 需要 `NTFY_TOPIC`

---

## 故障排查

### 问题 1：容器不断重启
```bash
# 查看错误日志
docker logs trend-radar | tail -50

# 检查配置文件
docker exec trend-radar cat /app/config/config.yaml
```

### 问题 2：没有生成输出文件
```bash
# 检查 RUN_MODE 和 CRON_SCHEDULE
docker logs trend-radar | grep -E "RUN_MODE|CRON_SCHEDULE"

# 检查 output 目录权限
ls -la output/
```

### 问题 3：占用端口被占用
```bash
# 查找占用 8080 端口的进程
lsof -i :8080

# 修改 docker-compose.yml 中的端口映射
# ports:
#   - "127.0.0.1:9090:8080"  # 改为 9090
```

### 问题 4：拉取镜像失败
```bash
# 检查网络连接
ping hub.docker.com

# 使用镜像加速（如果在中国）
# 编辑 Docker daemon 配置，添加镜像加速地址
```

---

## 性能优化建议

### 1. 调整执行频率
```yaml
# docker-compose.simple.yml
CRON_SCHEDULE=0 * * * *  # 改为每小时执行一次
```

### 2. 增加容器资源
```yaml
services:
  trend-radar:
    # ... 其他配置
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M
```

### 3. 持久化容器数据
```bash
# 创建 Docker 卷而不是目录映射
docker volume create trend-radar-output
docker volume create trend-radar-config

# 在 docker-compose.yml 中使用
volumes:
  - trend-radar-config:/app/config
  - trend-radar-output:/app/output
```

---

## 📚 更多资源

- 📖 [TrendRadar GitHub 仓库](https://github.com/sansan0/TrendRadar)
- 🐳 [Docker Hub 镜像](https://hub.docker.com/r/wantcat/trendradar)
- 📝 [Docker Compose 文档](https://docs.docker.com/compose/)

---

**需要帮助？** 检查本指南的相应部分或查看容器日志获取详细错误信息。
