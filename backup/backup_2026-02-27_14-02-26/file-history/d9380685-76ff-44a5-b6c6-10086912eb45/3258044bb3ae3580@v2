# 多语言视频学习软件

## Context
用户需要一个基于视频字幕的多语言学习桌面应用。通过下载YouTube等平台视频并提取字幕，实现字幕同步播放、单词点击查词（释义+发音+例句）、听写填空等功能。支持与中国经常做生意的十几个国家的语言。

借鉴 `/e/视频下载器具/` 的 Electron 架构和 yt-dlp 下载方案。

## 支持语言
英语、日语、韩语、俄语、泰语、越南语、马来语、印尼语、阿拉伯语、葡萄牙语、西班牙语、法语、德语、印地语

## 技术方案

### 词典与发音
- **离线词典**: ECDICT (SQLite, ~770K英文词条含中文释义) 用 `better-sqlite3` 查询
- **在线补充**: Free Dictionary API (`dictionaryapi.dev`) 获取例句，支持所有语言回退到翻译API
- **发音**: Web Speech API (Chromium内置TTS)，支持多语言 `speechSynthesis`
- **多语言翻译**: 使用免费翻译API（MyMemory Translation API，无需key）作为非英语语言的释义来源

### 核心架构 (复用视频下载器模式)
- Electron ^30.0.0, `nodeIntegration:true`, `contextIsolation:false`
- yt-dlp 路径指向 `/e/视频下载器具/bin/yt-dlp.exe`
- IPC: `ipcMain.handle()` ↔ `ipcRenderer.invoke()`
- UI: Aurora 极光玻璃拟态风格 (dark theme)

## 文件结构
```
/d/learn/
├── package.json
├── main.js          # Electron主进程: 窗口、yt-dlp字幕提取、ECDICT查询、翻译API
├── renderer.js      # 前端逻辑: 视频播放、字幕同步、单词点击、听写模式
├── index.html       # UI结构
└── styles.css       # Aurora玻璃拟态样式
```

## 实现步骤

### Step 1: package.json + main.js
- `package.json`: electron, better-sqlite3 依赖
- `main.js` IPC handlers:
  - `download-video`: 用yt-dlp下载视频+字幕 (`--write-sub --write-auto-sub --sub-lang en --sub-format vtt`)
  - `lookup-word`: ECDICT SQLite查询英文单词 → 返回 {word, phonetic, translation}
  - `translate-word`: 调用MyMemory API翻译非英语单词到中文
  - `fetch-examples`: 调用 dictionaryapi.dev 获取例句
- 下载目录: app目录下的 `downloads/` 文件夹

### Step 2: index.html + styles.css
- 极光背景 + 玻璃卡片 (复用视频下载器样式)
- 布局区域:
  - 顶部: URL输入 + 语言选择下拉框 + 下载按钮
  - 中部: HTML5 `<video>` 播放器
  - 下部: 字幕显示区 (每个单词可点击的 `<span>`)
  - 底部: 模式切换 (学习模式 / 听写模式)
- 单词弹窗: 释义、音标、发音按钮、例句
- 听写模式: 句子拆分为input框，正确绿色/错误红色

### Step 3: renderer.js
- **VTT解析**: 解析字幕文件提取 `{start, end, text}` 数组
- **字幕同步**: `video.ontimeupdate` 匹配当前时间对应字幕，高亮显示
- **单词点击**: 每个单词包裹 `<span class="word">`，点击触发查词弹窗
- **发音**: `speechSynthesis.speak()` 设置对应语言的 `lang` 属性
- **听写模式**:
  - TTS朗读当前字幕句子
  - 句子拆分为单词，每个显示为 `<input>` 框
  - 提交后对比，正确绿/错误红+显示正确答案
  - 点击任意单词仍可弹出释义

### Step 4: 下载ECDICT数据库
- 从GitHub releases下载 `stardict.db` 放到项目目录
- 首次运行时如果不存在，提示用户

## 数据流
```
用户粘贴URL → renderer.js → IPC → main.js
  → yt-dlp 下载视频(.mp4) + 字幕(.vtt)
  → 返回文件路径给 renderer.js
  → video.src = 本地文件路径
  → 解析VTT → 字幕数组
  → video.timeupdate → 匹配字幕 → 渲染可点击单词
  → 点击单词 → IPC lookup-word → ECDICT/翻译API → 弹窗显示
  → 听写模式 → TTS朗读 → 用户填写 → 校验
```

## 验证方式
1. `cd /d/learn && npm install && npm start`
2. 粘贴一个YouTube链接，选择英语，点击下载
3. 视频播放时字幕同步滚动
4. 点击字幕中的单词，弹出释义+发音
5. 切换听写模式，测试填空功能
6. 切换其他语言测试翻译功能
