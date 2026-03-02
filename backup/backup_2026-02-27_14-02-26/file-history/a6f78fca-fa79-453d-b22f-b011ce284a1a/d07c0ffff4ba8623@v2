# Android 多语言学习应用 - Bug 修复计划

## 上下文

用户报告了三个问题：
1. **听写模式中不能查词** - 点击听写模式中的"查词"按钮后，单词卡片不显示
2. **视频解析失败** - 在线视频链接解析（B站、抖音、YouTube等）经常失败
3. **转录失败** - Groq Whisper 转录功能失败

## 问题分析

### 问题1: 听写模式中不能查词

**根本原因**：`WordCardPanel` 组件被放置在"学习模式"的代码块内，导致在听写模式下无法显示。

**位置**：`d:\llapk\app\src\main\java\com\multilanglearner\ui\player\PlayerScreen.kt`
- 第204-213行：`WordCardPanel` 在 `if (uiState.mode == "learn")` 块内
- 第216-224行：`AI analysis result` 也在学习模式块内
- 第258-266行：听写模式只显示 `DictationPanel`，没有单词卡片显示

**代码流程**：
1. 用户点击听写模式中的"查词"按钮（第356-358行）
2. 调用 `viewModel.lookupWord(word)`，设置 `showWordCard = true`
3. 但 `WordCardPanel` 只在学习模式下渲染，所以卡片不显示

### 问题2: 视频解析失败

**根本原因**：依赖的第三方视频解析 API 不可靠。

**位置**：`d:\llapk\app\src\main\java\com\multilanglearner\data\network\VideoDownloadService.kt`

- **YouTube**（第123-246行）：API 只提供元数据，不提供下载链接
- **B站/抖音/TikTok**（第272-728行）：依赖多个第三方 API，这些 API 可能：
  - 暂时不可用或维护中
  - 触发限流
  - 返回 HTML 而非 JSON（API 失败标志）

**当前状态**：代码已有良好的重试机制，但需要更好的错误提示和备用方案。

### 问题3: 转录失败

**根本原因**：Groq API 配置问题或文件大小超限。

**位置**：`d:\llapk\app\src\main\java\com\multilanglearner\data\network\WhisperService.kt`

**可能的失败原因**：
1. 未配置 Groq API Key（第45-47行）
2. 文件超过 25MB 限制（第55-62行）
3. 网络连接问题（第128-130行）
4. API 返回错误（第90-104行）

## 修复方案

### 修复1: 听写模式查词功能

**修改文件**：`d:\llapk\app\src\main\java\com\multilanglearner\ui\player\PlayerScreen.kt`

将 `WordCardPanel` 和 `AI analysis result` 移到模式判断之外，使两种模式都能显示。

**具体修改**：
```kotlin
// 在第132行，if (uiState.mode == "learn") 之前
// 将 WordCardPanel (第204-213行) 移到这里
// 将 AI analysis result (第216-224行) 移到这里

if (uiState.mode == "learn") {
    // 学习模式内容
} else {
    // 听写模式内容
}

// WordCardPanel 和 AI analysis result 放在这里，两种模式都可见
```

### 修复2: 视频解析改进

**修改文件**：`d:\llapk\app\src\main\java\com\multilanglearner\data\network\VideoDownloadService.kt`

由于第三方 API 不可控，主要改进：
1. 更清晰的错误提示（建议用户使用导入功能）
2. 更新 API 列表（替换失效的 API）
3. 添加更详细的日志用于调试

**注意**：视频解析依赖外部 API，长期解决方案是：
- 引导用户使用"导入本地视频"功能
- 或在 PC 上使用 yt-dlp 下载后导入

### 修复3: 转录改进

**修改文件**：
1. `d:\llapk\app\src\main\java\com\multilanglearner\ui\player\PlayerViewModel.kt` - 改进错误处理
2. `d:\llapk\app\src\main\java\com\multilanglearner\data\network\WhisperService.kt` - 更详细的错误信息

**具体改进**：
1. 检测并显示文件大小问题（建议用户压缩视频）
2. 改进 API 错误解析（显示更具体的错误信息）
3. 添加重试机制（针对网络错误）

## 修改文件清单

| 优先级 | 文件 | 修改内容 |
|--------|------|----------|
| 高 | `PlayerScreen.kt` | 移动 WordCardPanel 到模式判断之外 |
| 中 | `PlayerViewModel.kt` | 改进转录错误提示 |
| 中 | `WhisperService.kt` | 改进错误信息 |
| 低 | `VideoDownloadService.kt` | 更新 API 列表（如需要） |

## 验证步骤

### 验证听写模式查词
1. 打开应用，进入播放器
2. 切换到"听写模式"
3. 点击任意单词下方的"查词"按钮
4. 确认单词卡片正常显示

### 验证视频解析
1. 在主页尝试输入 B站/抖音/YouTube 视频链接
2. 查看错误提示是否清晰
3. 如果失败，尝试使用"导入本地视频"功能

### 验证转录功能
1. 准备一个小于 25MB 的视频文件
2. 确认已在 AI 设置中配置 Groq API Key
3. 点击"重新转录"按钮
4. 查看转录进度和结果

## 关键代码位置参考

- 听写模式查词按钮：`PlayerScreen.kt:356-358`
- WordCardPanel 定义：`PlayerScreen.kt:271-311`
- lookupWord 函数：`PlayerViewModel.kt:261-309`
- 转录入口：`PlayerViewModel.kt:158-248`
- Whisper 服务：`WhisperService.kt:43-132`
