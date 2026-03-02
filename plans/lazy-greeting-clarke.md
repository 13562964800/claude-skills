# 浏览器扩展 + 本地服务视频下载转录系统设计方案

## Context（背景）

用户当前有一个 Android 应用（d:\llapk）用于视频下载和转录，但存在以下问题：
1. **YouTube 解析极其不稳定** - InnerTube API 经常被限制，第三方 API 不提供下载链接
2. **哔哩哔哩解析尚可** - 但 DASH 格式处理复杂
3. **转录功能问题** - 本地转录未实现，仅依赖云端 Groq API

用户需要一个**全新的浏览器扩展 + 本地服务架构**，实现：
- 浏览器自动侦测视频页面，注入提取按钮
- 一键提取真实视频链接并发送到本地服务
- 本地服务自动下载视频
- 快速精准检测语言并自动转录
- 支持在线/本地转录引擎无缝切换

---

## 系统架构设计

### 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    浏览器扩展层                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Content      │  │ Background   │  │ Popup UI     │      │
│  │ Script       │←→│ Service      │←→│              │      │
│  │ (页面注入)   │  │ Worker       │  │ (用户界面)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         ↓                 ↓                                  │
└─────────┼─────────────────┼──────────────────────────────────┘
          │                 │
          │    ┌────────────┴────────────┐
          │    │  Native Messaging API   │
          │    │  或 WebSocket (ws://)   │
          │    └────────────┬────────────┘
          │                 ↓
┌─────────┴─────────────────────────────────────────────────────┐
│                    本地服务层 (Python)                         │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  HTTP/WebSocket 服务器 (FastAPI/Flask)                   │ │
│  │  - 接收视频链接                                          │ │
│  │  - 任务队列管理                                          │ │
│  │  - 进度推送                                              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  视频解析模块                                            │ │
│  │  - yt-dlp (YouTube/B站/抖音等)                          │ │
│  │  - 平台检测与多 API 轮询                                │ │
│  │  - Cookie 管理                                           │ │
│  └──────────────────────────────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  下载模块                                                │ │
│  │  - 分段下载 (aria2c 或自实现)                           │ │
│  │  - 断点续传                                              │ │
│  │  - 进度回调                                              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  转录模块 (混合引擎)                                     │ │
│  │  ┌────────────────┐  ┌────────────────┐                 │ │
│  │  │ 本地引擎       │  │ 在线引擎       │                 │ │
│  │  │ - faster-whisper│  │ - Groq API     │                 │ │
│  │  │ - whisper.cpp  │  │ - OpenAI API   │                 │ │
│  │  │ - Moonshine    │  │                │                 │ │
│  │  └────────────────┘  └────────────────┘                 │ │
│  │  语言检测: SenseVoice / Qwen3-ASR / Whisper             │ │
│  └──────────────────────────────────────────────────────────┘ │
│                           ↓                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  存储模块                                                │ │
│  │  - 视频文件: ~/Downloads/videos/                        │ │
│  │  - 字幕文件: .vtt / .srt / .json                        │ │
│  │  - 数据库: SQLite (任务历史/配置)                       │ │
│  └──────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

---

## 核心功能设计

### 1. 浏览器扩展 (Chrome/Edge)

#### 1.1 Content Script (内容脚本)
**职责**: 注入到视频页面，检测视频并添加下载按钮

**实现要点**:
- **平台检测**: 通过 URL 正则匹配识别平台
  ```javascript
  const PLATFORMS = {
    youtube: /youtube\.com|youtu\.be/i,
    bilibili: /bilibili\.com|b23\.tv/i,
    douyin: /douyin\.com/i,
    // ... 更多平台
  };
  ```

- **视频检测**:
  - YouTube: 监听 `yt-navigate-finish` 事件
  - B站: 监听 DOM 变化，检测 `<video>` 标签
  - 通用: MutationObserver 监听页面变化

- **按钮注入**:
  ```javascript
  // 创建浮动按钮
  const button = document.createElement('div');
  button.className = 'video-dl-button';
  button.innerHTML = '📥 下载';
  button.onclick = () => extractAndSend();

  // 注入到合适位置（平台特定）
  const container = document.querySelector('.video-controls');
  container.appendChild(button);
  ```

- **链接提取**:
  - 获取当前页面 URL
  - 提取视频标题、时长等元数据
  - 发送到 Background Script

#### 1.2 Background Service Worker (后台脚本)
**职责**: 中央通信枢纽，管理与本地服务的连接

**实现要点**:
- **通信管理**:
  ```javascript
  // 接收来自 Content Script 的消息
  chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
    if (msg.action === 'download') {
      sendToLocalService(msg.data);
    }
  });

  // 与本地服务通信 (WebSocket)
  const ws = new WebSocket('ws://localhost:8765');
  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    // 更新进度、通知用户
    updateProgress(data);
  };
  ```

- **Cookie 管理**:
  ```javascript
  // 获取当前站点 Cookie
  async function getCookies(domain) {
    return await chrome.cookies.getAll({ domain });
  }

  // 发送 Cookie 到本地服务
  const cookies = await getCookies('bilibili.com');
  sendToLocalService({ url, cookies });
  ```

- **通知系统**:
  ```javascript
  chrome.notifications.create({
    type: 'progress',
    title: '下载中',
    message: '正在下载视频...',
    progress: 50
  });
  ```

#### 1.3 Popup UI (弹出界面)
**职责**: 用户配置和任务管理

**功能**:
- 查看下载队列和进度
- 配置转录引擎（在线/本地）
- 管理 Cookie 和 API Key
- 查看历史记录

---

### 2. 本地服务 (Python)

#### 2.1 服务器框架
**选择**: FastAPI (异步高性能) + WebSocket

**核心代码结构**:
```python
from fastapi import FastAPI, WebSocket
from fastapi.middleware.cors import CORSMiddleware
import asyncio

app = FastAPI()

# CORS 配置（允许扩展访问）
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# WebSocket 连接管理
class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def broadcast(self, message: dict):
        for connection in self.active_connections:
            await connection.send_json(message)

manager = ConnectionManager()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    manager.active_connections.append(websocket)
    try:
        while True:
            data = await websocket.receive_json()
            await handle_download_request(data)
    except:
        manager.active_connections.remove(websocket)

@app.post("/download")
async def download_video(request: DownloadRequest):
    task_id = create_task(request)
    asyncio.create_task(process_download(task_id))
    return {"task_id": task_id}
```

#### 2.2 视频解析模块
**核心工具**: yt-dlp (最强大的视频下载工具)

**实现策略**:
```python
import yt_dlp

class VideoParser:
    def __init__(self):
        self.ydl_opts = {
            'quiet': True,
            'no_warnings': True,
            'extract_flat': False,
        }

    async def parse(self, url: str, cookies: dict = None) -> dict:
        """解析视频信息"""
        opts = self.ydl_opts.copy()

        # 添加 Cookie
        if cookies:
            opts['cookiefile'] = self._save_cookies(cookies)

        # 平台特定配置
        platform = self._detect_platform(url)
        if platform == 'youtube':
            opts.update({
                'format': 'bestvideo[height<=720]+bestaudio/best[height<=720]',
                'merge_output_format': 'mp4',
            })
        elif platform == 'bilibili':
            opts.update({
                'format': 'bestvideo+bestaudio/best',
            })

        with yt_dlp.YoutubeDL(opts) as ydl:
            info = ydl.extract_info(url, download=False)
            return {
                'title': info['title'],
                'duration': info['duration'],
                'url': info['url'],
                'formats': info['formats'],
            }

    def _detect_platform(self, url: str) -> str:
        """检测平台"""
        if 'youtube.com' in url or 'youtu.be' in url:
            return 'youtube'
        elif 'bilibili.com' in url or 'b23.tv' in url:
            return 'bilibili'
        # ... 更多平台
        return 'unknown'
```

**为什么选择 yt-dlp**:
- 支持 1000+ 网站（YouTube、B站、抖音等）
- 自动处理 DASH 格式合并
- 内置 Cookie 支持
- 活跃维护，更新频繁
- 比 Android 应用的多 API 轮询更稳定

#### 2.3 下载模块
**策略**: yt-dlp 内置下载 + 进度回调

```python
class Downloader:
    async def download(self, url: str, output_path: str,
                      progress_callback=None) -> str:
        """下载视频"""
        ydl_opts = {
            'outtmpl': output_path,
            'progress_hooks': [self._progress_hook],
            'postprocessors': [{
                'key': 'FFmpegVideoConvertor',
                'preferedformat': 'mp4',
            }],
        }

        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])

        return output_path

    def _progress_hook(self, d):
        """进度回调"""
        if d['status'] == 'downloading':
            percent = d.get('_percent_str', '0%')
            speed = d.get('_speed_str', 'N/A')
            eta = d.get('_eta_str', 'N/A')

            # 推送进度到扩展
            asyncio.create_task(manager.broadcast({
                'type': 'progress',
                'percent': percent,
                'speed': speed,
                'eta': eta,
            }))
```

#### 2.4 转录模块 (混合引擎)
**架构**: 策略模式 + 自动降级

```python
from abc import ABC, abstractmethod
from typing import Optional

class TranscriptionEngine(ABC):
    @abstractmethod
    async def transcribe(self, audio_path: str,
                        language: Optional[str] = None) -> dict:
        pass

class FasterWhisperEngine(TranscriptionEngine):
    """本地引擎 - faster-whisper (推荐)"""
    def __init__(self, model_size: str = "base"):
        from faster_whisper import WhisperModel
        self.model = WhisperModel(model_size, device="cpu",
                                  compute_type="int8")

    async def transcribe(self, audio_path: str,
                        language: Optional[str] = None) -> dict:
        segments, info = self.model.transcribe(
            audio_path,
            language=language,
            word_timestamps=True,
        )

        words = []
        text = ""
        for segment in segments:
            text += segment.text + " "
            for word in segment.words:
                words.append({
                    'word': word.word,
                    'start': word.start,
                    'end': word.end,
                })

        return {
            'text': text.strip(),
            'words': words,
            'language': info.language,
        }

class GroqEngine(TranscriptionEngine):
    """在线引擎 - Groq API"""
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.client = openai.OpenAI(
            api_key=api_key,
            base_url="https://api.groq.com/openai/v1"
        )

    async def transcribe(self, audio_path: str,
                        language: Optional[str] = None) -> dict:
        with open(audio_path, 'rb') as f:
            response = self.client.audio.transcriptions.create(
                model="whisper-large-v3-turbo",
                file=f,
                language=language,
                response_format="verbose_json",
                timestamp_granularities=["word"],
            )

        return {
            'text': response.text,
            'words': response.words,
            'language': response.language,
        }

class TranscriptionService:
    """转录服务 - 自动选择引擎"""
    def __init__(self, config: dict):
        self.config = config
        self.engines = {
            'local': FasterWhisperEngine(config.get('model_size', 'base')),
            'online': GroqEngine(config.get('groq_api_key')),
        }

    async def transcribe(self, video_path: str) -> dict:
        """自动选择引擎并转录"""
        # 1. 提取音频
        audio_path = await self._extract_audio(video_path)

        # 2. 快速语言检测（前 30 秒）
        language = await self._detect_language(audio_path)

        # 3. 选择引擎
        engine_type = self._select_engine(audio_path)
        engine = self.engines[engine_type]

        # 4. 转录
        try:
            result = await engine.transcribe(audio_path, language)
            result['engine'] = engine_type
            return result
        except Exception as e:
            # 降级到备用引擎
            if engine_type == 'local':
                print(f"本地引擎失败，降级到在线引擎: {e}")
                result = await self.engines['online'].transcribe(
                    audio_path, language
                )
                result['engine'] = 'online'
                return result
            else:
                raise

    def _select_engine(self, audio_path: str) -> str:
        """选择引擎策略"""
        mode = self.config.get('transcription_mode', 'auto')

        if mode == 'local':
            return 'local'
        elif mode == 'online':
            return 'online'
        else:  # auto
            # 文件大小 > 100MB 或无网络 -> 本地
            file_size = os.path.getsize(audio_path)
            if file_size > 100 * 1024 * 1024:
                return 'local'

            # 检查网络
            if not self._check_network():
                return 'local'

            # 默认在线（更快）
            return 'online'

    async def _detect_language(self, audio_path: str) -> str:
        """快速语言检测（前 30 秒）"""
        # 提取前 30 秒
        sample_path = await self._extract_sample(audio_path, duration=30)

        # 使用本地引擎快速检测
        result = await self.engines['local'].transcribe(sample_path)

        return result['language']
```

**引擎对比**:

| 引擎 | 速度 | 准确度 | 成本 | 网络 | 推荐场景 |
|------|------|--------|------|------|---------|
| faster-whisper | 4x 实时 | 高 | 免费 | 无需 | 大文件、离线 |
| Groq API | 1-2x 实时 | 高 | 按次 | 必需 | 小文件、快速 |
| whisper.cpp | 2-3x 实时 | 高 | 免费 | 无需 | 低内存设备 |
| Moonshine | 实时 | 中 | 免费 | 无需 | 边缘设备 |

---

### 3. 自动启动与进程管理

#### 3.1 Windows 服务包装 (NSSM)
**安装步骤**:
```bash
# 1. 下载 NSSM
# https://nssm.cc/download

# 2. 安装服务
nssm install VideoDownloadService "C:\Python\python.exe" "C:\path\to\server.py"

# 3. 配置服务
nssm set VideoDownloadService AppDirectory "C:\path\to"
nssm set VideoDownloadService DisplayName "视频下载转录服务"
nssm set VideoDownloadService Description "浏览器扩展后端服务"
nssm set VideoDownloadService Start SERVICE_AUTO_START

# 4. 启动服务
nssm start VideoDownloadService
```

#### 3.2 看门狗机制
```python
import psutil
import time

class Watchdog:
    """进程监控和自动重启"""
    def __init__(self, process_name: str):
        self.process_name = process_name

    def monitor(self):
        while True:
            if not self._is_running():
                print(f"{self.process_name} 已停止，正在重启...")
                self._restart()
            time.sleep(10)

    def _is_running(self) -> bool:
        for proc in psutil.process_iter(['name']):
            if proc.info['name'] == self.process_name:
                return True
        return False

    def _restart(self):
        os.system(f"nssm restart VideoDownloadService")
```

---

## 关键文件清单

### 浏览器扩展
```
video-downloader-extension/
├── manifest.json              # 扩展配置
├── background.js              # 后台脚本
├── content/
│   ├── youtube.js            # YouTube 注入脚本
│   ├── bilibili.js           # B站 注入脚本
│   └── common.js             # 通用工具
├── popup/
│   ├── popup.html            # 弹出界面
│   ├── popup.js              # 界面逻辑
│   └── popup.css             # 样式
└── icons/                     # 图标资源
```

### 本地服务
```
video-download-service/
├── server.py                  # FastAPI 服务器
├── parsers/
│   ├── base.py               # 解析器基类
│   ├── ytdlp_parser.py       # yt-dlp 解析器
│   └── cookie_manager.py     # Cookie 管理
├── downloaders/
│   ├── base.py               # 下载器基类
│   └── ytdlp_downloader.py   # yt-dlp 下载器
├── transcription/
│   ├── base.py               # 转录引擎基类
│   ├── faster_whisper.py     # faster-whisper 引擎
│   ├── groq.py               # Groq API 引擎
│   └── language_detector.py  # 语言检测
├── storage/
│   ├── database.py           # SQLite 数据库
│   └── file_manager.py       # 文件管理
├── utils/
│   ├── ffmpeg.py             # FFmpeg 工具
│   └── progress.py           # 进度管理
├── config.yaml               # 配置文件
└── requirements.txt          # Python 依赖
```

---

## 实施步骤

### Phase 1: 浏览器扩展开发 (2-3 天)
1. 创建扩展基础结构（manifest.json）
2. 实现 Content Script（YouTube + B站）
3. 实现 Background Service Worker
4. 实现 Popup UI
5. 测试链接提取和通信

### Phase 2: 本地服务开发 (3-4 天)
1. 搭建 FastAPI 服务器框架
2. 集成 yt-dlp 解析和下载
3. 实现 WebSocket 通信
4. 实现任务队列和进度推送
5. 测试下载功能

### Phase 3: 转录引擎集成 (2-3 天)
1. 集成 faster-whisper 本地引擎
2. 集成 Groq API 在线引擎
3. 实现语言检测
4. 实现引擎自动选择和降级
5. 测试转录功能

### Phase 4: 系统集成与优化 (2 天)
1. 扩展与服务端到端测试
2. 实现 NSSM 服务包装
3. 实现看门狗机制
4. 性能优化和错误处理
5. 用户文档编写

### Phase 5: 部署与测试 (1 天)
1. 打包扩展（.crx）
2. 安装本地服务
3. 完整流程测试
4. 修复 bug

---

## 验证方案

### 端到端测试流程
1. **安装扩展**: 加载到 Chrome/Edge
2. **启动服务**: 运行本地服务（或安装为 Windows 服务）
3. **测试 YouTube**:
   - 打开 YouTube 视频页面
   - 点击注入的下载按钮
   - 验证视频下载成功
   - 验证自动转录（英文）
4. **测试 B站**:
   - 打开 B站视频页面
   - 点击下载按钮
   - 验证 DASH 格式处理
   - 验证自动转录（中文）
5. **测试引擎切换**:
   - 在 Popup 中切换到本地引擎
   - 下载并转录视频
   - 验证 faster-whisper 工作正常
6. **测试降级机制**:
   - 断开网络
   - 下载视频（应自动使用本地引擎）
   - 验证转录成功

### 性能指标
- YouTube 解析成功率: > 95%
- B站解析成功率: > 98%
- 下载速度: 接近带宽上限
- 转录速度（本地）: 4x 实时
- 转录速度（在线）: 1-2x 实时
- 语言检测准确率: > 90%

---

## 技术决策理由

| 决策 | 理由 |
|------|------|
| **yt-dlp** | 最强大的视频下载工具，支持 1000+ 网站，比多 API 轮询更稳定 |
| **FastAPI** | 异步高性能，原生 WebSocket 支持，易于开发 |
| **faster-whisper** | 4x 速度提升，准确度不变，Python 生态友好 |
| **WebSocket** | 实时双向通信，低延迟，适合进度推送 |
| **NSSM** | 轻量级服务包装器，易于安装和管理 |
| **混合引擎** | 在线快速，本地离线，自动降级保证可用性 |

---

## 风险与缓解

| 风险 | 缓解措施 |
|------|---------|
| YouTube 反爬虫 | yt-dlp 持续更新，社区活跃 |
| 扩展审核不通过 | 提供开发者模式安装说明 |
| 本地服务崩溃 | NSSM 自动重启 + 看门狗监控 |
| 转录引擎失败 | 自动降级到备用引擎 |
| 网络不稳定 | 断点续传 + 重试机制 |

---

## 依赖项

### Python 依赖 (requirements.txt)
```
fastapi==0.115.0
uvicorn[standard]==0.32.0
websockets==13.1
yt-dlp==2024.12.13
faster-whisper==1.1.0
openai==1.59.5
ffmpeg-python==0.2.0
psutil==6.1.0
pyyaml==6.0.2
sqlalchemy==2.0.36
```

### 系统依赖
- Python 3.10+
- FFmpeg (音视频处理)
- NSSM (Windows 服务管理)
- Chrome/Edge 浏览器

---

## 后续优化方向

1. **支持更多平台**: 抖音、TikTok、小红书等
2. **批量下载**: 支持播放列表和频道
3. **字幕翻译**: 集成翻译 API
4. **云同步**: 支持多设备同步
5. **移动端**: 开发 Android/iOS 应用
