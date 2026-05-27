# Obsidian RAG 知识问答插件


基于 RAG（检索增强生成）的 Obsidian 知识库问答插件。将选定的笔记目录向量化，提问时自动检索最相关的内容作为上下文，由 LLM 生成带来源引用的回答。

## 想法来源
- 当我们想得到一些疑惑或者问题的解决方法时，自己的知识库是经过筛选过的符合自己认知与想法，可以很好接受和实践的知识来源，所以现从中提取与问题相关的方案是切实可行的，也能让自己可以回顾笔记内容尤其是技术以外的关于软实力等书籍所提供给我们的经验

## 功能

- **语义检索**：用自然语言提问，向量搜索匹配知识库中最相关的笔记
- **流式对话**：LLM 回答实时流式输出，支持随时停止生成
- **增量索引**：文件修改/新增/删除后 30 秒自动更新索引，只处理变更部分
- **目录筛选**：可选择仅索引特定文件夹，不索引整个 vault
- **双模型分离**：Embedding 模型和 LLM 模型独立配置，API Key / 接口地址 / 模型名均可自定义
- **Markdown 渲染**：回答内容支持 Markdown 格式化显示
- **多提供商兼容**：任何兼容 OpenAI API 的 Embedding 和 LLM 服务均可使用

## 架构

```
用户提问 → Embedding API（向量化问题）
         → Orama 向量搜索（相似度排序）
         → 读取 Top-K 原文
         → 构建 System Prompt（原文作上下文）
         → LLM API（流式生成回答）
```

- **向量引擎**：[Orama](https://orama.com) v3 — 纯 JS 向量数据库，索引持久化到本地 JSON 文件
- **嵌入模型**：支持任何 OpenAI 兼容的 Embedding API（默认阿里百炼 text-embedding-v3）
- **对话模型**：支持任何 OpenAI 兼容的 Chat Completions API（百炼 Qwen / OpenAI / DeepSeek / Ollama 等）
- **变更检测**：SHA-256 内容哈希 + 30s 防抖，增量更新避免重复调用 API

## 安装

1. 下载 `main.js`、`manifest.json`、`styles.css` 三个文件
2. 放入 Obsidian vault 的 `.obsidian/plugins/rag-kn/` 目录下
3. 重启 Obsidian，在「设置 → 第三方插件」中启用「RAG」

## 配置

插件设置分为四个区域：

### ① 向量嵌入（Embedding）
| 配置项 | 说明 | 默认值 |
|---|---|---|
| API Key | Embedding 服务的 API Key | — |
| 接口地址 | OpenAI 兼容的 Embedding 接口地址 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 模型 | Embedding 模型名称 | `text-embedding-v3` |
| 向量维度 | 256 / 512 / 1024 / 1536 | 512 |

### ② 大语言模型（LLM）
| 配置项 | 说明 | 默认值 |
|---|---|---|
| API Key | LLM 服务的 API Key | — |
| 接口地址 | OpenAI 兼容的接口地址 | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 模型 | 模型名称（如 qwen-plus） | `qwen-plus` |
| 最大输出 Tokens | 限制单次回答长度 | 4096 |
| 系统提示词 | 控制 AI 回答风格 | — |

### ③ 知识库范围
- **扫描文件夹**：选择要索引的目录，不选则扫描全部
- **检索数量**：每次问答检索的 Top-K 笔记数（默认 5）

### ④ 索引管理
- 显示当前已索引笔记数
- **重建索引**按钮：修改 Embedding 配置或维度后需要重建

## 常见服务配置

### Embedding 服务
| 服务 | 接口地址 | 模型 |
|---|---|---|
| 阿里百炼 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `text-embedding-v3` |
| OpenAI | `https://api.openai.com/v1` | `text-embedding-3-small` |

### LLM 服务
| 服务 | 接口地址 | 模型 |
|---|---|---|
| 阿里百炼 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-plus` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| 本地 Ollama | `http://localhost:11434/v1` | `llama3`（或已下载的模型） |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` |



## 技术要点

- **批量 + 重试**：Embedding API 以 10 条/批调用，失败自动重试 2 次（指数退避）
- **空内容过滤**：纯 frontmatter / 纯代码块的 md 文件会被跳过
- **文件名嵌入**：文件 basename 会拼入 embedding 文本，使书名/标题等文件名信息可被检索
- **相似度阈值**：向量搜索不设硬过滤门槛，靠 Top-K 排序取最相关结果
- **故障恢复**：索引重建失败时自动回滚旧索引，避免数据丢失
- **仅桌面端**：依赖 `crypto.subtle` 等 Electron API，不支持移动端 Obsidian

## 许可证

MIT
