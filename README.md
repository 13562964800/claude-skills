# Claude Skills & Commands 集合

完整的 Claude Code 技能和命令集合，包含50+个skills和多个commands。

## 快速同步到其他电脑

### 方法1：Git同步（推荐）

**在当前电脑（已配置）：**
```bash
cd ~/.claude
git remote add origin <你的GitHub仓库URL>
git push -u origin main
```

**在新电脑：**
```bash
# 克隆仓库
git clone <你的GitHub仓库URL> ~/.claude

# 安装Python依赖（如果需要）
pip install paddleocr-mcp -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 方法2：直接复制

**在当前电脑：**
```bash
# 打包整个.claude目录
cd ~
tar -czf claude-skills-backup.tar.gz .claude/
```

**在新电脑：**
```bash
# 解压到用户目录
cd ~
tar -xzf claude-skills-backup.tar.gz
```

### 方法3：云盘同步

将 `~/.claude` 目录放到云盘（OneDrive/Dropbox/坚果云），在其他电脑创建符号链接：

```bash
# Windows
mklink /D C:\Users\你的用户名\.claude "D:\OneDrive\.claude"

# Linux/Mac
ln -s /path/to/cloud/.claude ~/.claude
```

## 已安装内容

### Skills (50+)
- 法律文档处理（4个）
- 文档处理（PDF/Word/Excel/PPT）
- OCR识别（PaddleOCR + MinerU）
- 开发工具（MCP构建、Skill创建）
- 媒体处理（视频下载、语音转录）
- 研究工具（多主题搜索、仓库研究）

### MCP服务器
- PaddleOCR MCP（多语言文档解析）

### Commands
- brainstorm, write-plan, execute-plan
- zcf系列（git-commit, git-worktree等）

## 配置说明

某些skills需要配置API密钥：
- MinerU OCR: `~/.claude/skills/mineru-ocr/config/.env`
- 翻译工具: `~/.clawdbot/skills/funasr-transcribe/scripts/translate.py --setup`

## 更新同步

```bash
cd ~/.claude
git pull origin main
```
