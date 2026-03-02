# 多大模型并行调用功能实现计划

## 背景

CourtAssistant 项目需要接入大模型提高复杂格式（传票 PDF、短信）的识别率。采用**并行调用对比**策略，同时调用多家大模型，对比结果取最优。

## 目标

1. 接入国内 3 家 + 国外 4 家大模型
2. 并行调用所有模型，对比结果选择最优
3. 用户自行配置各模型的 API Key

## 大模型配置清单

### 国内三巨头

| 提供商 | Base URL | API Key 格式 | 备注 |
|--------|----------|--------------|------|
| **百度文心一言** | `https://aip.baidubce.com` | `API Key` + `Secret Key` | 需要两步认证获取 Access Token |
| **阿里通义千问** | `https://dashscope.aliyuncs.com` | `sk-xxxxxxxx` | DashScope API |
| **智谱 GLM** | `https://open.bigmodel.cn` | `id.secret` | JWT Token 认证 |

### 国外四巨头

| 提供商 | Base URL | API Key 格式 | 备注 |
|--------|----------|--------------|------|
| **OpenAI GPT-4** | `https://api.openai.com` | `sk-xxxxxxxx` | Chat Completions API |
| **Anthropic Claude** | `https://api.anthropic.com` | `sk-ant-xxxxxxxx` | Messages API |
| **Google Gemini** | `https://generativelanguage.googleapis.com` | `AIzaxxxxxxxx` | REST API |
| **Mistral AI** | `https://api.mistral.ai` | `xxxxxxxxxxxxx` | Chat Completions |

## 架构设计

### 1. 目录结构

```
app/src/main/java/com/shanfuchun/courtassistant/
├── ai/
│   ├── model/                      # 数据模型
│   │   ├── LLMProvider.kt          # 提供商枚举
│   │   ├── LLMRequest.kt           # 请求模型
│   │   ├── LLMResponse.kt          # 响应模型
│   │   ├── LLMConfig.kt            # 配置模型
│   │   └── SummonsExtraction.kt    # 传票抽取结果
│   ├── provider/                   # 各大模型客户端
│   │   ├── BaseLLMProvider.kt      # 基类接口
│   │   ├── WenxinProvider.kt       # 百度文心
│   │   ├── QwenProvider.kt         # 阿里通义
│   │   ├── ZhipuProvider.kt        # 智谱 GLM
│   │   ├── OpenAIProvider.kt       # OpenAI
│   │   ├── ClaudeProvider.kt       # Anthropic
│   │   ├── GeminiProvider.kt       # Google
│   │   └── MistralProvider.kt      # Mistral
│   ├── orchestrator/               # 并行调用编排
│   │   ├── LLMOrchestrator.kt     # 主编排器
│   │   └── ResultComparator.kt    # 结果对比器
│   ├── repository/                 # 数据层
│   │   └── LLMConfigRepository.kt  # 配置管理
│   └── prompt/                     # Prompt 模板
│       └── SummonsPrompt.kt       # 传票抽取 Prompt
├── ui/
│   └── settings/
│       └── AISettingsFragment.kt   # AI 设置页面
└── data/
    └── local/
        └── dao/
            └── LLMConfigDao.kt     # 配置存储
```

### 2. 核心接口设计

```kotlin
// 统一的 LLM 提供商接口
interface LLMProvider {
    val name: String                  // 提供商名称
    val baseUrl: String               // API Base URL
    val enabled: Boolean              // 是否启用
    val apiKey: String                // API Key

    suspend fun extractSummons(
        pdfText: String,
        ocrText: String?
    ): Result<SummonsExtraction>

    suspend fun extractFromSms(
        smsBody: String
    ): Result<SummonsExtraction>
}

// 响应结果模型
data class LLMResult(
    val provider: String,
    val response: SummonsExtraction,
    val latency: Long,
    val success: Boolean,
    val error: Throwable? = null,
    val confidence: Float            // AI 自信度
)

// 传票抽取结果
data class SummonsExtraction(
    val caseNo: String? = null,          // 案号
    val courtName: String? = null,       // 法院名称
    val hearingTime: Long? = null,       // 开庭时间戳
    val location: String? = null,        // 地点
    val partyName: String? = null,        // 当事人
    val confidence: Float = 0f,          // 自信度 0-1
    val rawResponse: String = ""          // 原始响应
)
```

### 3. 并行调用编排器

```kotlin
class LLMOrchestrator(
    private val providers: List<LLMProvider>,
    private val comparator: ResultComparator
) {
    suspend fun extractSummonsParallel(
        pdfText: String,
        ocrText: String? = null
    ): LLMResult? {
        // 并行调用所有启用的提供商
        val results = providers
            .filter { it.enabled && it.apiKey.isNotBlank() }
            .map { provider ->
                async(Dispatchers.IO) {
                    val startTime = System.currentTimeMillis()
                    try {
                        withTimeout(30.seconds) {
                            val extraction = provider.extractSummons(pdfText, ocrText)
                            val latency = System.currentTimeMillis() - startTime
                            LLMResult(
                                provider = provider.name,
                                response = extraction.getOrNull() ?: SummonsExtraction(),
                                latency = latency,
                                success = extraction.isSuccess,
                                confidence = extraction.getOrNull()?.confidence ?: 0f
                            )
                        }
                    } catch (e: Exception) {
                        LLMResult(
                            provider = provider.name,
                            response = SummonsExtraction(),
                            latency = System.currentTimeMillis() - startTime,
                            success = false,
                            error = e
                        )
                    }
                }
            }
            .awaitAll()

        // 使用对比器选择最佳结果
        return comparator.selectBest(results)
    }
}
```

