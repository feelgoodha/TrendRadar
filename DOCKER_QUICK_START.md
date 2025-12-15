# 🚀 TrendRadar Docker 快速启动

## 一行命令启动（无推送）
```bash
cd /Users/runysun/program/TrendRadar && docker-compose -f docker-compose.simple.yml up -d
```

## 查看实时日志
```bash
docker logs -f trend-radar
```

## 查看爬虫结果
```bash
# macOS/Linux
cat output/$(ls -t output | head -1)/txt/$(ls -t output/$(ls -t output | head -1)/txt | head -1)

# 或直接看最新文件
ls -lt output/*/txt/*.txt | head -5
```

## 停止容器
```bash
cd /Users/runysun/program/TrendRadar && docker-compose -f docker-compose.simple.yml down
```

---

## 📊 项目架构

```
TrendRadar
├── 📥 多平台爬虫
│   ├── 今日头条
│   ├── 百度热搜
│   ├── 微博、抖音、B站
│   └── 其他 10+ 平台
│
├── 🔍 新闻分析
│   ├── 关键词匹配（从 config/frequency_words.txt）
│   ├── 热度统计
│   └── 去重聚合
│
├── 📋 报告生成（3种模式）
│   ├── daily：日报汇总
│   ├── current：当前榜单
│   └── incremental：增量监控
│
├── 🔔 推送渠道（可选）
│   ├── Telegram、飞书、钉钉
│   ├── 企业微信、邮件、Slack
│   └── Bark、ntfy
│
└── 🤖 MCP 服务（AI集成）
    └── 可作为 Claude 工具使用
```

---

## 🎯 使用流程

### 1️⃣ 启动容器（一次性）
```bash
docker-compose -f docker-compose.simple.yml up -d
```
✅ 容器会自动：
- 立即执行一次爬虫
- 每30分钟执行一次
- 输出到 `output/` 目录

### 2️⃣ 监控运行状态
```bash
docker logs -f trend-radar
```

### 3️⃣ 查看爬虫结果
```bash
output/日期/txt/时间.txt
```

### 4️⃣ 后续启用推送（可选）
```bash
# 编辑 .env 文件
cp .env.docker .env
nano .env  # 添加推送配置

# 重启容器
docker-compose restart trend-radar
```

---

## 🔧 核心配置项

### 爬虫配置
```yaml
ENABLE_CRAWLER=true              # 启用爬虫
CRAWLER_REQUEST_INTERVAL=1000    # 请求延迟（毫秒）
```

### 报告配置
```yaml
REPORT_MODE=daily                # daily/current/incremental
MAX_NEWS_PER_KEYWORD=0           # 每个关键词的新闻数（0=不限）
```

### 定时配置
```yaml
RUN_MODE=cron                    # cron 定时执行
CRON_SCHEDULE=*/30 * * * *       # 每30分钟执行
IMMEDIATE_RUN=true               # 启动时立即执行
```

### ⭐ 推送控制（关键！）
```yaml
ENABLE_NOTIFICATION=false        # 不推送任何消息
```

---

## 📋 文件清单

| 文件 | 说明 | 修改频率 |
|------|------|---------|
| `docker-compose.simple.yml` | **推荐使用**，无推送配置 | 稀少 |
| `.env.docker` | 环境变量参考模板 | 稀少 |
| `config/config.yaml` | 平台、关键词配置 | 常常 |
| `config/frequency_words.txt` | 监听的关键词列表 | 常常 |
| `output/` | 爬虫输出结果 | 自动生成 |

---

## 🐛 故障排查

### 容器无法启动？
```bash
docker logs trend-radar | head -20
```
检查是否缺少 `config/config.yaml` 或 `config/frequency_words.txt`

### 没有输出文件？
```bash
# 检查 RUN_MODE
docker exec trend-radar bash -c 'echo $RUN_MODE'

# 检查 CRON_SCHEDULE
docker exec trend-radar bash -c 'echo $CRON_SCHEDULE'
```

### 容器占用高资源？
编辑 `docker-compose.simple.yml`，增大 `CRON_SCHEDULE` 间隔：
```yaml
CRON_SCHEDULE=0 * * * *  # 改为每小时一次
```

---

## 💡 高级用法

### 启用 Web UI 查看
```bash
# 编辑 docker-compose.simple.yml
ENABLE_WEBSERVER=true
WEBSERVER_PORT=8080

# 访问 http://localhost:8080
```

### 启用 MCP 服务（AI集成）
在 `docker-compose.simple.yml` 中取消注释 `trend-radar-mcp` 部分

### 切换报告模式
```yaml
REPORT_MODE=daily        # 日报（默认）
REPORT_MODE=current      # 当前榜单
REPORT_MODE=incremental  # 增量监控
```

---

## 📞 更多帮助

- 📖 详细指南：`DOCKER_DEPLOY_GUIDE.md`
- 🔗 项目主页：https://github.com/sansan0/TrendRadar
- 💬 遇到问题？查看 `docker logs trend-radar` 的错误信息

**现在就试试吧！** 🎉
