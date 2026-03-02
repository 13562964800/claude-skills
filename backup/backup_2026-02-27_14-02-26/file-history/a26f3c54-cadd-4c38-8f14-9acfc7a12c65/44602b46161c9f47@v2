# AIHotspot 功能增强计划

## 背景
用户在真机测试后提出4个问题：缺少一键更新按钮、无法自定义RSS源、大模型配置未接入、抓取内容为英文。

## 修改清单

### 1. 主界面添加醒目的一键更新 FAB 按钮
- 文件：`HomeScreen.kt`
- 在 Scaffold 中添加 `floatingActionButton`，使用 `ExtendedFloatingActionButton`，显示"一键抓取"
- 刷新时显示加载动画，完成后显示 Snackbar 提示结果

### 2. 用户自定义 RSS 源管理
- 新建 `data/local/entity/CustomRssEntity.kt` — Room 实体存储用户添加的 RSS 源
- 新建 `data/local/dao/CustomRssDao.kt` — CRUD 操作
- 修改 `AppDatabase.kt` — 添加 CustomRssDao，version 升到 2
- 修改 `HotspotRepositoryImpl.kt` — refreshHotspots 时同时抓取用户自定义 RSS 源
- 修改 `IHotspotRepository.kt` — 添加自定义 RSS 源管理接口
- 扩展 `SettingsScreen.kt` — 添加"订阅源管理"区块，支持添加/删除自定义 RSS URL

### 3. 大模型配置（设置页 + 关键词抓取）
- 精简 `LLMProvider.kt` — 国内3家（DeepSeek、通义千问、智谱AI）+ 国外3家（OpenAI、Anthropic、Google），更新为最新顶尖模型
- 新建 `data/local/UserPrefsManager.kt` — DataStore 存储 LLM API Key、选中模型、用户关键词
- 扩展 `SettingsScreen.kt` — 添加"大模型配置"区块（选择提供商、输入 API Key、选择模型）和"关键词设置"区块
- 修改 `HotspotRepositoryImpl.kt` — 添加基于关键词的抓取逻辑（用关键词过滤 RSS 内容 + GitHub 搜索）
- 修改 `HomeViewModel.kt` — 支持关键词触发抓取

### 4. 内容中文化
- 问题本质：GitHub Trending 和 Hacker News 的内容本身是英文
- 方案：在 `HotspotCard.kt` 和 `DetailScreen.kt` 中，contentDescription 等残留英文改为中文
- 在 `GitHubApi.kt` 的 `aiKeywords` 中增加更多中文关键词
- 添加更多中文 RSS 源到默认列表（如 36kr AI、机器之心、量子位）
- 未来可用 LLM 翻译标题，但当前先通过增加中文源解决

## 关键文件修改列表
1. `app/build.gradle.kts` — 无需改动
2. `domain/model/LLMProvider.kt` — 精简为6家，更新模型名
3. `data/local/entity/CustomRssEntity.kt` — 新建
4. `data/local/dao/CustomRssDao.kt` — 新建
5. `data/local/AppDatabase.kt` — 添加新 DAO，升级版本
6. `data/local/UserPrefsManager.kt` — 新建，DataStore 管理用户偏好
7. `data/remote/rss/RssSource.kt` — 添加中文 RSS 源
8. `data/repository/HotspotRepositoryImpl.kt` — 支持自定义源 + 关键词过滤
9. `domain/repository/IHotspotRepository.kt` — 添加自定义源接口
10. `di/AppModule.kt` — 提供 UserPrefsManager
11. `presentation/settings/SettingsScreen.kt` — 重写，三个区块：主题、订阅源、大模型配置
12. `presentation/home/HomeScreen.kt` — 添加 FAB 按钮
13. `presentation/home/HomeViewModel.kt` — 注入 UserPrefsManager
14. `presentation/home/HomeState.kt` — 添加关键词状态
15. `presentation/components/HotspotCard.kt` — 英文 contentDescription 改中文
16. `presentation/detail/DetailScreen.kt` — 同上
17. `data/remote/api/GitHubApi.kt` — 增加中文关键词

## 验证方式
1. `./gradlew.bat assembleDebug --no-daemon` 构建通过
2. `adb install` 安装到真机
3. 验证：主界面有 FAB 按钮、设置页有三个配置区块、添加自定义 RSS 源后能抓取、中文内容优先显示
