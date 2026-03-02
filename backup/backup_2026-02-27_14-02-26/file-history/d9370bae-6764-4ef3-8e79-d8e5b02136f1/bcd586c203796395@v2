# Android 版多语言视频学习 - 全功能复刻实施计划

## Context

将 D:\ll 的 Electron 桌面应用"多语言视频学习"完面复刻为 Android 原生应用，部署到 D:\llapk。原应用包含视频下载、字幕提取、AI 词汇释义、听写训练、法规库等完整功能，使用 JavaScript + Electron 构建。Android 版采用 Kotlin + Jetpack Compose + Material 3 现代技术栈。

## 技术栈

| 层级 | 技术 |
|------|------|
| UI | Jetpack Compose + Material 3 |
| 视频 | Media3 ExoPlayer |
| 数据库 | Room (SQLite) |
| 网络 | Retrofit + OkHttp + Kotlin Serialization |
| DI | Hilt |
| 视频下载 | youtubedl-android (yt-dlp wrapper) |
| TTS | Android TextToSpeech |
| 偏好 | DataStore Preferences |
| 图片 | Coil + Compose Canvas |

## 项目结构

```
D:\llapk/
├── app/                              # 主模块
│   └── src/main/java/com/multilanglearner/
│       ├── MainApplication.kt        # Hilt + YoutubeDL 初始化
│       ├── MainActivity.kt           # 单 Activity
│       ├── navigation/NavGraph.kt    # 导航图
│       ├── data/                     # 数据层
│       │   ├── db/                   # Room 数据库
│       │   │   ├── AppDatabase.kt
│       │   │   ├── entity/           # FavoriteWord, UserRegulation, VideoEntity
│       │   │   └── dao/              # FavoriteDao, RegulationDao, VideoDao
│       │   ├── store/                # DataStore
│       │   │   ├── AIConfigStore.kt
│       │   │   └── ThemeStore.kt
│       │   ├── repository/           # Repository 实现
│       │   │   ├── FavoriteRepository.kt
│       │   │   ├── VideoRepository.kt
│       │   │   └── RegulationRepository.kt
│       │   └── network/              # API 服务
│       │       ├── AIService.kt      # 统一 AI Chat API
│       │       ├── WhisperService.kt # Groq Whisper
│       │       ├── DictionaryService.kt # 词典 API
│       │       └── TranslateService.kt  # 百度翻译
│       ├── domain/                   # 领域模型
│       │   ├── model/                # SubtitleCue, WordTimestamp, AIConfig 等
│       │   └── util/                 # VttParser, TextSegmenter 等算法
│       ├── ui/                       # UI 层
│       │   ├── theme/                # 3 套主题 (极光/宋式/系统)
│       │   ├── component/            # 共享组件
│       │   ├── home/                 # 主页 (URL输入+视频列表+控制栏)
│       │   ├── player/               # 播放器 (视频+字幕+词卡+听写)
│       │   ├── vocabulary/           # 词书管理 + 成图
│       │   ├── ai/                   # AI 设置
│       │   ├── assistant/            # 语音助手
│       │   └── regulation/           # 法规库
│       └── di/AppModule.kt          # Hilt 模块
├── build.gradle.kts                  # 根构建
├── app/build.gradle.kts              # 应用构建
├── settings.gradle.kts
└── gradle.properties
```

## 功能模块对应 (共 10 大模块)

### 1. 视频下载 (对应 main.js download-video)
- youtubedl-android 集成，支持 URL 下载 + 字幕提取
- 进度条实时反馈
- 本地视频导入 (ActivityResultContracts)
- 已下载视频列表管理

### 2. 字幕系统 (对应 main.js generate-subtitle + renderer.js 字幕渲染)
- VTT 解析器 (处理 YouTube 双行格式、去重、跳过极短条目)
- 卡拉OK 式逐词高亮 (30ms 轮询 + 二分查找)
- 逐词时间戳精准对齐 (Whisper word-level timestamps)
- 字幕滚动列表 (LazyColumn + 自动滚动)
- 双语字幕翻译 (AI/百度)

### 3. 语音识别 (对应 main.js Groq Whisper)
- Groq Whisper API 调用 (分段 10 分钟)
- 逐词时间戳 JSON 生成
- 前台转录 + 后台继续处理

