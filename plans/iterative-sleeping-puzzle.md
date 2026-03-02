# 配音文字核对宝 v2.7 修复与新功能计划

## Context

当前 v2.6 版本存在4个bug：LLM审核因 SharedPreferences key 不匹配而从未生效、百度ASR分段异常被吞掉、pypinyin 未正确打包导致同音字全部误标红、时间戳映射可能因异常丢失。同时用户需要新功能：任意录音可直接转写（无需文档）、转写结果可预览和分享。

---

## Bug 修复

### Fix 1: LLM 审核从未生效（SharedPreferences key 不匹配）

**文件**: `app/src/main/java/com/voicedoc/app/MainActivity.java:679-680`

SettingsActivity 保存时用的 key：
- `llm_provider_id` → 存 provider ID（如 "zhipu"）
- `llm_key_zhipu` → 存对应 API key

但 MainActivity 读取时用的 key：
- `llm_provider` → 永远读到空字符串
- `llm_key` → 永远读到空字符串

**修改**：将第679-680行改为：
```java
String llmProvider = prefs.getString("llm_provider_id", "");
String llmKey = llmProvider.isEmpty() ? "" : prefs.getString("llm_key_" + llmProvider, "");
```

### Fix 2: 百度ASR分段异常被吞掉

**文件**: `app/src/main/python/engine.py`

- 第314行 `except:` → 改为 `except Exception as e:` + print 日志
- 第340行 `except: pass` → 改为 `except Exception as e: print(...)` 记录失败原因
- 第353行 `_split_wav_simple()` 中类似的 except 也加日志

### Fix 3: pypinyin 未打包到 APK

**文件**: `app/build.gradle:17-25`

当前 pip 块已有 `install "pypinyin"`，但需要同时添加 `jieba`（虽然当前 `_segment()` 是逐字切分，但 README 提到 jieba 分词）。确认 pypinyin 版本兼容 Chaquopy：
```groovy
pip {
    install "requests"
    install "python-docx"
    install "pypinyin"
    install "jieba"
}
```

同时在 `engine.py:59-66` 的 `_get_pinyin()` 中确保 ImportError 时返回 None（已有），并在 `_same_pinyin()` 中确保 None 时返回 False 而非抛异常。

### Fix 4: 时间戳映射异常保护

**文件**: `app/src/main/python/engine.py:883-1001` `run_compare_with_timestamps()`

在 `_same_pinyin()` 调用处（第956行）确保不会因 pypinyin 异常导致整个对比崩溃。当前代码已有 `if pa is None or pb is None: return False` 逻辑，但需确认 `_get_pinyin()` 的 try/except 覆盖了所有异常类型（改为 `except Exception`）。

---

## 新功能：仅转写模式 + 预览分享

### 改动1: activity_main.xml 添加按钮

在"开始对比"按钮下方添加"仅转写"按钮，在报告按钮行添加"预览/分享转写"按钮：

```xml
<!-- 仅转写按钮，紧跟在 btnCompare 之后 -->
<Button
    android:id="@+id/btnTranscribeOnly"
    android:layout_width="match_parent"
    android:layout_height="48dp"
    android:text="仅转写（无需文档）"
    android:textSize="15sp"
    android:backgroundTint="#FF9800"
    android:layout_marginTop="8dp" />
```

在报告按钮行添加分享转写结果按钮：
```xml
<Button
    android:id="@+id/btnShareTranscript"
    android:layout_width="0dp"
    android:layout_height="48dp"
    android:layout_weight="1"
    android:text="分享转写"
    android:textSize="14sp"
    android:backgroundTint="#FF9800"
    android:visibility="gone"
    android:layout_marginStart="4dp" />
```

### 改动2: MainActivity.java 添加仅转写逻辑

新增成员变量：
```java
private String lastTranscriptText = "";  // 保存最近一次转写文本
```

在 `onCreate()` 中绑定按钮：
```java
findViewById(R.id.btnTranscribeOnly).setOnClickListener(v -> startTranscribeOnly());
btnShareTranscript = findViewById(R.id.btnShareTranscript);
if (btnShareTranscript != null) btnShareTranscript.setOnClickListener(v -> shareTranscript());
```

新增 `startTranscribeOnly()` 方法：
- 只需要 audioPath 非空
- 复用现有转录逻辑（Sherpa/Python engine）
- 转录完成后将文本显示在 tvResult 中
- 显示 btnShareTranscript 按钮

新增 `shareTranscript()` 方法：
- 将 lastTranscriptText 保存为 txt 文件
- 通过 FileProvider + Intent.ACTION_SEND 分享
- 同时支持纯文本分享（Intent.EXTRA_TEXT）

### 改动3: engine.py 无需新增函数

仅转写模式复用现有的 `transcribe_with_ts()` 和 `create_approx_timestamps()`，Java 端直接调用即可。

---

## 修改文件清单

| 文件 | 改动 |
|------|------|
| `app/src/main/java/.../MainActivity.java` | Fix1: 修复 LLM prefs key (2行)；新功能: 添加 startTranscribeOnly() + shareTranscript() |
| `app/src/main/python/engine.py` | Fix2: 百度分段加日志 (3处)；Fix4: _get_pinyin except 加固 |
| `app/build.gradle` | Fix3: 确认 pypinyin + 添加 jieba |
| `app/src/main/res/layout/activity_main.xml` | 新功能: 添加仅转写按钮 + 分享转写按钮 |

---

## 验证方案

1. **LLM 审核**：在设置页配置任意 LLM provider + key → 执行对比 → 确认状态栏显示"大模型审核差异中..."
2. **百度分段**：选择百度引擎 → 用超过60秒的音频测试 → 查看 logcat 中是否有 `[BaiduASR]` 日志
3. **pypinyin**：构建 APK → 用包含同音字的文本测试（如"上帝"vs"尚地"）→ 确认不标红
4. **仅转写**：只选择录音不选文档 → 点击"仅转写" → 确认显示转写文本
5. **分享转写**：转写完成后 → 点击"分享转写" → 确认可分享到微信
6. **构建验证**：`cd /d/VoiceDocCompare && ./gradlew assembleDebug` 确认编译通过
