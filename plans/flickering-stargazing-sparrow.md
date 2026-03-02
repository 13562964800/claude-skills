# Groq CLI 聊天工具

## Context
用户已配置好 Groq API Key，希望开发一个简单的命令行聊天工具，在 Windows bash 下运行。

## 方案
用 Python 开发一个交互式 CLI 工具，支持与 Groq API 对话，流式输出。

## 功能
- 交互式对话（多轮上下文）
- 流式输出（打字机效果）
- 单次提问模式：`groq "你的问题"`
- 管道输入：`echo "翻译这段话" | groq`
- 支持选择模型
- 彩色终端输出

## 文件
```
~/groq-chat-app/
├── groq_cli.py         # 主程序
```

一个文件搞定，用已安装的 groq SDK。

## 用法
```bash
# 交互式聊天
python ~/groq-chat-app/groq_cli.py

# 单次提问
python ~/groq-chat-app/groq_cli.py "今天天气怎么样"

# 管道输入
cat code.py | python ~/groq-chat-app/groq_cli.py "解释这段代码"

# 指定模型
python ~/groq-chat-app/groq_cli.py -m llama-3.1-8b-instant "你好"
```

## 验证
运行 `python ~/groq-chat-app/groq_cli.py "你好"` 确认能正常返回响应。