### 4. AI 多模型 (对应 main.js callAI + 6 家预设)
- 统一 OpenAI 兼容接口 + Claude 单独处理
- 6 家预设: OpenAI, Claude, Groq, DeepSeek, Qwen, Zhipu
- 自定义 OpenAI 兼容 API
- 功能: 词汇释义、句子分析、词汇分组、场景故事、讲稿整理

### 5. 词汇学习 (对应 renderer.js 词卡系统)
- 点击字幕单词即时查词 (ECDICT 本地 + 在线词典 + AI)
- 音形义例句四位一体词卡
- 收藏/取消收藏
- 短语选择收藏
- TTS 发音

### 6. 词书管理 (对应 vocabModal)
- 收藏词汇网格展示
- 全选/批量删除
- 导入/导出
- AI 分类成图
- Canvas 词汇图片生成 (Bitmap + Canvas)

### 7. 听写训练 (对应 dictationArea)
- 逐词输入框
- 实时检查反馈
- 答案提示
- TTS 朗读

### 8. 语音助手 (对应 assistModal)
- 文档上传
- AI 讲稿整理
- 分段 TTS 朗读 (≤200字/段)

### 9. 法规库 (对应 lawModal + regulations.json)
- 内置中美日韩法规
- 用户自定义法规
- TTS 朗读
- 增删管理

### 10. 主题与设置
- 3 套主题: 极光暗色、宋式美学、跟随系统
- 4 档字号: 小/中/大/特大
- AI 配置持久化 (DataStore)

## 关键算法移植

### VTT 解析 (main.js parseVTT → VttParser.kt)
- 按 `\n\n` 分块 → 正则匹配时间戳 → 跳过 <50ms → 取最后行 → 去重合并

### 逐词对齐 (main.js buildWordTimestamps → WordAligner.kt)
- Whisper API 返回 word-level timestamps → 按标点分句 → 显示词与 API 词模糊匹配

### 字幕同步 (renderer.js syncSubtitle → PlayerViewModel)
- 30ms 协程轮询 ExoPlayer.currentPosition → 二分查找当前 cue → Flow 驱动 UI 更新

### AI 调用 (main.js callAI → AIRepository.kt)
- Claude: POST /messages, x-api-key header, anthropic-version
- 其他: POST /chat/completions, Bearer token

### Canvas 成图 (renderer.js generateImage → VocabImageGenerator.kt)
- Android Bitmap.createBitmap → Canvas 绘制渐变背景 + 标题 + 词卡网格 + 水印

### TTS 分段 (renderer.js speakLecture → TextSegmenter.kt)
- 按句末标点切分 → 每段 ≤200 字 → Android TextToSpeech 逐段播放

## 实施步骤

### Phase 1: 项目骨架
1. 创建 Android 项目 (D:\llapk)，配置 Gradle、Hilt、Compose
2. 定义 Room 数据库 + 实体 + DAO
3. 实现 3 套主题系统
4. 搭建导航框架

### Phase 2: 核心播放
5. 集成 ExoPlayer 视频播放
6. 实现 VTT 解析器
7. 实现字幕同步 + 卡拉OK 逐词高亮
8. 实现字幕列表滚动

### Phase 3: 视频获取
9. 集成 youtubedl-android 视频下载
10. 实现本地视频导入
11. 实现 Groq Whisper 语音识别
12. 实现字幕翻译 (AI + 百度)

### Phase 4: AI 与词汇
13. 实现统一 AI 服务 (6 家 + 自定义)
14. 实现词卡系统 (查词 + 释义 + 例句)
15. 实现收藏系统 (Room CRUD)
16. 实现词书管理界面

### Phase 5: 扩展功能
17. 实现听写训练模式
18. 实现语音助手 (文档 + TTS)
19. 实现法规库
20. 实现 Canvas 词汇成图
21. 实现 AI 图片生成 (Pollinations)

### Phase 6: 完善
22. 实现 AI 分类成图完整流程
23. 实现场景故事生成
24. 全面测试 + 修复

## 验证方式
1. 构建 APK: `./gradlew assembleDebug`
2. 逐功能验证: 下载视频 → 字幕显示 → 点词查询 → 收藏 → 听写 → AI 分析 → 词书成图 → 法规库
3. 主题切换验证: 3 套主题 + 4 档字号
