# AI热点信息抓取Android应用 - 实现计划

## 📋 项目概述

### 需求背景
用户希望开发一个Android APK应用，能够**实时抓取AI科技热点信息并自动更新**。参考数据源包括：
- WaytoAGI (www.waytoagi.com)
- GitHub Trending
- 各大科技媒体RSS源

### 核心功能
1. **多源聚合** - 整合GitHub、WaytoAGI、RSS等多个AI信息源
2. **实时更新** - 后台定时抓取，支持推送通知
3. **智能分类** - 按项目、工具、模型、资讯等分类
4. **本地缓存** - 离线阅读支持
5. **搜索过滤** - 关键词搜索和标签过滤

---

## 🏗️ 技术架构

### 技术栈选择（原生Android方案）

| 层级 | 技术选型 | 说明 |
|------|----------|------|
| **UI层** | Jetpack Compose + Material 3 | 现代声明式UI，代码量减少40% |
| **网络层** | Ktor Client + OkHttp | 协程友好，支持RSS/XML解析 |
| **数据层** | Room Database + DataStore | 本地缓存，偏好设置 |
| **依赖注入** | Hilt | Google官方推荐 |
| **后台任务** | WorkManager | 定时抓取，电池优化 |
| **图片加载** | Coil | Compose友好 |

### 架构模式：MVVM + Clean Architecture

```
┌─────────────────────────────────────────┐
│          UI Layer (Compose)             │
│  ├── Screens (Home, Detail, Settings)   │
│  └── Components (Cards, Lists)          │
├─────────────────────────────────────────┤
│         Presentation Layer              │
│  ├── ViewModels                         │
│  └── UI State (StateFlow)               │
├─────────────────────────────────────────┤
│          Domain Layer                   │
│  ├── UseCases                           │
│  ├── Repository Interfaces              │
│  └── Domain Models                      │
├─────────────────────────────────────────┤
│           Data Layer                    │
│  ├── Repository Implementations         │
│  ├── Remote (API/RSS Clients)           │
│  └── Local (Room Database)              │
└─────────────────────────────────────────┘
```

---

## 📁 项目结构

```
app/
├── src/main/java/com/ai/hotspot/
│   ├── App.kt                          # Application类
│   ├── di/                             # 依赖注入模块
│   │   ├── AppModule.kt
│   │   ├── NetworkModule.kt
│   │   └── DatabaseModule.kt
│   ├── data/                           # 数据层
│   │   ├── remote/
│   │   │   ├── api/
│   │   │   │   ├── GitHubApi.kt
│   │   │   │   └── WaytoAGIApi.kt
│   │   │   ├── rss/
│   │   │   │   ├── RssParser.kt
│   │   │   │   └── RssSource.kt
│   │   │   └── dto/
│   │   ├── local/
│   │   │   ├── dao/
│   │   │   │   └── HotspotDao.kt
│   │   │   ├── entity/
│   │   │   │   └── HotspotEntity.kt
│   │   │   └── AppDatabase.kt
│   │   ├── mapper/
│   │   │   └── HotspotMapper.kt
│   │   └── repository/
│   │       └── HotspotRepositoryImpl.kt
│   ├── domain/                         # 领域层
│   │   ├── model/
│   │   │   └── Hotspot.kt
│   │   ├── repository/
│   │   │   └── IHotspotRepository.kt
│   │   └── usecase/
│   │       ├── GetHotspotsUseCase.kt
│   │       ├── RefreshHotspotsUseCase.kt
│   │       └── SearchHotspotsUseCase.kt
│   ├── presentation/                   # 表现层
│   │   ├── home/
│   │   │   ├── HomeScreen.kt
│   │   │   ├── HomeViewModel.kt
│   │   │   └── HomeState.kt
│   │   ├── detail/
│   │   │   ├── DetailScreen.kt
│   │   │   └── DetailViewModel.kt
│   │   ├── settings/
│   │   │   ├── SettingsScreen.kt
│   │   │   └── SettingsViewModel.kt
│   │   ├── components/
│   │   │   ├── HotspotCard.kt
│   │   │   ├── SourceChip.kt
│   │   │   └── LoadingIndicator.kt
│   │   ├── theme/
│   │   │   ├── Color.kt
│   │   │   ├── Theme.kt
│   │   │   └── Type.kt
│   │   └── navigation/
│   │       └── NavGraph.kt
│   └── worker/
│       └── RefreshWorker.kt            # 后台刷新任务
├── src/main/res/
│   └── values/
│       ├── strings.xml
│       └── themes.xml
└── build.gradle.kts
```

---

## 🔌 数据源配置

### 1. GitHub Trending API
```kotlin
// 使用非官方API或RSS
// RSS: https://mshibanami.github.io/GitHubTrendingRSS/daily/all.xml
// API: https://api.gitterapp.com/repositories?language=&since=daily
```

### 2. WaytoAGI
```kotlin
// 网页抓取或RSS
// 需要解析HTML结构获取最新内容
```

### 3. RSS源列表
```kotlin
enum class RssSource(val url: String, val category: String) {
    GITHUB_TRENDING("https://mshibanami.github.io/GitHubTrendingRSS/daily/all.xml", "GitHub"),
    HACKER_NEWS("https://hnrss.org/frontpage", "资讯"),
    AI_WEEKLY("https://aiweekly.co/rss", "AI周刊"),
    MACHINE_LEARNING("https://feeds.feedburner.com/oreilly/radar", "技术雷达")
}
```

---

## 📊 数据模型

