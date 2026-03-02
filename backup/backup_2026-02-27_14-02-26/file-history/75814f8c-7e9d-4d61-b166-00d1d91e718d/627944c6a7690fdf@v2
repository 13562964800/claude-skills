# CourtAssistant - 法院文档管理应用实现计划

## Context

开发一款 Android 应用，用于法院文档管理和开庭信息同步。应用需要自动读取法院相关短信、解析关键信息（开庭时间、案号、法院名称等）、下载传票 PDF，并支持同步到系统日历。

**技术选择**：Java + XML，最低 Android 8.0+ (API 26)，仅本地存储

---

## 架构设计

### 项目结构
```
CourtAssistant/
├── app/src/main/
│   ├── java/com/courassistant/
│   │   ├── model/              # 数据模型
│   │   │   ├── CourtCase.java      # 案件信息
│   │   │   ├── CourtMessage.java   # 短信信息
│   │   │   ├── CourtDocument.java  # 文档信息
│   │   │   └── CalendarEvent.java   # 日历事件
│   │   ├── receiver/           # 广播接收器
│   │   │   └── SmsReceiver.java     # 短信监听
│   │   ├── parser/             # 解析器
│   │   │   ├── SmsParser.java       # 短信解析
│   │   │   ├── CaseInfoExtractor.java # 案件信息提取
│   │   │   └── PdfParser.java       # PDF解析
│   │   ├── service/            # 服务
│   │   │   ├── SmsMonitorService.java  # 短信监控服务
│   │   │   └── CalendarSyncService.java # 日历同步服务
│   │   ├── ui/                 # 界面
│   │   │   ├── MainActivity.java
│   │   │   ├── DocumentListAdapter.java
│   │   │   ├── PdfWebViewActivity.java
│   │   │   └── PdfViewerActivity.java
│   │   ├── utils/              # 工具类
│   │   │   ├── CalendarUtils.java    # 日历工具
│   │   │   ├── FileUtils.java        # 文件工具
│   │   │   ├── PermissionUtils.java  # 权限工具
│   │   │   └── DeduplicationHelper.java # 去重工具
│   │   ├── database/           # 数据库
│   │   │   ├── CourtDatabase.java
│   │   │   ├── CourtCaseDao.java
│   │   │   ├── CourtMessageDao.java
│   │   │   └── CourtDocumentDao.java
│   │   └── repository/         # 仓库层
│   │       ├── CaseRepository.java
│   │       └── DocumentRepository.java
│   ├── res/
│   │   ├── layout/
│   │   │   ├── activity_main.xml
│   │   │   ├── item_document_header.xml
│   │   │   ├── item_pdf_card.xml
│   │   │   ├── activity_webview.xml
│   │   │   └── activity_pdf_viewer.xml
│   │   ├── values/
│   │   │   ├── colors.xml          # 宋氏美学配色
│   │   │   ├── strings.xml
│   │   │   └── styles.xml
│   │   └── drawable/
│   └── AndroidManifest.xml
```

---

## 详细实现步骤

### 第一阶段：项目基础搭建

#### 1. 创建 Android 项目
- minSdkVersion: 26 (Android 8.0)
- targetSdkVersion: 35
- 使用 Java 语言
- 依赖库：
  - Room 数据库
  - AndroidX 核心库
  - WebView 支持

#### 2. 配置 AndroidManifest.xml
**权限声明**：
```xml
<!-- 短信读取权限 -->
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.RECEIVE_SMS" />

<!-- 日历读写权限 -->
<uses-permission android:name="android.permission.READ_CALENDAR" />
<uses-permission android:name="android.permission.WRITE_CALENDAR" />

<!-- 存储权限 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<!-- 网络权限（用于WebView加载） -->
<uses-permission android:name="android.permission.INTERNET" />
```

**注册组件**：
- SmsReceiver 广播接收器
- SmsMonitorService 前台服务

---

### 第二阶段：核心功能实现

#### 1. 短信监听与读取 (SmsReceiver + SmsMonitorService)

**文件**: `SmsReceiver.java`, `SmsMonitorService.java`

**功能**：
- 监听系统短信广播
- 过滤法院相关短信（关键词：法院、开庭、传票、案号等）
- 将短信内容传递给解析器

**实现要点**：
```java
// SmsReceiver.java
public class SmsReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 获取短信内容
        // 过滤法院相关关键词
        // 启动SmsMonitorService处理
    }
}

// 法院关键词
private static final String[] COURT_KEYWORDS = {
    "法院", "开庭", "传票", "案号", "诉讼", "审判",
    "中级人民法院", "基层人民法院", "高级人民法院"
};
```

