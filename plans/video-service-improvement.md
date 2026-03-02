# VideoDownloadService 改进计划

## 研究总结

基于GitHub top项目研究，以下是关键发现：

### 1. yt-dlp - 最佳参考项目
- **仓库**: https://github.com/yt-dlp/yt-dlp
- **支持**: 1000+ 网站
- **架构**: 每个平台独立extractor，统一InfoExtractor基类
- **关键**: 多层备用API策略

### 2. Evil0ctal/TikTok_Download_API - 多平台解析
- **仓库**: https://github.com/Evil0ctal/Douyin_TikTok_Download_API
- **支持**: 抖音、TikTok、B站、快手、小红书
- **特点**: 异步高性能、无水印下载
- **技术**: Python + FastAPI + AIOHTTP

### 3. B站API策略
- **主API**: `api.bilibili.com/x/player/playurl`
- **备用1**: `app.bilibili.com` (App API)
- **备用2**: `grpc.biliapi.net` (gRPC)
- **备用3**: `interface.bilibili.com` (Legacy)
- **认证**: SESSDATA + bili_jct + WBI签名

### 4. 长视频分段策略
- **当前实现**: 10分钟分段 + 5秒重叠 ✅
- **可改进**: 添加静音检测优化分段点

## 改进方案

### Phase 1: 多层备用API策略 (高优先级)

#### B站改进
```kotlin
private val bilibiliApis = listOf(
    "https://api.bilibili.com/x/player/playurl",
    "https://app.bilibili.com/bilibili.pgc.gateway.player.v2",
    "https://grpc.biliapi.net/bilibili.app.playurl.v1"
)
```

#### 抖音/TikTok改进
- 参考 Evil0ctal 项目的API调用方式
- 添加更多备用解析API
- 改进X-Bogus签名处理

#### YouTube改进
- 使用InnerTube API
- 添加signature cipher处理
- 参考NewPipe的实现

### Phase 2: 错误处理和重试机制

```kotlin
data class ApiEndpoint(
    val url: String,
    val priority: Int,
    val requiresAuth: Boolean = false,
    val headers: Map<String, String> = emptyMap()
)

suspend fun fetchWithFallback(
    endpoints: List<ApiEndpoint>,
    onSuccess: (Response) -> Unit,
    onFailure: (Exception) -> Unit
) {
    for (endpoint in endpoints.sortedBy { it.priority }) {
        try {
            val response = httpClient.get(endpoint.url) {
                headers { endpoint.headers.forEach { (k, v) -> append(k, v) } }
            }
            if (response.status.value == 200) {
                onSuccess(response)
                return
            }
        } catch (e: Exception) {
            Log.w(TAG, "Endpoint ${endpoint.url} failed: ${e.message}")
        }
    }
    onFailure(Exception("All endpoints failed"))
}
```

### Phase 3: 响应格式兼容

不同API可能返回不同格式：
```kotlin
when {
    response.contains("\"data\"") -> parseAsJson()
    response.contains("<?xml") -> parseAsXml()
    response.startsWith("var __playinfo__") -> parseAsJavascript()
}
```

## 实施清单

| 优先级 | 任务 | 文件 |
|--------|------|------|
| 🔴 高 | 添加B站备用API列表 | VideoDownloadService.kt |
| 🔴 高 | 实现通用fallback机制 | VideoDownloadService.kt |
| 🟡 中 | 添加抖音更多备用API | VideoDownloadService.kt |
| 🟡 中 | 改进错误提示信息 | PlayerViewModel.kt |
| 🟢 低 | 添加静音检测分段 | FFmpegSegmentedTranscriptionService.kt |
| 🟢 低 | YouTube InnerTube支持 | VideoDownloadService.kt |

## 关键代码位置

- **B站解析**: `VideoDownloadService.kt:272-450`
- **抖音解析**: `VideoDownloadService.kt:452-600`
- **YouTube解析**: `VideoDownloadService.kt:123-246`
- **错误处理**: `WhisperService.kt:100-133`

## 预期效果

1. ✅ 更稳定的视频解析（多层备用API）
2. ✅ 更清晰的错误提示（引导用户使用导入功能）
3. ✅ 更好的重试机制（自动切换备用API）
4. ✅ 长视频转录更可靠（当前已实现）