### Domain Model
```kotlin
data class Hotspot(
    val id: String,
    val title: String,
    val description: String,
    val source: Source,           // 数据来源
    val category: Category,       // 分类
    val url: String,              // 原文链接
    val stars: Int? = null,       // GitHub stars
    val author: String? = null,   // 作者
    val tags: List<String>,       // 标签
    val publishedAt: LocalDateTime,
    val fetchedAt: LocalDateTime, // 抓取时间
    val isRead: Boolean = false,
    val isBookmarked: Boolean = false
)

enum class Source { GITHUB, WAYTOAGI, RSS, OTHER }
enum class Category { PROJECT, TOOL, MODEL, NEWS, TUTORIAL }
```

### Room Entity
```kotlin
@Entity(tableName = "hotspots")
data class HotspotEntity(
    @PrimaryKey val id: String,
    val title: String,
    val description: String,
    val source: String,
    val category: String,
    val url: String,
    val stars: Int?,
    val author: String?,
    val tags: String,        // JSON序列化
    val publishedAt: Long,
    val fetchedAt: Long,
    val isRead: Boolean,
    val isBookmarked: Boolean
)
```

---

## 🚀 实现步骤

### 阶段1：项目初始化（1-2小时）
1. [ ] 创建Android项目，配置Kotlin + Compose
2. [ ] 添加依赖：Hilt, Room, Ktor, Coil, WorkManager
3. [ ] 配置主题和基础UI组件

### 阶段2：数据层实现（3-4小时）
1. [ ] 创建Room数据库和DAO
2. [ ] 实现GitHub API客户端
3. [ ] 实现RSS解析器
4. [ ] 实现Repository层

### 阶段3：UI层实现（3-4小时）
1. [ ] HomeScreen - 热点列表
2. [ ] DetailScreen - 详情页
3. [ ] SettingsScreen - 设置页
4. [ ] 导航配置

### 阶段4：后台任务（1-2小时）
1. [ ] WorkManager定时任务
2. [ ] 通知推送
3. [ ] 增量更新逻辑

### 阶段5：优化与测试（2小时）
1. [ ] UI优化和动画
2. [ ] 错误处理
3. [ ] 性能优化

---

## 🔧 核心代码示例

### build.gradle.kts 依赖
```kotlin
dependencies {
    // Compose
    implementation("androidx.compose.ui:ui:1.7.0")
    implementation("androidx.compose.material3:material3:1.3.0")
    implementation("androidx.activity:activity-compose:1.9.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")

    // Hilt
    implementation("com.google.dagger:hilt-android:2.52")
    kapt("com.google.dagger:hilt-compiler:2.52")

    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")

    // Ktor
    implementation("io.ktor:ktor-client-android:3.0.0")
    implementation("io.ktor:ktor-client-content-negotiation:3.0.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.0.0")

    // Coil
    implementation("io.coil-kt:coil-compose:2.7.0")

    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.9.0")

    // XML Parsing
    implementation("com.rometools:rome:2.1.0")
}
```

### HomeViewModel
```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getHotspotsUseCase: GetHotspotsUseCase,
    private val refreshHotspotsUseCase: RefreshHotspotsUseCase
) : ViewModel() {

    private val _state = MutableStateFlow(HomeState())
    val state: StateFlow<HomeState> = _state.asStateFlow()

    init {
        loadHotspots()
    }

    fun loadHotspots() {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            getHotspotsUseCase()
                .catch { e -> _state.update { it.copy(error = e.message, isLoading = false) } }
                .collect { hotspots ->
                    _state.update { it.copy(hotspots = hotspots, isLoading = false) }
                }
        }
    }

    fun refresh() {
        viewModelScope.launch {
            refreshHotspotsUseCase()
        }
    }
}
```

### HomeScreen
```kotlin
@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel(),
    onItemClick: (Hotspot) -> Unit
) {
    val state by viewModel.state.collectAsState()
    val refreshState = rememberPullToRefreshState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("AI 热点") },
                actions = {
                    IconButton(onClick = { /* 搜索 */ }) {
                        Icon(Icons.Default.Search, "搜索")
                    }
                }
            )
        }
    ) { padding ->
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .padding(padding),
            contentPadding = PaddingValues(16.dp),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            items(state.hotspots, key = { it.id }) { hotspot ->
                HotspotCard(
                    hotspot = hotspot,
                    onClick = { onItemClick(hotspot) }
                )
            }
        }

        if (state.isLoading) {
            CircularProgressIndicator(
                modifier = Modifier.align(Alignment.Center)
            )
        }
    }
}
```

---

## ✅ 验证方案

### 功能测试
1. **启动测试** - 应用能正常启动并显示数据
2. **刷新测试** - 下拉刷新能获取最新数据
3. **缓存测试** - 离线模式能显示缓存数据
4. **通知测试** - 后台更新能触发通知

### 性能指标
- 启动时间 < 2秒
- 列表滚动流畅度 60fps
- 内存占用 < 100MB
- 后台刷新不显著影响电池

---

## 📚 参考资源

### 开源项目参考
1. **ReadYou** - https://github.com/Ashinch/ReadYou
   - Material You设计，Compose架构典范
2. **Twine** - https://github.com/msasikanth/reader
   - Kotlin Multiplatform RSS阅读器
3. **FeedFlow** - https://github.com/prof18/feed-flow
   - 现代化RSS阅读器

### API资源
- GitHub Trending RSS: https://mshibanami.github.io/GitHubTrendingRSS/
- Hacker News RSS: https://hnrss.org/
- AI Weekly: https://aiweekly.co/rss

---

## 🎯 预计产出

完成后的APK将具备：
- ✅ 多源AI热点聚合
- ✅ Material 3 现代化UI
- ✅ 后台自动更新
- ✅ 离线阅读支持
- ✅ 搜索和分类功能
- ✅ 收藏和分享功能
