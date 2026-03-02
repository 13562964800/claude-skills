# 多语言视频学习应用 - 4大新功能实现计划

## Context
现有 Electron 应用（D:\learn）已具备：视频下载、字幕同步、点词翻译、AI分析、听写模式、14种语言支持。需新增4个功能模块。

## 实现顺序
1. 单词收藏本（基础，其他功能依赖）
2. 跨国法规库（独立，简单JSON数据）
3. 智能语音助手（独立，AI+TTS）
4. 批量单词成图（依赖收藏本数据）

## 修改文件清单

### 1. main.js - 新增IPC处理器
- `initUserDB()`: 创建 userdata.db，建 favorites 表
- `fav-add/fav-remove/fav-check/fav-list/fav-import`: 收藏CRUD
- `ai-group-words`: AI按语义分组单词
- `ai-scene-desc`: AI生成场景描述
- `ai-lecture`: AI整理讲稿
- `read-file-text`: 文件选择对话框读取文本
- `get-regulations`: 读取 regulations.json

### 2. index.html - 新增UI元素
- 控制栏：📖词书、🎙助手、⚖法规 三个按钮
- 收藏按钮：word popup 和 inline panel 各加一个 ☆收藏
- 4个新 modal：词书、批量成图、语音助手、法规库
- 隐藏 canvas 用于成图

### 3. styles.css - 新增样式
- `.btn-fav` 收藏按钮样式
- `.vocab-grid/.vocab-card` 词书卡片网格
- `.img-card` 成图卡片
- `.law-item` 法规条目

### 4. renderer.js - 新增功能函数
- 收藏：`updateFavState()`, `loadVocabList()`, 收藏按钮事件
- 成图：`renderWordGroupImage()` Canvas渲染, AI分组+导出流程
- 助手：`speakLecture()` 分段朗读, 文档上传+AI处理
- 法规：`loadRegulations()`, 国家选择+朗读

### 5. 新文件: regulations.json
- 中国、美国、日本、韩国等国家的示例法规条文

## 数据库
userdata.db (新建，与只读的 stardict.db 分离):
```sql
CREATE TABLE favorites (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  word TEXT NOT NULL, lang TEXT NOT NULL DEFAULT 'en',
  phonetic TEXT DEFAULT '', translation TEXT DEFAULT '',
  context TEXT DEFAULT '', created_at INTEGER NOT NULL
);
CREATE UNIQUE INDEX idx_fav_word_lang ON favorites(word, lang);
```

## 验证方式
1. `npm start` 启动应用
2. 播放视频 → 点击字幕单词 → 点☆收藏 → 打开词书确认卡片显示
3. 词书中点"导入"（剪贴板粘贴单词列表）→ 点"批量成图" → 确认图片生成并可下载
4. 点🎙助手 → 上传txt文档 → AI生成讲稿 → 点朗读
5. 点⚖法规 → 选择国家 → 查看法规 → 点朗读按钮
