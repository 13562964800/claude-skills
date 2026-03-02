# GitHub Star 追踪器

> 让你的 GitHub Star 不再吃灰 - 自动追踪项目更新、版本发布和活跃度

## 特性

- ✅ **更新检测**：自动发现新版本 Release 和重要更新
- ✅ **活跃度监控**：追踪项目的 Commit 活跃度
- ✅ **增量对比**：只报告有变化的项目
- ✅ **AI 摘要**：可选使用 LLM 生成项目智能摘要
- ✅ **本地缓存**：快照机制，无需重复请求 API

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 设置 GitHub Token（推荐）

```bash
export GITHUB_PAT="你的_github_personal_access_token"
```

> Token 作用：提高 API 请求限额（60 → 5000次/小时）

### 3. 使用

```bash
# 首次初始化 - 保存当前 Star 快照
python main.py --init --user yourname

# 检查更新 - 对比快照查看变化
python main.py --check --user yourname

# 生成报告 - 包含版本更新和活跃度分析
python main.py --report --user yourname --days 7
```

## 命令详解

### `--init` 初始化模式

首次使用时运行，建立 Star 列表的快照基准。

```bash
python main.py --init --user yourname --limit 100
```

参数：
- `--limit`: 处理的 Star 数量上限（默认 100）
- `--summarize`: 同时生成 AI 项目摘要（需要 OpenAI API Key）

### `--check` 检查模式

对比上次快照，检测新增、取消 Star 和有更新的项目。

```bash
python main.py --check --user yourname
```

输出示例：
```
🆕 新增 Star (2 个):
  • anthropic/claude-code
  • openai/tiktoken

🔄 有更新的项目 (5 个):
  • vercel/next.js (2026-02-10)
  • facebook/react (2026-02-09)
```

### `--report` 报告模式

生成包含版本更新、活跃度分析的完整 Markdown 报告。

```bash
python main.py --report --user yourname --days 7 --output weekly_report.md
```

参数：
- `--days`: 时间范围（天），默认 7
- `--output`: 输出文件路径，默认 `star_report.md`

## 配置文件

编辑 `config.json` 自定义行为：

```json
{
  "check_interval_days": 7,      // 检查间隔
  "min_stars": 100,              // 最少 stars（过滤小项目）
  "focus_areas": ["AI", "Claude"], // 关注领域
  "ignore_repos": ["deprecated/repo"], // 忽略的项目
  "report_format": "markdown",     // 报告格式
  "include_forks": false,          // 是否包含 fork 项目
  "max_repos": 200               // 最大处理数量
}
```

## 定期运行建议

使用 cron 或类似工具定期运行：

```bash
# 每周一上午 9 点检查更新
0 9 * * 1 cd /path/to/github-star-tracker && python main.py --check --user yourname

# 每月生成完整报告
0 9 1 * * cd /path/to/github-star-tracker && python main.py --report --user yourname --days 30
```

## 与其他工具对比

| 功能 | github-star-tracker | repo-research | 其他星标管理工具 |
|------|---------------------|---------------|------------------|
| 更新追踪 | ✅ 核心功能 | ❌ 不支持 | ⚠️ 部分支持 |
| 版本通知 | ✅ Release 检测 | ❌ 不支持 | ⚠️ 部分支持 |
| 深度分析 | ❌ 轻量摘要 | ✅ 架构/代码解读 | ❌ 不支持 |
| 用途 | 日常订阅更新 | 一次性深度调研 | 基础管理 |

## License

MIT