#### 2. 短信解析 (SmsParser + CaseInfoExtractor)

**文件**: `SmsParser.java`, `CaseInfoExtractor.java`

**解析规则**：
```java
// 案号正则： (2024)京01民初123号
Pattern caseNoPattern = Pattern.compile("\\(\\d{4}\\)[^\\s]+?[民刑行]初\\d+号");

// 开庭时间正则： 2024年3月15日上午9:30
Pattern datePattern = Pattern.compile(
    "(\\d{4})年(\\d{1,2})月(\\d{1,2})日[上下]午(\\d{1,2}):(\\d{2})"
);

// 法院名称提取
// 当事人名称提取
```

**提取的信息**：
- 案号 (caseNo)
- 法院简称 (courtShortName)
- 开庭时间 (hearingDate)
- 当事人名称 (partyName)
- 网页链接 (url)

#### 3. 日历同步 (CalendarSyncService + CalendarUtils)

**文件**: `CalendarSyncService.java`, `CalendarUtils.java`

**功能**：
- 读取所有可用日历账户
- 创建日历事件
- **去重机制**：使用 案号+开庭时间 作为唯一标识

**去重实现**：
```java
// CalendarUtils.java
public static boolean isEventExists(ContentResolver resolver, String caseNo, long startTime) {
    // 查询已存在的事件
    // 比较案号（在事件标题中）和开始时间
    // 返回是否存在
}

public static long addCourtEvent(Context context, CourtCase courtCase) {
    // 检查是否已存在
    // 如果不存在，创建新事件
    // 事件标题格式："[案号] 当事人 - 开庭"
}
```

#### 4. WebView PDF下载交互 (PdfWebViewActivity)

**文件**: `PdfWebViewActivity.java`, `activity_webview.xml`

**关键交互设计**：

**界面布局**：
- 顶部：提示文字 "点击PDF链接下载文档"
- 中间：WebView 显示网页
- 底部：下载进度条（下载时显示）
- 下载完成后显示："已下载X份文档，点击返回查看"

**交互流程**：
```java
// 1. 拦截URL加载
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (url.endsWith(".pdf") || url.contains("pdf")) {
            // 显示下载确认对话框
            showDownloadDialog(url);
            return true;
        }
        return false;
    }
});

// 2. 下载管理
private void downloadPdf(String url) {
    // 使用DownloadManager
    // 监听下载完成
    // 获取文件路径
    // 保存到数据库
    // 去重处理
}

// 3. 用户引导
private void highlightPdfLinks() {
    // 注入JS高亮PDF链接
    // 显示点击提示
}
```

**时机管控**：
- 只在用户明确点击后下载
- 显示下载确认对话框
- 下载完成后显示成功提示
- 返回后自动刷新文档列表

#### 5. PDF解析与文档管理

**文件**: `PdfParser.java`, `MainActivity.java`, `DocumentListAdapter.java`

**PDF解析**（文件名包含"传票"时）：
```java
// PdfParser.java
public CourtCase parseSummonsPdf(String filePath) {
    // 提取PDF文本
    // 解析案号、开庭时间、法院、当事人
    // 返回CourtCase对象
}
```

**文档列表界面**：
```
┌─────────────────────────────────────────┐
│ 📅 2024-03-15 09:30                    │
│ 👤 张三 vs 李四                         │
│ 🏛️ 北京市第一中级人民法院                │
│ 📋 (2024)京01民初123号                 │
│ ▼ (点击展开PDF，共3份)                  │
├─────────────────────────────────────────┤
│   ┌─────────────────────────────────┐   │
│   │ 📄 开庭传票.pdf                  │   │
│   │    2.3 MB  ·  2024-03-10下载    │   │
│   └─────────────────────────────────┘   │
│   ┌─────────────────────────────────┐   │
│   │ 📄 应诉通知书.pdf                │   │
│   │    1.5 MB  ·  2024-03-10下载    │   │
│   └─────────────────────────────────┘   │
│   ┌─────────────────────────────────┐   │
│   │ 📄 证据清单.pdf                  │   │
│   │    856 KB ·  2024-03-10下载     │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**去重机制**：
```java
// DeduplicationHelper.java
public class DeduplicationHelper {

    // 短信去重：基于短信内容和时间
    public static boolean isDuplicateSms(CourtMessage newMsg, List<CourtMessage> existing) {
        for (CourtMessage msg : existing) {
            if (msg.getContent().equals(newMsg.getContent()) &&
                Math.abs(msg.getReceiveTime() - newMsg.getReceiveTime()) < 5000) {
                return true;
            }
        }
        return false;
    }

