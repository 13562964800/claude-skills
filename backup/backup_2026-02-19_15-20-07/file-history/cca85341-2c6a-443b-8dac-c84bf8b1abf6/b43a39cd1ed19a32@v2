# 法律期限速查系统 - 打包 EXE 和 APK

## Context
用户有一个纯前端的法律期限速查系统（单个 HTML 文件），希望将其打包为 Windows EXE 桌面应用和 Android APK。技术方案：Electron（EXE）+ Capacitor（APK）。

## 步骤

### 1. 创建项目结构
在 `C:/Users/Administrator/Desktop/LegalDeadlineApp/` 下初始化 npm 项目，安装 Electron 和 Capacitor 依赖。

### 2. Electron 桌面版（EXE）
- 创建 `main.js`（Electron 主进程），加载现有 HTML 文件
- 配置 `package.json` 的 Electron 相关字段
- 安装 `electron` 和 `electron-builder`
- 使用 `electron-builder` 打包为 Windows EXE 安装包

### 3. Capacitor 移动版（APK）
- 安装 `@capacitor/core` 和 `@capacitor/cli`
- 初始化 Capacitor 项目，将 HTML 作为 web 资源
- 添加 Android 平台
- 使用 Gradle 构建 APK（需要 Android SDK，如未安装会提供指引）

### 4. 验证
- 运行 `npx electron .` 测试桌面版
- 检查 EXE 输出目录
- 检查 APK 输出目录

## 关键文件
- `C:/Users/Administrator/Desktop/LegalDeadlineApp/法律期限速查系统.html` — 现有主应用
- 新建 `main.js` — Electron 主进程入口
- 新建 `package.json` — 项目配置
- 新建 `capacitor.config.ts` — Capacitor 配置
