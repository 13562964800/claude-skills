# 计划：添加设置页字号选择 + 传票开庭倒计时红字提示

## 背景
用户需要两个功能：
1. 设置页面让用户自选字号大小（小/中/大）
2. 有传票PDF的短信卡片上，显示红字倒计时提示，格式如"周二-3日后14:30"

## 修改文件清单

### 1. 新建 `SettingsScreen.kt` — 设置页面
- 路径：`app/src/main/java/com/court/assistant/ui/SettingsScreen.kt`
- 提供字号选择：小/中/大 三档
- 使用 SharedPreferences 持久化（项目无 DataStore 依赖，不额外引入）

### 2. 修改 `Type.kt` — 动态字号
- 将 `CourtTypography` 改为函数 `CourtTypography(scale: Float)`，根据用户选择的缩放比例生成 Typography
- 小=0.85f，中=1.0f（默认），大=1.15f

### 3. 修改 `Theme.kt` — 读取用户字号偏好
- `CourtAssistantTheme` 中读取 SharedPreferences 获取字号缩放，传给 Typography

### 4. 修改 `MainScreen.kt` — 开庭倒计时红字
- 在 `SmsMenuCard` 头部，当 `sms.hearingTime != null` 时，显示红字倒计时
- 格式："周二-3日后14:30"
- 添加设置按钮入口（TopAppBar actions 中加齿轮图标）

### 5. 修改 `Navigation.kt` — 添加设置路由
- 添加 `"settings"` 路由指向 `SettingsScreen`

## 倒计时格式逻辑
```
hearingTime → Calendar:
- 星期几 → 周一~周日
- 距今天数 → 今天/明天/X日后/已过期
- 时间 → HH:mm
- 组合："周二-3日后14:30"
```

## 验证
- 打包安装到真机
- 测试设置页切换字号，返回主页字号变化
- 确认有传票的卡片显示红字倒计时
