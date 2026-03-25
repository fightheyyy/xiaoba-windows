---
name: opencli
description: "通过命令行操作社交/内容平台（B站、知乎、Twitter/X、YouTube、微博、小红书、Reddit、HackerNews、雪球等）。复用人已登录的 Chrome 浏览器会话，无需 API Key。需要先安装 npm install -g @jackwener/opencli 和对应的 Chrome 扩展。"
invocable: user
autoInvocable: true
argument-hint: "<操作描述>"
max-turns: 30
npm-dependencies:
  - "@jackwener/opencli@^1.1.1"
---

# OpenCLI

通过 `opencli` 命令行工具操作各大平台，复用 Chrome 浏览器已登录的会话。

## 前置条件

1. 安装 opencli CLI：`npm install -g @jackwener/opencli`
2. 在 Chrome 安装 Playwright MCP Bridge 扩展
3. 确保 Chrome 已打开并登录目标平台

## 支持的平台（16个）

| 平台 | 读取 | 搜索 | 写操作 |
|------|------|------|--------|
| B站 (Bilibili) | ✅ 热门/排行/动态/历史 | ✅ 视频/用户 | — |
| 知乎 | ✅ 热榜 | ✅ | ✅ 问题详情 |
| 微博 | ✅ 热搜 | — | ✅ 发微博 |
| Twitter/X | ✅ 时间线/热门/书签 | ✅ | ✅ 发推/回复/点赞 |
| YouTube | — | ✅ | — |
| 小红书 | ✅ 推荐 Feed | ✅ | — |
| Reddit | ✅ 首页/热门 | ✅ | — |
| HackerNews | ✅ Top | — | — |
| V2EX | ✅ 热门/最新 | — | ✅ 签到 |
| 雪球 | ✅ 热门/行情/自选股 | ✅ | — |
| BOSS直聘 | — | ✅ 职位 | — |
| BBC | ✅ 新闻 | — | — |
| 路透社 | — | ✅ | — |
| 什么值得买 | — | ✅ | — |
| Yahoo Finance | ✅ 股票行情 | — | — |
| 携程 | — | ✅ 景点/城市 | — |

## 常用命令

### B站

```bash
opencli bilibili hot --limit 10 -f json       # 热门视频
opencli bilibili search --keyword "AI"         # 搜索视频
opencli bilibili history                       # 我的观看历史
opencli bilibili ranking                       # 排行榜
```

### 知乎

```bash
opencli zhihu hot -f json                     # 热榜
opencli zhihu search --keyword "大模型"        # 搜索
opencli zhihu question <question-id>           # 问题详情
```

### Twitter/X

```bash
opencli twitter timeline -f json              # 时间线
opencli twitter trending                      # 热门话题
opencli twitter search --query "AI"           # 搜索
opencli twitter post --text "Hello!"         # 发推
opencli twitter bookmarks                     # 书签
```

### YouTube

```bash
opencli youtube search --query "LLM教程"      # 搜索视频
```

### HackerNews

```bash
opencli hackernews top --limit 20 -f json    # Top 20
```

### Reddit

```bash
opencli reddit hot                           # 热门帖子
opencli reddit hot --subreddit MachineLearning  # 指定板块
opencli reddit search --query "AI"           # 搜索
```

### 雪球

```bash
opencli xueqiu stock --symbol SH600519       # 茅台行情
opencli xueqiu watchlist                     # 我的自选股
opencli xueqiu hot                          # 热门
```

### 微博

```bash
opencli weibo hot -f json                    # 热搜
```

### 小红书

```bash
opencli xiaohongshu feed                     # 推荐 Feed
opencli xiaohongshu search --keyword "美食"  # 搜索
```

## 输出格式

所有命令支持 `-f` 参数指定输出格式：

```bash
-f table    # 表格（默认）
-f json     # JSON
-f yaml     # YAML
-f md       # Markdown
-f csv      # CSV
```

## 工作流程

1. 用户提出需求（如"查下B站今天的热门"）
2. 判断最合适的 opencli 命令
3. 执行命令并展示结果
4. 如需写操作（发推、发微博），先展示内容确认

## 注意事项

- 读操作：直接执行并返回结果
- 写操作：先展示内容，征得用户确认后再执行
- 某些平台可能触发验证码或限流，避免频繁操作
- Chrome 必须保持打开状态