### 4. 结果对比策略

```kotlin
class ResultComparator {
    fun selectBest(results: List<LLMResult>): LLMResult? {
        val successful = results.filter { it.success }
        if (successful.isEmpty()) return null

        // 选择策略：
        // 1. 优先选择自信度最高的
        // 2. 自信度相同时，选择响应最快的
        // 3. 多个结果相似时，选择出现次数最多的（共识）

        return successful
            .sortedByDescending { it.confidence }
            .thenBy { it.latency }
            .firstOrNull()
    }
}
```

### 5. 配置管理

```kotlin
// 使用 DataStore 存储配置
class LLMConfigRepository(private val context: Context) {
    private val Context.dataStore by preferencesDataStore("llm_config")

    val providerConfigs: Map<String, Flow<ProviderConfig>> = mapOf(
        "wenxin" to getConfigFlow("wenxin"),
        "qwen" to getConfigFlow("qwen"),
        "zhipu" to getConfigFlow("zhipu"),
        "openai" to getConfigFlow("openai"),
        "claude" to getConfigFlow("claude"),
        "gemini" to getConfigFlow("gemini"),
        "mistral" to getConfigFlow("mistral")
    )

    suspend fun saveConfig(provider: String, config: ProviderConfig) {
        context.dataStore.edit { prefs ->
            prefs[stringPreferencesKey("${provider}_enabled")] = config.enabled
            prefs[stringPreferencesKey("${provider}_apiKey")] = config.apiKey
            // 文心一言需要额外的 secretKey
            config.secretKey?.let {
                prefs[stringPreferencesKey("${provider}_secretKey")] = it
            }
        }
    }
}

data class ProviderConfig(
    val enabled: Boolean = false,
    val apiKey: String = "",
    val secretKey: String? = null  // 仅文心一言需要
)
```

### 6. Prompt 模板

```kotlin
object SummonsPrompt {
    private const val SYSTEM_PROMPT = """
你是一位专业的法律文书分析助手，专门负责从法院传票和相关文档中提取关键信息。

请从提供的文本中提取以下信息（如果不存在则返回 null）：
1. 案号 (caseNo) - 格式如：(2024)鲁1312民初4967号
2. 法院名称 (courtName) - 如：北京市朝阳区人民法院
3. 开庭时间 (hearingTime) - 转换为 Unix 时间戳（毫秒）
4. 地点 (location) - 开庭地点
5. 当事人 (partyName) - 原告或被告名称

请以 JSON 格式返回结果。
"""

    fun buildPrompt(pdfText: String, ocrText: String? = null): String {
        val content = buildString {
            append("PDF 文本内容：\n$pdfText\n\n")
            ocrText?.let { append("OCR 识别内容：\n$it\n\n") }
            append("请根据上述内容提取传票信息。")
        }
        return content
    }
}
```

## 实现步骤

### Phase 1: 基础架构搭建
1. 创建 `ai/` 目录结构
2. 定义数据模型 (`LLMProvider`, `LLMResult`, `SummonsExtraction`)
3. 创建配置 Repository 和 DataStore

### Phase 2: 各大模型客户端实现
4. 实现 `BaseLLMProvider` 基类接口
5. 实现国内 3 家提供商客户端
6. 实现国外 4 家提供商客户端

### Phase 3: 并行调用编排
7. 实现 `LLMOrchestrator` 并行调用器
8. 实现 `ResultComparator` 结果对比器
9. 实现 `SummonsPrompt` Prompt 模板

### Phase 4: UI 配置界面
10. 创建 `AISettingsFragment` 设置页面
11. 为每个提供商创建配置项（开关 + API Key 输入）
12. 实现配置保存和读取

### Phase 5: 集成到现有流程
13. 修改 `PdfParser.kt` 集成 AI 抽取
14. 修改 `TextParser.kt` 增加 AI 回退
15. 添加 UI 开关控制 AI 功能

### Phase 6: 测试验证
16. 测试单个提供商调用
17. 测试并行调用和结果对比
18. 测试各种异常场景

## 需要修改的关键文件

| 文件 | 操作 | 说明 |
|------|------|------|
| `app/build.gradle.kts` | 修改 | 添加 OkHttp、Coroutines、DataStore 依赖 |
| `PdfParser.kt` | 修改 | 集成 AI 抽取 |
| `TextParser.kt` | 修改 | 增加 AI 回退 |
| `SettingsFragment.kt` | 修改 | 添加 AI 配置入口 |
| `RepositoryManager.kt` | 修改 | 添加 LLMConfigRepository |

## 验证方案

1. **单个提供商测试**：配置一个 API Key，验证能否正确抽取传票信息
2. **并行调用测试**：配置多个 API Key，验证并行调用是否正常
3. **结果对比测试**：模拟不同提供商返回不同结果，验证选择逻辑
4. **异常测试**：无 API Key、API Key 错误、网络超时等场景
5. **性能测试**：并行调用响应时间是否符合预期

## 依赖添加

```kotlin
// build.gradle.kts
dependencies {
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.0")

    // OkHttp
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")

    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.0.0")

    // JSON 解析
    implementation("com.google.code.gson:gson:2.10.1")

    // 可选：使用官方 SDK
    implementation("com.alibaba:dashscope-sdk-java:2.14.2")  // 阿里通义
}
```
