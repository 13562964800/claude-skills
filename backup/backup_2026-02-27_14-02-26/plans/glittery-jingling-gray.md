# SmartFormFiller 功能增强计划

## 背景
当前应用只能通过正则提取固定类型的字段（身份证、电话、角色名等），无法处理表格中的键值对，也不支持用户自定义提取字段。需要增加灵活性和 LLM 辅助能力。

## 改动概览

### 1. 改进字段提取（nlp_engine.py + extractor.py）
- 新增键值对提取：匹配 "姓名：张三"、"性别：男" 等格式
- 表格感知提取：从 PDF/Word 表格中提取 2列/4列/6列 键值对，格式化为 "key：value" 追加到文本
- 内置法律字段：姓名、性别、民族、出生日期、职业、联系电话、律师事务所、统一社会信用代码、法定代表人、身份证号、案号等

### 2. 用户自定义提取字段（新 DB 表 + 设置对话框）
- 用户可添加/删除自定义字段名（如"标的金额"、"合同编号"）
- 存储在 SQLite `custom_fields` 表
- 提取时动态构建正则，合并内置+自定义字段

### 3. LLM 配置（新 DB 表 + 设置对话框 + llm_client.py）
- 8家 LLM 提供商（国内4 + 国外4），每家配置 base_url / api_key / model
- "测试连通"按钮（QThread 异步请求）
- 可选择一家作为当前活跃提供商

## 文件变更

### 修改文件
| 文件 | 改动 |
|------|------|
| `app/data/db_manager.py` | 新增 custom_fields、llm_providers 表及 CRUD 方法 |
| `app/core/nlp_engine.py` | 新增键值对提取，支持 custom_keys 参数 |
| `app/core/extractor.py` | PDF/Word 表格键值对提取 |
| `app/main_window.py` | 添加"设置"按钮，传递自定义字段到 NLP |
| `app/styles/song_dynasty.qss` | 新增 QTabWidget、测试按钮等样式 |

### 新建文件
| 文件 | 用途 |
|------|------|
| `app/core/llm_client.py` | LLM API 客户端（OpenAI 兼容格式 + Anthropic 特殊处理） |
| `app/widgets/settings_dialog.py` | 设置对话框（两个 Tab：自定义字段 + LLM 配置） |

## 实施顺序
1. `db_manager.py` — 新增表和方法（基础层）
2. `extractor.py` — 表格键值对提取
3. `nlp_engine.py` — 键值对提取 + custom_keys
4. `llm_client.py` — LLM 客户端
5. `settings_dialog.py` — 设置对话框 UI
6. `song_dynasty.qss` — 新样式
7. `main_window.py` — 串联所有功能

## LLM 提供商列表
| Key | 名称 | 默认 Base URL |
|-----|------|--------------|
| qwen | 通义千问 | https://dashscope.aliyuncs.com/compatible-mode/v1 |
| wenxin | 文心一言 | https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat |
| zhipu | 智谱AI | https://open.bigmodel.cn/api/paas/v4 |
| deepseek | DeepSeek | https://api.deepseek.com/v1 |
| openai | OpenAI | https://api.openai.com/v1 |
| anthropic | Anthropic | https://api.anthropic.com/v1 |
| gemini | Google Gemini | https://generativelanguage.googleapis.com/v1beta/openai |
| groq | Groq | https://api.groq.com/openai/v1 |

## 验证方式
1. 启动应用，点击"设置"按钮，确认两个 Tab 正常显示
2. 在"自定义字段"Tab 添加/删除字段，重启后仍存在
3. 在"LLM配置"Tab 填入 API 信息，点击"测试连通"确认结果
4. 导入含表格的 PDF/Word 文档，确认表格中的键值对被正确提取
5. 添加自定义字段名后重新导入文档，确认新字段被提取
