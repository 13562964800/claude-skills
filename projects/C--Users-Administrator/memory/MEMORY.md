# 项目记忆

## 工作流程标准

### 打包完成后的标准操作 ⭐
- **触发条件**: 完成任何安装包、压缩包、发布包的创建
- **标准动作**: 立即打开安装包所在文件夹
- **命令模式**:
  - Windows: `cmd.exe /c start "" "<文件夹路径>"`
  - 或: `explorer.exe "<文件夹路径>"`
- **适用场景**:
  - Electron 打包 (electron-builder)
  - Python 打包 (PyInstaller)
  - 压缩包创建 (zip/7z)
  - 任何构建产物生成
- **目的**: 方便用户立即查看和使用生成的文件
- **执行时机**: 打包命令成功完成后立即执行

### 配置文件更新规则
- **全局配置**: `~/.claude/` 或系统级配置
- **本地配置**: 项目根目录的配置文件
- **原则**: 同时更新全局和本地配置，确保一致性

## 项目信息

### LegalFlow Studio (法律智能生产中台)
- **项目路径**: `C:\Users\Administrator\legalflow-studio\`
- **安装包位置**: `C:\Users\Administrator\legalflow-studio\build\`
- **最新版本**: LegalFlow Studio Setup 0.5.0.exe (158 MB)
- **技术栈**: FastAPI + Next.js 15 + Electron
- **特性**:
  - 用户可在界面配置 API Keys (国内外顶尖三家)
  - 支持多模型 LLM 路由
  - 4个法律工作流模板
- **状态**: 已完成打包

### 视频下载转录系统
- **项目路径**: `~/video-downloader-extension/` 和 `~/video-download-service/`
- **安装包位置**: `~/VideoDownloadTranscribe-Setup-v1.0.0-Installer.zip`
- **版本**: v1.0.0
- **状态**: 已完成开发、测试和打包

### 技术栈
- 前端: JavaScript (Chrome Extension) / Next.js 15
- 后端: Python + FastAPI
- 视频处理: yt-dlp + FFmpeg
- 转录: faster-whisper + Groq API
- 桌面应用: Electron 28

### 依赖安装
- 使用清华镜像源: `pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple`
- 核心依赖: fastapi, uvicorn, yt-dlp, faster-whisper, openai

## 用户偏好

### 操作习惯
- **打包完成后立即打开文件夹** (最重要)
- 不询问确认，直接执行操作
- 使用最快的下载源（清华镜像）
- 遇到选择时永远选择第一个选项

### 系统环境
- 操作系统: Windows 11
- Shell: bash
- Python: 3.14
- FFmpeg: 8.0.1