    // 文档去重：基于文件MD5或文件名+大小
    public static boolean isDuplicateDocument(File newFile, List<CourtDocument> existing) {
        for (CourtDocument doc : existing) {
            if (doc.getFileName().equals(newFile.getName()) &&
                doc.getFileSize() == newFile.length()) {
                return true;
            }
        }
        return false;
    }

    // 日历去重：基于案号+开庭时间
    public static boolean isDuplicateCalendarEvent(CourtCase newCase, List<CourtCase> existing) {
        for (CourtCase c : existing) {
            if (c.getCaseNo().equals(newCase.getCaseNo()) &&
                c.getHearingDate().equals(newCase.getHearingDate())) {
                return true;
            }
        }
        return false;
    }
}
```

---

### 第三阶段：数据库设计

**文件**: `CourtDatabase.java`, 各Dao类

**数据表**：

```sql
-- 法院案件表
CREATE TABLE court_cases (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    case_no TEXT UNIQUE NOT NULL,           -- 案号
    court_full_name TEXT,                    -- 法院全称
    court_short_name TEXT,                   -- 法院简称
    party_name TEXT,                         -- 当事人名称
    hearing_date INTEGER,                    -- 开庭时间戳
    sms_id TEXT,                             -- 关联的短信ID
    calendar_event_id INTEGER,               -- 关联的日历事件ID
    created_at INTEGER,
    updated_at INTEGER
);

-- 短信记录表
CREATE TABLE court_messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    sender TEXT,                             -- 发送者号码
    content TEXT,                            -- 短信内容
    receive_time INTEGER,                    -- 接收时间
    is_parsed INTEGER DEFAULT 0,             -- 是否已解析
    parsed_case_no TEXT,                     -- 解析出的案号
    has_url INTEGER DEFAULT 0,               -- 是否包含链接
    created_at INTEGER
);

-- 文档记录表
CREATE TABLE court_documents (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    case_id INTEGER,                          -- 关联的案件ID
    file_name TEXT NOT NULL,                  -- 文件名
    file_path TEXT NOT NULL,                  -- 文件路径
    file_size INTEGER,                        -- 文件大小
    file_type TEXT,                           -- 文件类型
    download_url TEXT,                        -- 下载来源URL
    is_summons INTEGER DEFAULT 0,             -- 是否为传票
    download_time INTEGER,                    -- 下载时间
    created_at INTEGER,
    FOREIGN KEY (case_id) REFERENCES court_cases(id) ON DELETE CASCADE
);
```

---

### 第四阶段：UI设计 - 宋氏美学

**配色方案** (`colors.xml`):
```xml
<!-- 主色调：典雅青灰 -->
<color name="primary">#2C3E50</color>
<color name="primary_dark">#1A252F</color>
<color name="primary_light">#34495E</color>

<!-- 强调色：朱红（用于重要标记） -->
<color name="accent">#C0392B</color>
<color name="accent_light">#E74C3C</color>

<!-- 背景色：米白 -->
<color name="background">#FAF8F5</color>
<color name="card_background">#FFFFFF</color>

<!-- 文字色：墨黑 -->
<color name="text_primary">#2C2C2C</color>
<color name="text_secondary">#666666</color>
<color name="text_hint">#999999</color>

<!-- 分割线：浅灰 -->
<color name="divider">#E0E0E0</color>

<!-- 功能色 -->
<color name="success">#27AE60</color>
<color name="warning">#F39C12</color>
<color name="error">#C0392B</color>
```

**样式定义** (`styles.xml`):
```xml
<!-- 应用主题 -->
<style name="CourtAssistantTheme" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
    <item name="colorPrimary">@color/primary</item>
    <item name="colorPrimaryDark">@color/primary_dark</item>
    <item name="colorAccent">@color/accent</item>
    <item name="android:windowBackground">@color/background</item>
</style>

<!-- 文档卡片样式 -->
<style name="DocumentCardStyle">
    <item name="cardCornerRadius">8dp</item>
    <item name="cardElevation">2dp</item>
    <item name="cardBackgroundColor">@color/card_background</item>
</style>
```

**布局特点**：
- 简洁的卡片式设计
- 清晰的层次结构
- 适当的留白
- 优雅的字体排版

---

### 第五阶段：权限管理

**文件**: `PermissionUtils.java`, `MainActivity.java`

**权限请求流程**：
```java
// 运行时权限请求
private static final int PERMISSION_REQUEST_CODE = 1001;

