# CourtAssistant Android App - 实现计划

## Context
开发一款法院文档管理 Android 应用，核心功能：自动读取法院短信 → 解析案件信息 → WebView 打开链接下载 PDF → 文档管理展示 → 日历同步。技术栈：Kotlin + Jetpack Compose。项目路径：`C:\Users\Administrator\Desktop\CourtAssistant`

## 架构设计

### 技术选型
- **语言**: Kotlin
- **UI**: Jetpack Compose + Material3
- **数据库**: Room (本地持久化)
- **PDF解析**: iText (文本提取) + ML Kit OCR (图片型PDF备选)
- **日历**: Android CalendarProvider API
- **WebView**: AndroidX WebView，拦截 PDF 下载
- **权限**: SMS、Calendar、Storage

### 数据模型

```
CourtSms (短信记录)
├── id, sender, body, receivedAt
├── caseNumber (案号)
├── courtName (法院简称)
├── hearingTime (开庭时间)
├── partyNames (当事人名称)
├── webUrl (网页链接)
└── calendarEventId (日历事件ID，去重用)

PdfDocument (PDF文档)
├── id, smsId (关联短信)
├── fileName, filePath, fileSize
├── downloadedAt
├── isSubpoena (是否传票)
├── hearingInfo (从传票提取的开庭信息)
└── calendarEventId (日历事件ID，去重用)
```

### 模块划分

```
com.court.assistant
├── data/
│   ├── db/          # Room 数据库、DAO、Entity
│   ├── repository/  # Repository 层
│   └── model/       # 数据模型
├── sms/
│   ├── SmsReader.kt          # 读取法院短信
│   └── SmsParser.kt          # 正则解析案号、法院、开庭时间、当事人
├── pdf/
│   ├── PdfDownloadManager.kt # PDF 下载管理
│   └── PdfParser.kt          # iText文本提取 + ML Kit OCR
├── calendar/
│   └── CalendarSync.kt       # 日历同步（含去重）
├── webview/
│   └── CourtWebViewClient.kt # WebView 拦截 PDF 下载
├── ui/
│   ├── theme/       # 宋氏美学主题
│   ├── MainScreen.kt         # 主界面（文档管理）
│   ├── DocumentCard.kt       # PDF 卡片组件
│   ├── SmsMenuCard.kt        # 短信菜单（可展开）
│   ├── WebViewScreen.kt      # WebView 页面
│   └── PdfPreviewScreen.kt   # PDF 预览
├── viewmodel/
│   └── MainViewModel.kt      # 主 ViewModel
└── MainActivity.kt
```

## 实现步骤

### Step 1: 项目初始化
- 用标准 Android 项目结构创建 Gradle 项目
- 配置 build.gradle：Compose、Room、iText、ML Kit 依赖
- AndroidManifest：声明 SMS、Calendar、Storage 权限

### Step 2: 数据层
- Room Entity: `CourtSmsEntity`, `PdfDocumentEntity`
- DAO: 增删改查，按时间排序
- Repository: 统一数据访问

### Step 3: 短信读取与解析
- `SmsReader`: 请求 READ_SMS 权限，读取 ContentResolver 中法院相关短信
  - 过滤关键词：`法院`、`开庭`、`传票`、`案件`、`送达`、`court.gov.cn`
- `SmsParser`: 基于实际短信格式解析，样本：
  ```
  【临沂市河东区人民法院】单夫纯你好，请查收（2026）鲁1312民初1324号案件中你的送达文书。
  点击链接查阅：https://zxfw.court.gov.cn/zxfw/#/pagesAjkj/app/wssd/index?qdbh=xxx&sdbh=xxx&sdsin=xxx
  ```
  - 法院名称: `【(.+?法院)】` → 提取简称（去掉省市前缀，如"河东区人民法院"→"河东法院"）
  - 当事人: `】(\S+?)你好` → 紧跟法院名称后的姓名
  - 案号: `[（(]\d{4}[）)][^\s]+号` → 如 `（2026）鲁1312民初1324号`
  - URL: `https?://[^\s]+` → 法院送达链接
  - 开庭时间: `\d{4}年\d{1,2}月\d{1,2}日[\s\S]*?\d{1,2}[时点:]?\d{0,2}分?` 等多种格式
