# 从微信聊天记录导入文件

## Context

用户需要完善"从微信导入"功能。当前已有基础的 Share Intent 接收，但存在多个缺陷导致实际使用中不可靠。参考 GitHub 研究报告（receive_sharing_intent 等仓库的最佳实践），需要修复和增强。

## 现有实现

| 组件 | 文件 | 状态 |
|------|------|------|
| Intent-filter | `AndroidManifest.xml` | ✅ 有，但缺 launchMode |
| Java Intent 处理 | `MainActivity.java:176-297` | ✅ 有 onNewIntent + handleShareIntent，但只处理 ACTION_SEND |
| 微信目录扫描 | `MainActivity.java:100-147` | ✅ 有 autoSelectNewestFiles，扫描 Download/Weixin |
| Python Intent 接收 | `wechat_import.py` | ✅ 有，支持 ACTION_SEND + SEND_MULTIPLE |
| Kivy 启动检查 | `main.py:_check_intent` | ⚠️ 只在启动时调用一次 |

## 需要修复的问题（4 项）

### 1. AndroidManifest 添加 launchMode
`MainActivity` 添加 `android:launchMode="singleTask"`，确保前台运行时新 Intent 走 `onNewIntent()` 而非创建新实例。

文件：`d:\voicedoccompare\app\src\main\AndroidManifest.xml`

### 2. Java handleShareIntent 支持 ACTION_SEND_MULTIPLE
当前只处理 `ACTION_SEND`，需增加 `ACTION_SEND_MULTIPLE` 分支，遍历 URI 列表逐个导入。

文件：`d:\voicedoccompare\app\src\main\java\com\voicedoc\app\MainActivity.java`

### 3. Python 侧增加微信目录扫描
在 `wechat_import.py` 中添加 `scan_wechat_dir()` 函数，扫描 Android 常见微信文件目录：
- `/sdcard/Download/Weixin/`
- `/sdcard/Android/data/com.tencent.mm/MicroMsg/Download/`
- `/sdcard/Download/`

按修改时间排序，返回最近的音频和文档文件。

在 `main.py` 首页添加"从微信目录导入"按钮，调用扫描并弹出文件选择列表。

文件：`d:\voicedoccompare\app\core\wechat_import.py`、`d:\voicedoccompare\app\main.py`

### 4. 前台运行时实时响应新 Intent
在 `main.py` 中添加定时轮询机制：每 2 秒检查一次是否有新的 Intent 文件（通过检查缓存目录的新文件），或在 Java 侧通过 Chaquo 直接调用 Python 回调。

采用简单方案：在 `main.py` 的 `build()` 中用 `Clock.schedule_interval` 定期调用 `_check_intent()`，并加去重逻辑（记录已导入文件的路径集合）。

文件：`d:\voicedoccompare\app\main.py`

## 实现步骤

### Step 1: AndroidManifest 加 launchMode
```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:launchMode="singleTask"
    ...>
```

### Step 2: Java handleShareIntent 加 SEND_MULTIPLE
```java
} else if (Intent.ACTION_SEND_MULTIPLE.equals(action)) {
    ArrayList<Uri> uris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
    if (uris != null) {
        for (Uri uri : uris) importFile(uri);
    }
}
```

### Step 3: wechat_import.py 加目录扫描
```python
def scan_wechat_dirs():
    """扫描微信常见文件目录，返回 {'audio': [...], 'doc': [...]}"""
    # 扫描目录列表，按修改时间排序，分类返回
```

### Step 4: main.py 加导入按钮 + 轮询
- KV: 首页"从微信目录导入"按钮
- Python: `_scan_wechat_files()` 弹出文件列表 Popup
- Python: `_check_intent` 加去重 + 定时轮询（3秒间隔）

## 涉及文件

| 文件 | 改动 |
|------|------|
| `AndroidManifest.xml` | 加 launchMode |
| `MainActivity.java` | handleShareIntent 加 SEND_MULTIPLE |
| `wechat_import.py` | 加 scan_wechat_dirs() |
| `main.py` | 加导入按钮 + 文件选择 Popup + 轮询去重 |

## 验证

1. 语法检查所有修改文件
2. 模拟 Intent 数据验证 scan_wechat_dirs 逻辑
3. 确认 AndroidManifest XML 解析通过