private void requestPermissions() {
    String[] permissions = {
        Manifest.permission.READ_SMS,
        Manifest.permission.RECEIVE_SMS,
        Manifest.permission.READ_CALENDAR,
        Manifest.permission.WRITE_CALENDAR,
        Manifest.permission.READ_EXTERNAL_STORAGE,
        Manifest.permission.WRITE_EXTERNAL_STORAGE
    };

    // 分组请求，逐步引导用户授权
}

// 权限结果处理
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    // 处理每个权限的结果
    // 对于被拒绝的权限，显示说明为何需要的对话框
}
```

---

## 实现顺序

### 1. 项目初始化 (第1步)
- 创建Android项目结构
- 配置build.gradle依赖
- 配置AndroidManifest.xml

### 2. 数据库层 (第2步)
- 创建Room数据库
- 定义Entity、Dao、Database
- 实现Repository层

### 3. 短信监听与解析 (第3步)
- 实现SmsReceiver
- 实现SmsParser和CaseInfoExtractor
- 实现去重逻辑

### 4. 日历同步 (第4步)
- 实现CalendarUtils
- 实现CalendarSyncService
- 完成日历去重

### 5. UI界面 (第5步)
- 实现MainActivity布局
- 实现DocumentListAdapter
- 实现可展开/折叠的文档列表

### 6. WebView下载 (第6步)
- 实现PdfWebViewActivity
- 实现PDF下载拦截
- 实现下载完成后的数据处理

### 7. PDF解析 (第7步)
- 集成PDF解析库
- 实现传票信息提取
- 关联到日历

### 8. 优化与测试 (第8步)
- 完善宋氏美学UI
- 端到端测试
- 性能优化

---

## 依赖库

**build.gradle**:
```gradle
dependencies {
    // AndroidX Core
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'

    // Material Design
    implementation 'com.google.android.material:material:1.11.0'

    // Room Database
    implementation 'androidx.room:room-runtime:2.6.1'
    annotationProcessor 'androidx.room:room-compiler:2.6.1'

    // WebView
    implementation 'androidx.webkit:webkit:1.10.0'

    // PDF解析
    implementation 'com.tom-roush:pdfbox-android:2.0.27.0'

    // 文件选择器
    implementation 'androidx.documentfile:documentfile:1.0.1'

    // Lifecycle
    implementation 'androidx.lifecycle:lifecycle-runtime:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
}
```

---

## 验证测试计划

### 1. 短信监听测试
- 发送测试短信包含法院关键词
- 验证短信被正确捕获
- 验证非法院短信被过滤

### 2. 解析准确性测试
- 测试各种格式的案号
- 测试各种日期时间格式
- 测试不同法院名称格式

### 3. 日历同步测试
- 验证事件正确创建
- 验证去重功能有效
- 测试多个日历账户

### 4. WebView下载测试
- 测试PDF链接点击
- 验证下载完成回调
- 验证文件保存路径

### 5. 去重功能测试
- 重复短信处理
- 重复文档处理
- 重复日历事件处理

### 6. UI交互测试
- 展开/折叠文档列表
- PDF预览功能
- 删除文档功能

---

## 版权信息

**应用名称**: CourtAssistant
**制作者**: 宋氏美学，单夫纯律师
**保留权利**: © 2024 All Rights Reserved

在应用的 about 页面和启动页面显示：
```
CourtAssistant
宋氏美学 · 单夫纯律师 制作
保留所有权利
© 2024
```

---

## 关键文件清单

### 核心文件
1. `app/src/main/AndroidManifest.xml` - 权限和组件声明
2. `app/src/main/java/com/courassistant/database/CourtDatabase.java` - 数据库
3. `app/src/main/java/com/courassistant/receiver/SmsReceiver.java` - 短信监听
4. `app/src/main/java/com/courassistant/parser/SmsParser.java` - 短信解析
5. `app/src/main/java/com/courassistant/ui/MainActivity.java` - 主界面
6. `app/src/main/java/com/courassistant/ui/PdfWebViewActivity.java` - WebView下载
7. `app/src/main/java/com/courassistant/utils/CalendarUtils.java` - 日历工具
8. `app/src/main/java/com/courassistant/utils/DeduplicationHelper.java` - 去重工具

### 资源文件
1. `app/src/main/res/layout/activity_main.xml` - 主界面布局
2. `app/src/main/res/layout/item_document_header.xml` - 文档头部项
3. `app/src/main/res/layout/item_pdf_card.xml` - PDF卡片项
4. `app/src/main/res/values/colors.xml` - 宋氏美学配色
5. `app/src/main/res/values/strings.xml` - 字符串资源