- 解析后自动存入 Room

### Step 4: 日历同步
- `CalendarSync`:
  - 查询/创建 "法院日程" 日历账户
  - 插入事件前先查询是否已存在（案号+开庭时间去重）
  - 事件标题: `[案号] 法院名称 开庭`
  - 事件描述: 当事人信息

### Step 5: WebView + PDF 下载
- `WebViewScreen`: 打开 `zxfw.court.gov.cn` 等法院送达链接
  - 目标平台: 中国法院诉讼服务网电子送达系统
  - 页面会列出多个送达文书（传票、起诉状、举证通知书等），每个文书有独立的查看/下载按钮
  - **交互流程**:
    1. WebView 顶部显示提示条："请依次点击文书下载，下载完成会自动保存"
    2. 用户在网页中点击各文书的下载链接
    3. 拦截下载事件，自动保存 PDF
    4. 每下载一个，底部 Snackbar 提示 "已保存: xxx.pdf"
    5. 用户点击返回时，显示已下载文件数量确认
  - **拦截策略** (三层拦截):
    1. `setDownloadListener`: 拦截浏览器触发的下载（Content-Disposition: attachment）
    2. `shouldInterceptRequest`: 拦截 URL 以 `.pdf` 结尾的请求
    3. `shouldOverrideUrlLoading`: 拦截跳转到 PDF 的 URL
  - **下载实现**: 使用 OkHttp 携带 WebView 的 Cookie 下载（法院平台需要登录态）
  - **文件命名**: 优先使用 Content-Disposition 中的 filename，其次用网页中的文书标题
- PDF 存储到 `app外部存储/Documents/CourtAssistant/` 目录

### Step 6: PDF 解析
- `PdfParser`:
  - 先用 iText 提取文本
  - 若文本为空，用 ML Kit OCR
  - 文件名含"传票"时，提取开庭时间、案号等，同步到日历

### Step 7: 文档管理 UI
- `MainScreen`:
  - 按短信接收时间倒序排列
  - 每条短信 = 一个可展开菜单
  - 菜单头部: 短信接收时间 | 当事人名称 | 法院简称 | 案号
  - 点击展开 → 显示该短信关联的所有 PDF 卡片
  - PDF 卡片名称和数量与网页一致（从 WebView 下载时记录原始文件名）
  - 卡片去重: 按文件名 + smsId 唯一约束
- `PdfPreviewScreen`: 点击卡片预览 PDF，显示解析的开庭信息

### Step 8: 主题 - 宋氏美学
- 配色: 素雅中式风格（米白底、深灰文字、朱红点缀）
- 字体: 衬线体风格
- 底部标注: "宋氏美学，单夫纯律师制作保留权利"

## 关键难点处理

### PDF 下载交互（核心难点）
- **提示引导**: WebView 顶部固定提示条，告知用户"请点击文书下载，完成后自动保存"
- **三层拦截**: DownloadListener + shouldInterceptRequest + shouldOverrideUrlLoading 确保不遗漏
- **Cookie 同步**: 法院平台需登录态，下载时从 CookieManager 获取 Cookie 传给 OkHttp
- **并发控制**: 同一时间只允许一个下载任务，避免重复触发
- **下载状态**: 每个 PDF 下载完成后 Snackbar 提示文件名，WebView 页面底部显示"已下载 N 个文件"计数器
- **返回确认**: 用户按返回键时，若有已下载文件，显示确认对话框列出已下载文件

### 去重机制
- **文档去重**: Room 中 `(smsId, fileName)` 联合唯一索引，插入时 `ON CONFLICT IGNORE`
- **日历去重**: 插入前查询 CalendarProvider，匹配 `案号 + 开庭时间`，已存在则跳过
- **短信去重**: 按短信 body 的 hash 值去重，避免重复导入同一条短信

## 验证方式
1. 安装到 Android 设备，授予 SMS/Calendar 权限
2. 确认能读取并解析法院短信
3. 点击链接在 WebView 中打开，下载 PDF
4. 确认 PDF 出现在文档管理界面对应短信下
5. 确认传票 PDF 的开庭信息同步到系统日历
6. 确认重复操作不会产生重复记录
