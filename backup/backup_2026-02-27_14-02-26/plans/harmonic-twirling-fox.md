# 转录速度优化：流水线模式

## 背景
当前 `FFmpegSegmentedTranscriptionService.transcribeSegmentedInternal` 采用两阶段串行模式：
1. 先串行提取全部音频段（第256-291行）
2. 全部提取完后，再并行转录（第296-456行）

对于一个30分钟视频（6个5分钟段），假设每段提取10秒、转录30秒：
- 当前：提取60秒 + 转录约45秒（4并行）= **105秒**
- 优化后：提取第1段10秒 + 后续提取与转录重叠 ≈ **60-70秒**，节省约30-40%时间

## 修改方案

### 核心改动：`FFmpegSegmentedTranscriptionService.kt`

将 `transcribeSegmentedInternal` 方法中的两阶段模式改为 **Channel 流水线模式**：

1. **引入 `kotlinx.coroutines.channels.Channel`**
   - 生产者协程：串行提取音频段，每提取完一段就发送到 Channel
   - 消费者协程（多个）：从 Channel 取出段文件，立即开始转录
   - 保持现有 Semaphore(4) 控制并发

2. **保留语言检测逻辑**
   - 第一段仍然单独处理：提取 → 转录 → 检测语言
   - 检测到语言后，启动流水线处理剩余段
   - 这样不影响精确度

3. **具体实现**：
   ```
   // 伪代码
   val channel = Channel<Pair<Int, File>>(capacity = 2)  // 缓冲2段

   // 生产者：串行提取
   launch {
       for (index in remainingIndices) {
           val file = extractSegment(index)
           channel.send(index to file)
       }
       channel.close()
   }

   // 消费者：并行转录（复用现有 semaphore）
   val jobs = (1..PARALLEL_SEGMENTS).map {
       async {
           for ((index, file) in channel) {
               semaphore.withPermit { transcribeSegment(index, file) }
           }
       }
   }
   jobs.awaitAll()
   ```

4. **进度更新调整**
   - 提取和转录进度合并显示
   - 进度计算基于已完成转录的段数

### 需要修改的文件
- `D:/llapk/app/src/main/java/com/multilanglearner/data/network/FFmpegSegmentedTranscriptionService.kt`
  - 新增 import: `kotlinx.coroutines.channels.Channel`
  - 重写 `transcribeSegmentedInternal` 方法（第219-456行）
  - 其他方法（extractAudioSegment、transcribeSegmentWithRetry 等）保持不变

### 不修改的部分
- `WhisperService.kt` — API 调用层不变
- 分段策略（calculateSegments）不变
- 重试机制不变
- 音频压缩参数不变
- 短视频直接转录路径不变

## 验证
1. `./gradlew installDebug` 编译通过
2. 真机测试：转录一个较长视频，观察日志确认提取和转录交替进行
3. 对比转录结果文本，确保精确度不受影响
