# 智能填表助手 - 开发计划

## Context

律师在处理法律文书时需要频繁从各类文档中提取当事人信息并填入表格。本应用旨在自动化这一流程：导入文档 → 实时提取字段 → 滚轮选择 → 一键填入目标位置。

## 技术栈

- **UI框架**: PyQt5 (Qt 5.15.2，已安装)
- **文档解析**: pdfplumber (PDF), python-docx (Word), openpyxl (Excel)
- **OCR**: 百度OCR API (baidu-aip，需先 `pip install chardet`)
- **NLP**: 正则表达式 + jieba 分词（轻量级，快速启动）
- **文本填入**: ctypes SendInput (KEYEVENTF_UNICODE) + 剪贴板 Ctrl+V 备选
- **数据存储**: SQLite (常用字段和分类)
- **打包**: PyInstaller onedir 模式
- **UI风格**: 宋代美学 QSS（宣纸底色、青瓷绿、暖金、楷体/宋体）

## 项目结构

```
C:\Users\Administrator\智能填表助手\
├── main.py                          # 入口
├── app/
│   ├── main_window.py               # 主窗口（状态管理、拖拽、信号连接）
│   ├── core/
│   │   ├── extractor.py             # 文档文本提取（PDF/Word/Excel/剪贴板）
│   │   ├── ocr_client.py            # 百度OCR封装
│   │   ├── nlp_engine.py            # 法律字段提取（正则+jieba）
│   │   └── field_filler.py          # SendInput Unicode 填入
│   ├── data/
│   │   └── db_manager.py            # SQLite 常用字段/分类 CRUD
│   ├── widgets/
│   │   ├── import_bar.py            # 导入按钮栏
│   │   ├── field_selector.py        # 中间选择器（滚轮+填入按钮）
│   │   ├── field_list.py            # 已提取字段列表（编辑/删除/收藏）
│   │   └── favorite_dialog.py       # 常用字段管理对话框
│   └── styles/
│       └── song_dynasty.qss         # 宋代美学样式表
├── build.spec                       # PyInstaller 配置
└── requirements.txt
```

## 开发步骤

### 1. 环境准备
- `pip install chardet jieba`（补充缺失依赖）
- 创建项目目录结构

### 2. 核心模块（无UI依赖，可独立测试）
- `db_manager.py` — SQLite 建表、常用字段/分类 CRUD
- `extractor.py` — PDF/Word/Excel/剪贴板文本提取
- `ocr_client.py` — 百度OCR封装（配置存 %APPDATA%）
- `nlp_engine.py` — 法律字段提取引擎
  - 正则提取：身份证号、手机号、原告/被告+姓名、地址、法定代表人等
  - 标点分界：以标点为边界切分，字段≤30字符
  - 去重过滤
- `field_filler.py` — ctypes SendInput Unicode 逐字输入 + 剪贴板Ctrl+V备选

### 3. UI组件
- `song_dynasty.qss` — 宋代配色（宣纸#F5F0E8、青瓷#5B7F5B、暖金#C4956A、楷体）
- `import_bar.py` — 5个导入按钮（PDF/Word/Excel/剪贴板/图片）
- `field_selector.py` — 大字显示当前字段 + 滚轮切换 + 填入按钮
- `field_list.py` — 可滚动字段列表，每项有编辑/删除/收藏按钮
- `favorite_dialog.py` — 常用字段：显示全部 / 添加新字段 / 分类管理

### 4. 主窗口集成
- `main_window.py` — 组装所有组件、信号槽连接、拖拽支持、WindowStaysOnTopHint
- `main.py` — 入口，加载QSS，启动应用

### 5. 打包
- PyInstaller onedir 模式生成 exe
- 底部版权："著作权属于单夫纯律师所有"

## 关键设计决策

1. **填入方式**: SendInput + KEYEVENTF_UNICODE（支持中文，无需窗口句柄）；点击"填入"后延迟300ms让目标窗口获得焦点
2. **NLP策略**: 正则优先（快），jieba 懒加载（首次提取时才导入，避免拖慢启动）
3. **窗口置顶**: Qt.WindowStaysOnTopHint，律师可边看边填
4. **常用字段**: SQLite 持久化，分类管理，清空按钮在显示常用字段时不生效

## 验证方式

1. 启动应用，确认窗口正常显示宋代风格
2. 拖入一个法律文书PDF，确认字段自动提取并显示
3. 滚轮切换字段，点击"填入"，确认文字出现在记事本光标处
4. 添加/编辑/删除常用字段，重启后确认持久化
5. PyInstaller 打包后运行 exe 验证
