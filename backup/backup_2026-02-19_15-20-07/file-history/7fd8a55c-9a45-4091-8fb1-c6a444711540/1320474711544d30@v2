# 录音文档对比 - Windows 桌面版计划

## Context
Android APK 版已完成，用户需要一个功能相同的 Windows 桌面 exe 版本。用 Python + tkinter 实现，pyinstaller 打包。用户还要求支持长语音分块转写。

## 技术栈
- **GUI**: tkinter (Python 内置，无额外依赖)
- **语音转写**: 复用本地 FunASR 服务(中文) + OpenAI Whisper API(外语) + 阿里云 ASR
- **长语音分块**: pydub 切分音频为 ≤60s 片段，逐段转写后合并
- **Word 解析/生成**: python-docx
- **LLM 对比**: requests 调用各家 API (同 Android 版 8 个 LLM 服务商)
- **打包**: pyinstaller → 单 exe

## 文件结构
```
~/voice-doc-compare/desktop/
├── main.py              # 入口 + tkinter GUI (首页、结果页、设置页)
├── transcribe.py        # 语音转写 (阿里云/Whisper/FunASR + 分块逻辑)
├── compare.py           # LLM 语义对比 (8 个服务商)
├── docx_util.py         # Word 解析 + 标红生成
├── config.py            # 配置管理 (JSON 文件存储)
└── build.py             # pyinstaller 打包脚本
```

## 实现步骤

### 1. config.py - 配置管理
- 同 Android 版的 ASR_PROVIDERS (8个) 和 LLM_PROVIDERS (8个)
- JSON 文件存储在 `~/.voice-doc-compare/config.json`
- load_config / save_config / is_valid

### 2. transcribe.py - 语音转写 + 长语音分块
- **分块策略**: 用 pydub 按静音点切分，每段 ≤ 60s；若无静音点则硬切 55s + 5s 重叠
- **FunASR**: 调用本地 http://localhost:8765/transcribe
- **阿里云**: 移植 Android 版的 HMAC-SHA1 签名 + 流式上传
- **OpenAI Whisper**: multipart 上传
- 分块转写后按顺序合并文本

### 3. compare.py - LLM 语义对比
- 移植 Android 版 prompt 模板和 JSON 解析逻辑
- Claude API (特殊 header) + OpenAI 兼容 API (其余7个)
- 返回 DiffItem 列表 + summary

### 4. docx_util.py - Word 处理
- python-docx 读取段落文本
- python-docx 在差异段落后插入红色粗体标注段落
- 输出到桌面 `对比结果_时间戳.docx`

### 5. main.py - tkinter GUI
- **首页**: 选择录音文件 + Word 文件 → 开始对比按钮 + 进度条
- **结果页**: 差异数量 + 摘要 + 录音文本 + 文档文本 + 打开结果文档按钮
- **设置页**: ASR/LLM 服务商选择 + API Key 输入 + 测试连接
- 后台线程执行转写/对比，避免 GUI 卡死

### 6. build.py - 打包
- pyinstaller --onefile --windowed --name 录音文档对比

## 验证
1. 运行 `python main.py` 启动 GUI
2. 选择一个短音频 + Word 文档，测试完整流程
3. 选择一个长音频 (>5分钟)，验证分块转写
4. pyinstaller 打包后测试 exe 运行
