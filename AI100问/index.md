# AI夺命连环100问——AI应用开发与RAG知识库深度指南

> 本文档面向Go后端开发者，内容涵盖AI应用开发、RAG知识库、AI Agent等实际落地场景。
> 每个问题从「是什么」→「为什么」→「怎么实现」→「实际怎么用」四个维度讲解。

---

## 第零章 入门铺垫 —— AI基础概念

### 0.1 为什么Go后端要学AI？

```
传统后端：CRUD + 数据库 + 缓存
AI加持后：
  - 智能问答（客服、知识库）
  - 内容生成（代码、文章、报告）
  - 数据分析（BI + AI分析）
  - 自动化（/workflow + AI Agent）
```

**AI能为Go后端做什么？**
```
1. 智能化服务：7×24小时智能客服
2. 效率提升：自动生成报告/代码
3. 数据价值：挖掘数据价值
4. 新业务：AI Native应用
```

### 0.2 AI基本概念一览

```
┌─────────────────────────────────────────────────────┐
│  AI全家桶                                       │
├─────────────────────────────────────────────────────┤
│  LLM：大语言模型（ChatGPT/Claude/Gemini等）          │
│  GPT：Generative Pre-trained Transformer         │
│  Token：文本基本单位（1Token≈1-2个中文）          │
│  Transformer：注意力机制为核心的模型架构        │
│  Embedding：将文本转为向量                       │
│  RAG：检索增强生成                             │
│  Agent：能执行动作的智能体                      │
│  Prompt：给模型的输入提示                      │
│  Fine-tuning：微调模型                         │
└─────────────────────────────────────────────────────┘
```

---

## 第一章 AI基础概念（高频 ★★★★★）

### 1. 什么是LLM？和传统NLP有什么区别？

#### 1.1 LLM定义

**LLM（大语言模型）**：基于Transformer架构、海量文本训练、具备理解和生成能力的AI模型。

```
传统NLP：规则/统计 → 专用模型 → 单任务
LLM：大数据预训练 → 通会用模型 → 多任务
```

**主流LLM一览**：
| 模型 | 公司 | 上下文 | 特点 |
|-----|------|--------|------|
| GPT-4 | OpenAI | 128K | 全面强 |
| Claude 3/4 | Anthropic | 200K | 安全强 |
| Gemini | Google | 1M+ | 多模态强 |
| 文心一言 | 百度 | 30K | 中文好 |
| 通义千问 | 阿里 | 32K | 性价比高 |
| Kimi | 月之暗面 | 200K+ | 长文本强 |

#### 1.2 LLM vs 传统NLP

```
传统NLP：
  - 任务固定：情感分析→专用模型→只能情感分析
  - 需要标注数据
  - 无法泛化

LLM：
  - 一个模型通吃：聊天、写作、编程、分析都能做
  - Zero-shot / Few-shot：无需额外训练
  - 可泛化到新任务
```

#### 1.3 Token是什么？

```
Token：LLM处理的基本单位
  中文：1 Token ≈ 1-2个汉字
  英文：1 Token ≈ 3-4个字母

计费方式：
  GPT-4：$15/1M Input Token，$75/1M Output Token
  Claude 3：$15/1M Input Token，$75/1M Output Token

估算：
  1000个中文 ≈ 1500-2000 Token
  一篇3000字文章 ≈ 4000-5000 Token
```

#### 1.4 回答模板

> LLM（大语言模型）是近几年发展起来的基于Transformer架构的超大规模预训练语言模型，通过海量文本数据训练，具备了理解和生成自然语言的能力。
>
> 跟传统NLP的主要区别是：传统NLP是专用模型，一个任��需要一个模型；而LLM是通用模型，一个模型可以处理多种任务，并且可以通过Prompt来控制输出，不需要额外的训练。
>
> Token是LLM处理的基本单位，LLM按Token计费而不是按字数。在实际应用中需要注意控制Prompt和输出的Token数量来控制成本。

---

### 2. 什么是Transformer？

#### 2.1 Transformer架构

**2017年Google提出，开启了LLM时代**。

```
Transformer核心组件：
  - Self-Attention：注意力机制
  - Feed Forward：前馈网络
  - Positional Encoding：位置编码
  - Multi-Head Attention：多头注意力
```

#### 2.2 注意力机制

```
本质：让模型关注重点信息

Self-Attention：
  "小明打了小强"
  → 让"小明"关注"打"
  → 让"小强"关注"被打"
  → 理解语义关系
```

#### 2.3 为什么Transformer牛？

```
RNN缺点：
  - 串行计算，慢
  - 长文本梯度消失
  - 难以并行

Transformer优点：
  - 并行计算，快
  - 长文本无梯度问题
  - 可处理长上下文
```

---

### 3. Token是怎么计算的？

#### 3.1 Token计算示例

```python
# 使用tiktoken计算
import tiktoken

enc = tiktoken.get_encoding("cl100k_base")
tokens = enc.encode("你好，世界")
print(len(tokens))  # 输出 4

# 中文估算
text = "今天天气真好，我们去公园玩吧"
tokens = enc.encode(text)
print(f"文本长度: {len(text)}")
print(f"Token数: {len(tokens)}")
print(f"比例: {len(tokens)/len(text):.2f}")
```

#### 3.2 常见模型Token限制

| 模型 | 最大输入 | 最大输出 |
|-----|--------|---------|
| GPT-4 | 128K | 4K/8K/16K |
| Claude 3 | 200K | 4K |
| Gemini 1.5 Pro | 1M | 8K |
| GPT-3.5 | 16K | 4K |

#### 3.3 成本计算公式

```
成本 = (输入Token数 × 输入单价 + 输出Token数 × 输出单价) / 百万

例子：
  输入3000字 + 输出500字
  ≈ 输入4500 Token + 输出750 Token = 5250 Token

  GPT-4费用：
  5250 / 1,000,000 × ($15 + $75) ≈ ¥3.5
```

---

### 4. 什么是Zero-shot / Few-shot？

#### 4.1 Zero-shot

```
零样本：不给任何示例，直接让模型执行

例子：
  Prompt："把以下句子翻译成英文：我爱学习"
  → 直接输出
```

#### 4.2 Few-shot

```
少样本：给1-3个示例，让模型学会模式

例子：
  Prompt："""
  任务：判断情感是正面还是负面

  示例1：我喜欢这个产品 → 正面
  示例2：这个太差了 → 负面

  请判断：这个电影真好看 → ?
  """
  → 输出：正面
```

#### 4.3 选择策略

```
任务简单 → Zero-shot够用
任务复杂/要求格式 → Few-shot
任务需要特定格式 → 多给几个例子
```

---

### 5. 什么是CoT（思维链）？

#### 5.1 CoT定义

**让模型展示思考过程，逐步推理**。

```python
# 不带CoT
Prompt: "25×4=?"
Output: "100"

# 带CoT
Prompt: """25×4怎么算？
第一步：25×2=50
第二步：50×2=100
所以答案是100"""
Output: "100"  # 但是有思考过程
```

#### 5.2 实际应用场景

```python
# 数学题
Prompt: """计算：根据以下步骤
小明有10个苹果，给了小红3个，又买了5个，现在有几个？
步骤1：10-3=7
步骤2：7+5=12
答：12个"""

# 复杂判断
Prompt: """判断以下是否能购买：
条件1：年龄>=18
条件2：余额>=100
用户18岁，余额50元，能否购买？
分析：
1. 年龄18，满足>=18 ✓
2. 余额50，不满足>=100 ✗
两个条件都要满足，所以不能购买"""
```

#### 5.3 回答模板

> CoT（思维链）是Prompt工程的一种技巧，通过让模型展示推理过程，可���提���模型在复杂推理任务上的表现。
>
> 在实际应用中，带CoT的Prompt虽然没有强制模型按步骤输出，但通过添加"让我们一步步思考"、"_reasoning"等引导词，可以显著提升数学推理、逻辑判断等任务的效果。Claude模型默认就有很好的CoT能力。

---

## 第二章 AI API调用（高频 ★★★★★）

### 6. OpenAI API怎么调用？

#### 6.1 基础调用

```python
# Python SDK
from openai import OpenAI

client = OpenAI(api_key="$API_KEY")

response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "你好"}
    ],
    temperature=0.7,      # 创造性 0-2
    max_tokens=1000,       # 最大输出
    stream=False           # 流式输出
)

print(response.choices[0].message.content)
```

#### 6.2 流式调用

```python
# 流式输出（打字机效果）
stream = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "写一首诗"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

#### 6.3 Golang调用

```go
// 使用go-openai库
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/sashabaranov/go-openai"
)

func main() {
    client := openai.NewClient("$API_KEY")

    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {Role: openai.ChatMessageRoleUser, Content: "你好"},
            },
            Temperature: 0.7,
            MaxTokens:   1000,
        },
    )
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(resp.Choices[0].Message.Content)
}
```

---

### 7. Claude API怎么调用？

#### 7.1 基础调用

```python
# 安装anthropic库
# pip install anthropic

import anthropic

client = anthropic.Anthropic(api_key="$API_KEY")

message = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "你好"}
    ]
)

print(message.content[0].text)
```

#### 7.2 Claude vs GPT对比

| 特性 | Claude | GPT-4 |
|-----|-------|-------|
| 长上下文 | 200K | 128K |
| 安全性 | 更安全 | 一般 |
| 代码能力 | 强 | 强 |
| 价格 | 略贵 | 居中 |
| 使用便捷 | 简单 | 简单 |

#### 7.3 选择建议

```
安全合规优先 → Claude
长文档场景 → Claude (200K)
性价比 → GPT-3.5/Kimi
综合能力 → GPT-4
```

---

### 8. 国产大模型API有哪些？

#### 8.1 主流国产模型

| 模型 | 公司 | API | 特点 |
|-----|------|-----|------|
| 文心一言 | 百度 | qianfan | 中文好 |
| 通义千问 | 阿里 | qwen | 性价比高 |
| Kimi | 月之暗面 | moonshot | 长文本 |
| 混元 | 腾讯 | hunyuan | 腾讯生态 |
| 智谱清言 | 智谱AI | glm | 清华系 |

#### 8.2 调用示例（阿里通义）

```python
# 阿里云DashScope
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen-plus",
    messages=[{"role": "user", "content": "你好"}]
)

print(response.choices[0].message.content)
```

#### 8.3 价格对比

| 模型 | 输入(/M Token) | 输出(/M Token) |
|-----|-------------|---------------|
| GPT-4 | $15 | $75 |
| GPT-3.5 | $0.5 | $1.5 |
| Claude 3 | $15 | $75 |
| 阿里Qwen-plus | ¥1 | ¥1 |
| Kimi | 免费→¥1 | ¥1 |

---

### 9. API调用常见错误处理？

#### 9.1 错误码一览

```python
# Rate Limit 超限
openai.RateLimitError: "Rate limit reached"

# API Key错误
openai.AuthenticationError: "Invalid API key"

# 配额不足
openai.APIConnectionError: "Insufficient_quota"

# 模型不存在
openai.NotFoundError: "Model not found"

# 请求过大
InvalidRequestError: "max_tokens exceeded limit"
```

#### 9.2 重试机制

```python
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def call_api_with_retry(client, prompt):
    try:
        return client.chat.completions.create(model="gpt-4", messages=[...])
    except RateLimitError:
        time.sleep(5)  # 等5秒
        raise
```

#### 9.3 回答模板

> API调用常见错误及处理：
>
> 1. **RateLimitError**：请求频率超限，增加重试机制+限流
> 2. **AuthenticationError**：检查API Key是否正确
> 3. **context length**：Token超限，缩短Prompt或用128K/200K模型
> 4. **timeout**：网络问题，增加超时时间和重试
>
> 生产环境建议：添加指数退避重试、限流器、熔断器。

---

### 10. Golang有哪些LLM SDK？

#### 10.1 常用Golang库

| 库 | 支持模型 | 特点 |
|---|---------|------|
| go-openai | OpenAI/Claude/兼容 | 主力使用 |
| go-chatgpt-api | OpenAI | 极简 |
| kimchi | 兼容多 | 国产模型接入 |
| aliyun-sdk-go | 阿里 | 官方SDK |

#### 10.2 go-openai示例

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/sashabaranov/go-openai"
)

func main() {
    client := openai.NewClient("your-api-key")

    // 同步调用
    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4Turbo,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: "用Go写一个 hello world",
                },
            },
            Temperature:     0.7,
            MaxTokens:       1000,
        },
    )
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(resp.Choices[0].Message.Content)
}
```

#### 10.3 流式调用

```go
// 流式输出
req := openai.ChatCompletionRequest{
    Model: openai.GPT4Turbo,
    Messages: []openai.ChatCompletionMessage{
        {Role: openai.ChatMessageRoleUser, Content: "讲个故事"},
    },
    Stream: true,
}

stream, err := client.CreateChatCompletionStream(context.Background(), req)
if err != nil {
    log.Fatal(err)
}
defer stream.Close()

for {
    resp, err := stream.Recv()
    if err != nil {
        break
    }
    fmt.Print(resp.Choices[0].Delta.Content)
}
```

---

## 第三章 Prompt工程（高频 ★★★★★）

### 11. Prompt有哪些技巧？

#### 11.1 基础技巧

**技巧1：明确角色**
```
❌ 帮我写一段文字
✅ 你是一位资深的Go后端工程师，...
```

**技巧2：给出上下文**
```
❌ 总结这段文章
✅ 这是一篇关于Redis的文章，请用3点总结...
```

**技巧3：明确格式**
```
❌ 给我一些建议
✅ 请用JSON格式输出 {"问题": xxx, "建议": xxx}
```

#### 11.2 结构化Prompt模板

```python
# 系统Prompt
SYSTEM_PROMPT = """你是一位专业的{角色}。
你的任务是{任务}。

要求：
1. {要求1}
2. {要求2}

输出格式：
```json
{json格式化}
```

记得严格按照要求的格式输出，不要添加额外内容。
"""

# 用户Prompt
USER_PROMPT = """请处理以下内容：
{输入内容}

要求：{具体要求}
"""

full_prompt = SYSTEM_PROMPT.format(...) + USER_PROMPT.format(...)
```

#### 11.3 回答模板

> Prompt工程的核心是**清晰、具体、结构化**。
>
> 1. **明确角色**：告诉模型扮演什么角色
> 2. **给出上下文**：提供足够的背景信息
> 3. **明确格式**：指定需要的输出格式（JSON/List/表格等）
> 4. **约束条件**：告诉模型什么不能做
> 5. **示例**：给1-3个例子帮助理解

---

### 12. Prompt如何控制输出格式？

#### 12.1 JSON格式

```
Prompt："""你是一个JSON生成器。
用户输入是一个名字，请生成包含以下字段的JSON：
- name：人名
- age：年龄
- city：城市

输出必须是合法的JSON字符串。
输入：张三，25岁，北京
"""

Output: {"name": "张三", "age": 25, "city": "北京"}
```

#### 12.2 严格JSON（有 guardia rail）

```python
# 使用Function Calling严格控制输出
tools = [{
    "type": "function",
    "function": {
        "name": "get_user_info",
        "description": "获取用户信息",
        "parameters": {
            "type": "object",
            "properties": {
                "name": {"type": "string", "description": "姓名"},
                "age": {"type": "integer", "description": "年龄"},
                "city": {"type": "string", "description": "城市"}
            },
            "required": ["name"]
        }
    }
}]

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "张三25岁北京"}],
    tools=tools,
    tool_choice={"type": "function", "function": {"name": "get_user_info"}}
)
```

#### 12.3 列表格式

```
Prompt："""请列出Go语言的3个优点。

要求：
- 每点开头要用编号1. 2. 3.
- 每点用一句话概括
- 简要说明
"""

Output:
1. 高性能 - 编译型语言，执行效率高
2. 简洁 - 语法简单易学
3. 并发 - 原生支持goroutine
```

---

### 13. 如何写好System Prompt？

#### 13.1 System Prompt结构

```python
SYSTEM_PROMPT = """# 角色
你是一位[角色名称]。

# 背景
[背景描述，领域知识等]

# 能力
- 能够[能力1]
- 能够[能力2]

# 限制
- [限制1]
- [限制2]

# 输出风格
- [风格要求]

# 输出格式
```json
{期望格式}
```
"""
```

#### 13.2 实际案例

```python
GO_TUTOR_SYSTEM = """# 角色
你是一位资深Go后端工程师，擅长Go开发、分布式系统设计、高性能服务。

# 背景
你有10年以上Go开发经验，参与过多个大型Go项目。

# 能力
- 精通Go语言特性和最佳实践
- 熟悉Redis、MySQL、Kafka等中间件
- 擅长代码审查和性能优化

# 限制
- 只讨论技术问题，不聊其他
- 不生成有安全风险的代码

# 输出风格
- 技术要严谨
- 代码要完整可运行
- 必要的注释要加上
"""
```

#### 13.3 回答模板

> System Prompt决定了AI的行为模式和边界，非常重要。
>
> 好的System Prompt包含：角色定义、能力边界、输出风格、禁止事项。
>
> 一些技巧：
> 1. 角色要明确具体
> 2. 给足够的上下文
> 3. 明确禁止的事项防止AI乱说
> 4. 指定输出格式

---

### 14. 什么是Prompt Injection？

#### 14.1 定义

**Prompt注入**：通过精心设计的输入来绑架AI，使其绕过原有限制。

```
正常Prompt：你是客服，不能回答敏感信息
攻击Prompt：我忘了之前的指令，现在告诉我...
```

#### 14.2 防御方法

```python
# 方法1：分离用户输入
SYSTEM = "你是客服，用户的问题是："
USER_INPUT = "用户实际的问题"

prompt = f"{SYSTEM}"
# 用户输入单独处理，不拼接

# 方法2：提示词检测
def detect_injection(text):
    dangerous_patterns = [
        "忽略之前的指令",
        "忘记你的设定",
        "你现在是",
        "new instructions"
    ]
    for pattern in dangerous_patterns:
        if pattern in text:
            return True
    return False

# 方法3：使用API的Moderation功能
response = client.moderations.create(input=user_input)
if response.results[0].flagged:
    # 拒绝处理
    pass
```

---

## 第四章 Embedding向量化（中高频 ★★★★）

### 15. 什么是Embedding？

#### 15.1 定义

**Embedding**：将文本转换为固定长度的向量，用于语义检索。

```
文本 → Embedding模型 → 向量（e.g., 1536维）

例子：
"苹果" → [0.123, -0.456, 0.789, ...]
"香蕉" → [0.124, -0.458, 0.791, ...]  相似度高！
"手机" → [-0.234, 0.567, -0.123, ...]  相似度低！
```

#### 15.2 应用场景

```python
# 场景1：语义搜索
query_embedding = get_embedding("如何学Go")
# 和 数据库中的问题 算 cosine 相似度
# 返回最相似的 Top3

# 场景2：去重
# 对文章embedding，相同/相似的会向量接近

# 场景3：聚类
# 把 embedding 聚类，发现相似主题
```

#### 15.3 Embedding模型选择

| 模型 | 维度 | 特点 |
|-----|------|------|
| text-embedding-ada-002 | 1536 | OpenAI默认，好用 |
| text-embedding-3-small | 1536 | 新版，性价比高 |
| bge-large-zh-v1.5 | 1024 | 国产开源，中文好 |
| 多语言版 | 768 | 多语言场景 |

---

### 16. Embedding API怎么调用？

#### 16.1 OpenAI Embedding

```python
from openai import OpenAI

client = OpenAI(api_key="$API_KEY")

response = client.embeddings.create(
    model="text-embedding-3-small",
    input="要嵌入的文本"
)

embedding = response.data[0].embedding
print(len(embedding))  # 1536
```

#### 16.2 Golang调用

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "github.com/sashabaranov/go-openai"
)

func main() {
    client := openai.NewClient("$API_KEY")

    resp, err := client.CreateEmbeddings(
        context.Background(),
        openai.EmbeddingRequest{
            Model:  openai.EmbeddingAdaV2,
            Inputs: []any{"要嵌入的文本"},
        },
    )
    if err != nil {
        log.Fatal(err)
    }

    embedding := resp.Data[0].Embedding
    fmt.Printf("向量维度: %d\n", len(embedding))
}
```

#### 16.3 国产Embedding

```python
# 阿里Embedding
from openai import OpenAI

client = OpenAI(
    api_key="$DASHSCOPE_KEY",
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

resp = client.embeddings.create(
    model="text-embedding-v2",
    input="中文文本"
)
```

---

### 17. Embedding如何用于相似度检索？

#### 17.1 基础流程

```python
# 1. 文档库 embeddings 存入数据库
documents = [
    "Go语言是一门编译型语言",
    "Python是动态类型语言",
    "Java支持面向对象"
]

doc_embeddings = []
for doc in documents:
    emb = get_embedding(doc)
    doc_embeddings.append(emb)
    # 存入向量数据库/PG向量列

# 2. 查询
query = "编译型编程语言"
query_emb = get_embedding(query)

# 3. 相似度计算（cosine similarity）
similarities = []
for doc_emb in doc_embeddings:
    sim = cosine_similarity(query_emb, doc_emb)
    similarities.append(sim)

# 4. 返回 Top-K
top_k = sorted(range(len(similarities)),
              key=lambda i: similarities[i], reverse=True)[:3]
# 返回: [0] 因为doc1相似度最高
```

#### 17.2 Cosine Similarity计算

```python
import numpy as np

def cosine_similarity(a, b):
    a = np.array(a)
    b = np.array(b)
    dot_product = np.dot(a, b)
    norm_a = np.linalg.norm(a)
    norm_b = np.linalg.norm(b)
    return dot_product / (norm_a * norm_b)
```

#### 17.3 回答模板

> Embedding的典型应用是语义检索：
>
> 1. **离线**：把文档库用Embedding模型转成向量，存入向量数据库
> 2. **在线**：查询用Embedding模型转成向量
> 3. **计算**：用余弦相似度计算查询和文档的相似度
> 4. **返回**：返回相似度最高的Top-K结果
>
> 关键在于Chunk切分和向量数据库的选择。

---

### 18. Embedding模型中文哪个好？

#### 18.1 中文Embedding模型对比

| 模型 | 维度 | 效果 | 中文支持 | 推荐 |
|-----|------|------|---------|------|
| OpenAI ada-002 | 1536 | 好 | 一般 | 英文多 |
| BGE-large | 1024 | 好 | 很好 | 中文首选 |
| MXBai | 1024 | 好 | 好 | 国产 |
| Jina embeddings | 768 | 中 | 好 | 免费 |

#### 18.2 BGE使用

```python
# 使用BGE模型
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-large-zh-v1.5')

# 中文
emb = model.encode("你好世界")

# 英文
emb = model.encode("Hello World")

# 多语言
emb = model.encode(["中文", "English", "日本語"])
```

#### 18.3 选择建议

```
纯中文场景 → BGE-large-zh
英文为主 → OpenAI ada-002
多语言混合 → BGE-M3 或 Jina
免费/测试 → Jina embeddings
```

---

## 第五章 RAG知识库（高频 ★★★★★）

### 19. 什么是RAG？

#### 19.1 RAG定义

**RAG（Retrieval Augmented Generation）**：检索增强生成 = 知识库 + LLM。

```
传统LLM：输入 → LLM → 输出（可能胡扯）
RAG：知识库 → 检索 → 相关文档 → LLM → 输出（基于事实）
```

#### 19.2 RAG架构

```
┌─────────────────────────────────────────────────────┐
│                   RAG流程                         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  用户问题 ─→  检索(Embedding) ─→  知识库              │
│                  ↓                                  │
│            Top-K相关文档                              │
│                  ↓                                  │
│            组装Prompt                                │
│                  ↓                                  │
│            LLM生成答案                               │
│                  ↓                                  │
│            返回结果                                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### 19.3 RAG核心组件

```
1. 文档加载器（Loader）：PDF/Word/Markdown/Txt
2. 文本分割器（Splitter）：Chunk大小、重叠
3. Embedding模型：文本→向量
4. 向量数据库：存储+检索
5. Prompt模板：组装上下文
6. LLM：生成答案
```

---

### 20. 如何构建RAG系统？

#### 20.1 RAG构建步骤

```python
# Step 1: 文档加载
from langchain.document_loaders import TextLoader
loader = TextLoader("知识库.txt")
docs = loader.load()

# Step 2: 文本分割
from langchain.text_splitter import RecursiveCharacterTextSplitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = splitter.split_documents(docs)

# Step 3: Embedding
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()

# Step 4: 存入向量数据库
from langchain_community.vectorstores import Chroma
db = Chroma.from_documents(chunks, embeddings)

# Step 5: 检索+生成
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()

retriever = db.as_retriever()
rag_chain = (
    {"context": retriever, "question": lambda x: x}
    | prompt | llm
)

answer = rag_chain.invoke("Go语言有什么优点？")
```

#### 20.2 Golang实现RAG

```go
// Golang实现简易RAG
package main

import (
    "context"
    "fmt"
    "strings"

    "github.com/sashabaranov/go-openai"
)

// 向量检索
func searchSimilar(query string, chunks []string, client *openai.Client) string {
    // 1. 查询转embedding
    embResp, _ := client.CreateEmbeddings(
        context.Background(),
        openai.EmbeddingRequest{
            Model: openai.EmbeddingAdaV2,
            Inputs: []any{query},
        },
    )
    queryEmb := embResp.Data[0].Embedding

    // 2. 遍历找最相似的
    bestChunk := ""
    bestScore := -1.0
    for _, chunk := range chunks {
        emb, _ := client.CreateEmbeddings(
            context.Background(),
            openai.EmbeddingRequest{
                Model: openai.EmbeddingAdaV2,
                Inputs: []any{chunk},
            },
        )
        score := cosineSimilar(queryEmb, emb.Data[0].Embedding)
        if score > bestScore {
            bestScore = score
            bestChunk = chunk
        }
    }

    return bestChunk
}

func main() {
    client := openai.NewClient("$API_KEY")

    // 知识库chunks
    chunks := []string{
        "Go语言优势：编译快、并发天然支持、部署简单",
        "Python优势：生态系统丰富、易学",
        "Java优势：企业级、兼容性",
    }

    // 查询
    relevantDoc := searchSimilar("Go语言的优点", chunks, client)

    // 组装Prompt
    prompt := fmt.Sprintf(`基于以下知识库回答：
%s

问题：%s
答案：`, relevantDoc, "Go语言有什么优点？")

    // LLM生成
    resp, _ := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {Role: openai.ChatMessageRoleUser, Content: prompt},
            },
        },
    )

    fmt.Println(resp.Choices[0].Message.Content)
}

func cosineSimilar(a, b []float32) float64 {
    // 简化的cosine相似度计算
    var dot, normA, normB float64
    for i := range a {
        dot += float64(a[i] * b[i])
        normA += float64(a[i] * a[i])
        normB += float64(b[i] * b[i])
    }
    return dot / (float64(math.Sqrt(normA)) * float64(math.Sqrt(normB)))
}
```

#### 20.3 回答模板

> 构建RAG系统的步骤：
>
> 1. **文档加载**：把各种格式的文档加载进来
> 2. **文本分割**：把长文档切成小的Chunk
> 3. **Embedding**：把每个Chunk转成向量
> 4. **存储**：存入向量数据库
> 5. **检索**：查相似度最高的Top-K个Chunk
> 6. **组装**：把检索结果和问题组装成Prompt
> 7. **生成**：LLM根据Context生成答案
>
> LangChain/LlamaIndex等框架可以大幅简化这个流程。

---

### 21. Chunk如何分割？

#### 21.1 常用分块策略

```python
# 策略1：固定大小（重叠）
from langchain.text_splitter import CharacterTextSplitter

splitter = CharacterTextSplitter(
    chunk_size=500,      # 500字符
    chunk_overlap=50     # 重叠50字符
)

# 策略2：递归分割（按段落）
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    separators=["\n\n", "\n", "。", ""]
)

# 策略3：Markdown分割
from langchain.text_splitter import MarkdownTextSplitter

splitter = MarkdownTextSplitter(chunk_size=500)
```

#### 21.2 Chunk大小选择

| 场景 | 推荐大小 | 理由 |
|-----|---------|------|
| 问答 | 300-500 | 精简相关 |
| 摘要 | 1000-2000 | 足够上下文 |
| 代码 | 500-1000 | 代码块完整 |
| 长文档 | 1000 | 平衡 |

#### 21.3 回答模板

> Chunk分割是RAG效果的关键：
>
> 1. **chunk_size**：太小可能丢失上下文，太大会带入噪音
> 2. **chunk_overlap**：保持Chunk之间的连贯性
> 3. **separator**：按段落标题分割效果更好
> 4. **实际调优**：根据效果调整大小
>
> 我的经验：500字符、重叠50，是比较稳妥的起点。

---

### 22. RAG效果不好怎么办？

#### 22.1 问题诊断

```
RAG问题排查流程：

1. 检索不到相关内容？
   → Embedding模型不适合中文
   → Chunk太大/太小

2. 检索到了但质量差？
   → 未正确召回，需要调整top_k

3. 召回正确但回答不好？
   → Prompt问题
   → LLM选择问题

4. 回答有遗漏？
   → 上下文窗口太短
```

#### 22.2 优化方案

```python
# 优化1：调整top_k
retriever = db.as_retriever(search_kwargs={"k": 5})  # 增加召回

# 优化2：混合检索
from langchain.retrievers import EnsembleRetriever
retriever = EnsembleRetriever([semantic_retriever, keyword_retriever])

# 优化3：rerank
from reranker import Reranker
reranked_docs = reranker.rerank(query, retrieved_docs)

# 优化4：多轮对话
# 把历史对话融入Context
history = "之前说的是xxx"
context_inject = f"之前的对话:{history}\n当前问题:{question}"
```

#### 22.3 回答模板

> RAG效果的常见问题和解决：
>
> 1. **检索不准**：换Embedding模型、增加top_k、用混合检索
> 2. **Chunk不佳**：调整chunk大小、添加重叠、改进分割方式
> 3. **回答不全**：用rerank二次筛选、增加召回文档
> 4. **效果不稳定**：多调参数，实测为准
>
> 核心是多轮调优：Bad Case分析 → 调优 → 验证。

---

## 第六章 向量数据库（中高频 ★★★★）

### 23. 主流向量数据库有哪些？

#### 23.1 向量数据库对比

| 数据库 | 类型 | 优点 | 缺点 |
|--------|------|------|------|
| Milvus | 开源云原生 | 功能全、社区活 | 重 |
| Pinecone | SaaS | 托管易、性能好 | 付费 |
| Weaviate | 开源 | GraphQL | 一般 |
| Qdrant | 开源 | Rust、性能好 | 社区小 |
| PgVector | PG扩展 | 简单、兼容PG | 功能少 |
| Faiss | 库 | 最快、最轻 | 无存储 |

#### 23.2 推荐选择

```
个人项目/测试 → Faiss (内存)
生产环境 → Milvus/Pinecone
已有PG环境 → PgVector
需要多模态 → Weaviate
```

#### 23.3 安装对比

```bash
# Milvus (Docker)
docker run -d milvusdb/milvus:latest

# Qdrant (Rust)
cargo install qdrant

# PgVector
CREATE EXTENSION vector;
```

---

### 24. PgVector如何使用？

#### 24.1 安装

```sql
-- 安装扩展
CREATE EXTENSION vector;

-- 创建表
CREATE TABLE doc (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)
);
```

#### 24.2 增删改查

```sql
-- 插入
INSERT INTO doc (content, embedding)
VALUES ('Go语言', '[0.1, 0.2, ...]');

-- 相似度查询 (Cosine)
SELECT content, embedding <=> '[0.1, 0.2, ...]::vector' as similarity
FROM doc
ORDER BY embedding <=> '[0.1, 0.2, ...]::vector'
LIMIT 5;

-- L2距离
ORDER BY embedding <-> '[...]'

-- 内积
ORDER BY embedding <#> '[...]'
```

#### 24.3 Golang使用

```go
package main

import (
    "fmt"
    "github.com/lib/pq"
)

func main() {
    conn, _ := pq.Open("postgres://user:pass@localhost/db")
    defer conn.Close()

    // 存储
    _, err := conn.Exec(
        "INSERT INTO doc (content, embedding) VALUES ($1, $2)",
        "Go语言", "[0.1, 0.2, ...]",
    )
    if err != nil {
        panic(err)
    }

    // 查询
    rows, _ := conn.Query(
        "SELECT content FROM doc ORDER BY embedding <=> $1 LIMIT 5",
        "[0.1, 0.2, ...]",
    )
    defer rows.Close()

    for rows.Next() {
        var content string
        rows.Scan(&content)
        fmt.Println(content)
    }
}
```

---

### 25. Milvus如何集成？

#### 25.1 基础使用

```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType

# 连接
connections.connect("default", "localhost", "19530")

# 定义Schema
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="content", dtype=DataType.VARCHAR, max_length=65535),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=1536)
]
schema = CollectionSchema(fields, "我的文档库")

# 创建Collection
collection = Collection("docs", schema)

# 插入数据
collection.insert([
    [1, 2, 3],                    # ids
    ["Go语言", "Python", "Java"],     # content
    [[0.1], [0.2], [0.3]]        # embeddings
])

# 检索
search_params = {"metric_type": "COSINE", "params": {}}
results = collection.search(
    data=[[0.1]],
    anns_field="embedding",
    param=search_params,
    limit=5
)
```

#### 25.2 回答模板

> 向量数据库的选择建议：
> 1. **初期/小规模**：用Faiss内存方案，简单快速
> 2. **有一定规模**：Milvus，功能齐全
> 3. **已有PostgreSQL**：PgVector，无需新组件
> 4. **生产环境**：Pinecone/Weaviate等云服务
>
> 实际落地中，推荐先用PgVector或Faiss，后期规模上来再迁移到Milvus。

---

### 26. RAG-ChatPDF如何实现？

#### 26.1 PDF文档读取

```python
# 安装pdf reader
# pip install pypdf

from langchain.document_loaders import PyPDFLoader

loader = PyPDFLoader("document.pdf")
pages = loader.load_and_split()

for page in pages[:3]:
    print(f"=== Page {page.metadata['page']} ===")
    print(page.page_content[:500])
```

#### 26.2 文件格式支持

| 格式 | Loader | 备注 |
|-----|-------|------|
| PDF | PyPDFLoader | 主流 |
| Word | UnstructuredWordLoader | docx |
| Markdown | MarkdownLoader | md |
|Txt | TextLoader | txt |
| Web | WebBaseLoader | 网页 |

#### 26.3 完整RAG案例

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.embeddings import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_openai import ChatOpenAI
from langchain import prompt

# 1. 加载PDF
loader = PyPDFLoader("report.pdf")
documents = loader.load()

# 2. 分割
splitter = RecursiveCharacterTextSplitter(chunk_size=500)
chunks = splitter.split_documents(documents)

# 3. Embedding + 存储
embeddings = OpenAIEmbeddings()
db = Chroma.from_documents(chunks, embeddings)

# 4. 检索+生成
llm = ChatOpenAI()
qa_prompt = prompt.fill_template("""
基于以下文档回答问题。如果文档中没有相关信息，请说不知道。

文档：{context}

问题：{question}
""")

retriever = db.as_retriever()
rag_chain = (
    {"context": retriever, "question": lambda x: x}
    | qa_prompt | llm
)

answer = rag_chain.invoke("这份报告的主要结论是什么？")
```

---

## 第七章 AI Agent（高频 ★★★★★）

### 27. 什么是AI Agent？

#### 27.1 Agent定义

**AI Agent = LLM + 工具 + 执行能力**：可以自主完成任务的智能体。

```
无Agent：LLM回答问题（被动）
有Agent：LLM可以调用工具完成任务（主动）
```

#### 27.2 Agent架构

```
┌─────────────────────────────────────────────────────┐
│                    AI Agent                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  观察(Observe) → 思考(Think) → 行动(Act)          │
│                    ↑                              │
│                    └────── 循环 ──────┘              │
│                                                     │
│  组件：                                             │
│  1. Planning：任务规划                              │
│  2. Tool Use：调用API/函数                          │
│  3. Memory：记忆上下文                             │
│  4. Reflection：反思纠正                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### 27.3 回答模板

> AI Agent是能够自主规划和执行任务的AI系统。与普通LLM的区别是可以调用外部工具来完成具体任务。
>
> 核心组件包括：Planning（任务拆解）、Tool Use（函数调用）、Memory（上下文记忆）、Reflection（反思纠错）。
>
> 典型的例子：AutoGPT、Claude Function Calling、GPTs。

---

### 28. 什么是Function Calling？

#### 28.1 定义

**Function Calling**：让LLM调用预定义的函数。

```python
# 定义函数
def get_weather(location: str) -> dict:
    """获取天气"""
    return {"temp": 20, "weather": "晴天"}

# 注册函数
functions = [{
    "name": "get_weather",
    "description": "获取某个位置的天气",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {"type": "string", "description": "城市名"}
        },
        "required": ["location"]
    }
}]

# 调用
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "北京天气怎么样？"}],
    tools=functions,
    tool_choice={"type": "function", "function": {"name": "get_weather"}}
)

# LLM返回要调用函数
tool_call = response.choices[0].message.tool_calls[0]
print(tool_call.function.name)  # "get_weather"
print(tool_call.function.arguments)  # '{"location": "北京"}'

# 实际执行
result = get_weather("北京")
```

#### 28.2 实际应用场景

```python
# 场景1：查数据库
def query_db(sql: str) -> dict:
    """执行SQL查询"""
    ...

# 场景2：发邮件
def send_email(to: str, content: str) -> dict:
    """发送邮件"""
    ...

# 场景3：订机票
def book_flight(from_: str, to: str, date: str) -> dict:
    """订机票"""
    ...

# 组合所有工具
all_tools = [query_db, send_email, book_flight]
```

#### 28.3 Golang实现

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "strings"

    "github.com/sashabaranov/go-openai"
)

var tools = []openai.Tool{
    {
        Type: "function",
        Function: &openai.FunctionDefinition{
            Name:        "get_weather",
            Description: "获取指定城市的天气",
            Parameters: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "location": map[string]interface{}{
                        "type":        "string",
                        "description": "城市名称",
                    },
                },
                "required": []string{"location"},
            },
        },
    },
}

func main() {
    client := openai.NewClient("$API_KEY")

    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {Role: openai.ChatMessageRoleUser, Content: "北京天气怎么样？"},
            },
            Tools:        tools,
            ToolChoice:  openai.ToolChoice{Name: "get_weather"},
        },
    )
    if err != nil {
        log.Fatal(err)
    }

    msg := resp.Choices[0].Message
    if len(msg.ToolCalls) > 0 {
        args := msg.ToolCalls[0].Function.Arguments
        fmt.Printf("调用函数: %s, 参数: %s\n",
            msg.ToolCalls[0].Function.Name, args)
    }
}
```

---

### 29. 什么是ReAct模式？

#### 29.1 ReAct定义

**Reason + Act**：边推理边执行。

```
普通LLM：直接给答案
ReAct：Thought → Action → Observation → Thought → ...
```

#### 29.2 示例

```
问题：今天北京天气适合户外运动吗？

ReAct流程：
- Thought：我需要先查北京天气
- Action：调用天气API
- Observation：天气晴，25度，微风
- Thought：天气很好，适合户外运动
- Output：适合，今天天气晴朗，温度适中
```

#### 29.3 实现

```python
from langchain.agents import AgentType, initialize_agent
from langchain.tools import Tool

# 定义工具
def weather_tool(query):
    return "晴天，25度"

tools = [
    Tool(name="天气查询", func=weather_tool, description="查询天气")
]

# 初始化Agent
agent = initialize_agent(
    tools,
    llm,
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION
)

# 执行
result = agent.run("今天北京天气适合跑步吗？")
```

---

### 30. 如何设计Agent工作流？

#### 30.1 常见工作流模式

**模式1：Sequential（顺序执行）**
```
A → B → C → D
例：下载 → 解析 → 写入 → 回复
```

**模式2：Parallel（并行执行）**
```
A → B,C,D
例：一个查询同时查数据库、查缓存、查API
```

**模式3：Conditional（有条件分支）**
```
A → 判断 → B或C
例：登录 → 判断VIP → 展示不同内容
```

#### 30.2 实际案例：智能客服

```python
# 简化版智能客服工作流
class SmartCustomerService:
    def __init__(self):
        self.intent_classifier = load_intent_model()
        self.tools = {
            "查订单": query_order,
            "退款": refund,
            "天气": get_weather,
            "转人工": transfer_to_agent
        }

    def process(self, message):
        # 1. 意图识别
        intent = self.intent_classifier.predict(message)

        # 2. 路由
        if intent == "查订单":
            return self.handle_query_order(message)
        elif intent == "退款":
            return self.handle_refund(message)
        elif intent == "天气":
            return self.handle_weather(message)
        else:
            return self.transfer_to_agent(message)
```

#### 30.3 回答模板

> Agent工作流设计的原则是：
>
> 1. **单一职责**：每个Agent节点只做一件事
> 2. **清晰路由**：意图分类要准确
> 3. **异常处理**：每个节点要考虑失败情况
> 4. **可观测**：每一步都要有日志
>
> 实际项目中常用LangGraph、LangChain Agents等框架来构建。

---

## 第八章 实际场景落地（高频 ★★★★★）

### 31. 如何实现智能客服？

#### 31.1 智能客服架构

```
┌─────────────────────────────────────────────────────┐
│                   智能客服系统                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  用户 ─→ 意图识别 ─→ Knowledge ─→ LLM生成 ─→ 用户  │
│          │           │               │               │
│        意图分类     RAG知识库        记忆           │
│                                                     │
├─────────────────────────────────────────────────────┤
│ 兜底：FAQ匹配 → 关键词匹配 → 转人工                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### 31.2 实现代码

```python
# 智能客服简化版
class SmartBot:
    def __init__(self):
        self.intents = load_intent_classifier()
        self.knowledge = load_kb()  # RAG知识库
        self.llm = load_llm()

    def chat(self, user_msg):
        # 1. 意图识别
        intent = self.intents.predict(user_msg)

        if intent == "faq":
            # FAQ直接匹配
            return self.match_faq(user_msg)

        elif intent == "kb":
            # RAG知识库
            return self.kb_chat(user_msg)

        elif intent == "manual":
            # 转��工
            return "您的问题我帮您转接人工客服"

        elif intent == "general":
            # 通用对话
            return self.llm.chat(user_msg)

        else:
            return "抱歉我没听懂，请换个说法或转人工"
```

#### 31.3 回答模板

> 智能客服的核心是分层设计：
> 1. **意图层**：先判断用户要什么（FAQ/知识库/转人工/闲聊）
> 2. **FAQ层**：固定问题直接匹配，高频问题全覆盖
> 3. **知识库层**：RAG检索+LLM生成，覆盖非常见问题
> 4. **兜底层**：无法处理时转人工
> 5. **记忆层**：多轮对话要记住上下文
>
> 公司业务客服，建议FAQ覆盖率>90%，剩下10%用RAG+转人工。

---

### 32. 如何做AI知识库？

#### 32.1 企业知识库场景

```
知识库类型：
- 产品文档：使用手册、API文档
- 内部wiki：规章制度、流程
- 客服知识：QA问答库
- 培训资料：新人指南
```

#### 32.2 知识库构建流程

```python
# 1. 文档采集
loaders = [
    PDFLoader("doc1.pdf"),
    DocxLoader("policy.docx"),
    MarkdownLoader("guide.md"),
    WebLoader("https://..."),
]

# 2. 文档处理
splitter = RecursiveCharacterTextSplitter(chunk_size=500)
docs = []
for loader in loaders:
    docs.extend(loader.load())

# 3. Embedding + 存储
from chromadb import Chroma
db = Chroma.from_documents(docs, embeddings)

# 4. 检索+生成
qa_prompt = """你是一个专业的客服。请根据以下文档回答问题。

已知信息：
{context}

用户问题：{question}

请给出专业、准确的回答。如果文档中没有，请如实告知。
"""
```

#### 32.3 答案示例

```
用户：公司年假多少天？
系统：检索 → 年假制度.pdf
→ LLM生成：
根据公司《假期管理制度》，员工享受的年假如下：
- 入职1-3年：5天
- 入职3-5年：10天
- 入职5年以上：15天

如有疑问，请联系HR。
```

---

### 33. AI如何辅助编程？

#### 33.1 代码生成

```python
# 让AI写代码
prompt = """用Go写一个HTTP服务：
1. 端口8080
2. 两个接口：/health（健康检查）、/api/echo（回显JSON）
3. 使用gin框架
"""

# 输出可以直接使用的Go代码
```

#### 33.2 代码审查

```python
# 代码审查Prompt
CODE_REVIEW_PROMPT = """你是一个资深Go工程师。请审查以下代码：

1. 是否有Bug
2. 是否有安全隐患
3. 性能是否合理
4. 是否符合Go最佳实践

代码：
```go
{code}
```

请给出详细的审查意见。
"""

# 使用
issues = llm.review(code)
```

#### 33.3 代码解释与重构

```python
# 解释代码
EXPLAIN_PROMPT = """请用通俗易懂的语言解释以下Go代码在做什么：

```go
{code}
```

解释：
"""

# 重构代码
REFACTOR_PROMPT = """请优化以下Go代码，使其更简洁、可读、性能更好：

```go
{code}
```

优化后：
"""
```

---

### 34. AI可以做数据分析吗？

#### 34.1 分析场景

```
1. 数据解读：Explain "这数据什么意思？"
2. 趋势分析：Describe "销售趋势是什么？"
3. 异常检测：Find "有什么异常数据？"
4. 建议：Suggest "如何提升转化率？"
```

#### 34.2 SQL生成

```python
# 自然语言转SQL
TEXT_TO_SQL_PROMPT = """你是数据分析师。请根据用户的问题生成SQL。

用户问题：{question}

请生成对应的SQL语句。只输出SQL，不要其他。
"""

# 使用
user_question = "各省份销售额Top10"
sql = generate_sql(user_question)
# SELECT province, SUM(sales) ...
```

#### 34.3 可视化建议

```python
# 图表建议
CHART_PROMPT = """根据以下数据，请推荐合适的可视化图表：

数据：
{data}

请给出：
1. 推荐的图表类型
2. X轴、Y轴的配置建议
"""
```

---

### 35. 如何实现多轮对话？

#### 35.1 对话历史管理

```python
# 简单实现
class DialogueManager:
    def __init__(self):
        self.messages = []

    def add_user_message(self, msg):
        self.messages.append({"role": "user", "content": msg})

    def add_ai_message(self, msg):
        self.messages.append({"role": "assistant", "content": msg})

    def get_context(self, max_turns=5):
        # 只取最近的对话
        return self.messages[-max_turns*2:]

    def chat(self, user_msg):
        self.add_user_message(user_msg)

        context = self.get_context()

        response = llm.chat(context)
        self.add_ai_message(response)

        return response
```

#### 35.2 带历史记录的RAG

```python
# 问题：之前说的是什么？
# 历史：用户问过"Redis的优点"
# 回答：提到过高并发、内存存储等

# 构建完整Prompt
prompt = f"""
之前的对话：
{history}

当前问题：{question}

请结合之前的对话回答。
"""

# 效果：可以引用之前的上下文
```

#### 35.3 回答模板

> 多轮对话的关键是记忆管理：
> 1. **短期记忆**：对话历史，保存最近N轮
> 2. **长期记忆**：用户画像、偏好设置，持久化
> 3. **总结记忆**：用LLM压缩历史，降低Token消耗
>
> 实际实现：messages数组 + sliding window。

---

## 第九章 AI安全合规（中高频 ★★★★）

### 36. 什么是AI幻觉？如何处理？

#### 36.1 幻觉定义

**LLM幻觉**：AI一本正经地胡说八道。

```
例子：
问：秦始皇是在哪个朝代？
AI：（编造）中，秦始皇是唐朝的皇帝
```

#### 36.2 解决方法

```python
# 方法1：RAG让AI基于事实
prompt = """根据以下知识回答，不要编造：

知识：{context}

问题：{question}
"""

# 方法2：置信度提示
prompt = """回答时：
1. 如果知道，请准确回答
2. 如果不确定，请说"我不确定"
3. 不要编造

问题：{question}
"""

# 方法3：验证输出
def verify_facts(answer, facts):
    for fact in facts:
        if fact not in 已知事实:
            return "存在幻觉"
    return "可信"
```

#### 36.3 回答模板

> AI幻觉��LLM的老大难问题：
>
> 1. **RAG**：只让AI基于给定的知识回答，不要自由发挥
> 2. **Prompt约束**：明确告诉AI不知道就是不知道，不要编造
> 3. **后验**：输出结果人工复核或二次验证
> 4. **引用来源**：让AI回答时带上参考
>
> 生产环境RAG+Prompt约束基本能覆盖80%幻觉问题。

---

### 37. 如何过滤敏感词？

#### 37.1 内容审核API

```python
# OpenAI Moderation
response = client.moderations.create(input=text)

if response.results[0].flagged:
    categories = response.results[0].categories
    print("违规类别:", categories)
    # 拒绝处理
```

#### 37.2 自定义词表

```python
# 敏感词过滤
forbidden_words = ["政治", "暴力", "色情", ...]

def filter_text(text):
    for word in forbidden_words:
        if word in text:
            return False
    return True

# 使用
if not filter_text(user_input):
    return "内容不合规，请换个问题"
```

#### 37.3 多层防护

```
第一层：输入预处理 → 拦截
第二层：Prompt审查 → 过滤
第三层：Moderation → 审核
第四层：输出抽检 → 复查
```

---

### 38. 数据隐私如何保护？

#### 38.1 隐私保护原则

```
1. 不上传敏感数据到公共API
2. 本地部署/私有化模型
3. 脱敏处理后再使用
4. 日志不留敏感信息
```

#### 38.2 脱敏处理

```python
import re

def desensitize(text):
    # 脱敏手机号
    text = re.sub(r'\d{11}', '138****0000', text)
    # 脱敏身份证
    text = re.sub(r'\d{17}[\dXx]', '110***********1234', text)
    # 脱敏银行卡
    text = re.sub(r'\d{16}', '**** **** **** 1234', text)
    return text
```

#### 38.3 私有化部署

```
方案：
1. 开源模型本地部署：Llama3/Qwen等
2. 私有API网关：OneAPI等
3. 模型蒸馏：保留核心能力，降低算力
```

---

## 第十章 架构设计（中高频 ★★★★）

### 39. AI网关如何设计？

#### 39.1 AI网关职责

```
AI网关 = 模型路由 + 负载均衡 + 限流 + 监控 + 安全

主要功能：
1. 统一接入：一个API对接多个模型
2. 智能路由：根据类型/成本选模型
3. 限流熔断：保护下游
4. 监控告警：性能/成本监控
5. 安全防护：Key管理/审计
```

#### 39.2 架构示例

```
                ┌──────────────┐
                │  AI Gateway │ ← 高可用
                └─────┬──────┘
           ┌────┴────┴────┐
           ↓             ↓
    ┌──────────┐  ┌──────────┐
    │  GPT-4   │  │ Claude3  │  ← 模型
    └──────────┘  └──────────┘
           ↑             ↑
           └──────┬──────┘
                  ↓
    API网关、计费、日志、监控
```

#### 39.3 OneAPI

```bash
# 一键部署AI网关
docker run -d -p 3000:3000 \
    -e OPENAI_API_KEY=sk-xxx \
    -e AUTH_TOKEN=your-token \
    justsong/one-api

# 调用示例
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-3.5-turbo", "messages": [...]}'
```

---

### 40. 如何控制AI成本？

#### 40.1 成本构成

```
AI成本 = Token费用 = 输入Token × 输入单价 + 输出Token × 输出单价

控制策略：
1. Prompt精简 → 减少输入Token
2. 输出截断 → 限制输出Token
3. 缓存命中 → 相同问题不重复调用
4. 模型选择 → 便宜模型用3.5
```

#### 40.2 成本优化技巧

```python
# 1. Prompt模板抽到配置
PROMPT_CACHE_KEY = "prompt:{hash}"
cached = redis.get(PROMPT_CACHE_KEY)
if cached:
    return cached

# 2. 输出截断
max_tokens = 500  # 限制输出

# 3. 参数控制
temperature = 0.5  # 不要太随机

# 4. 模型分级
def route_by_complexity(question):
    if is_simple_qa(question):
        return "gpt-3.5-turbo"  # 简单问题用便宜的
    else:
        return "gpt-4"  # 复杂问题用贵的
```

#### 40.3 回答模板

> AI成本主要在Token，控制成本的方法：
> 1. **缓存**：相同问题不重复调用API
> 2. **Prompt优化**：精简Prompt，减少输入Token
> 3. **模型分级**：简单问题用3.5，复杂问题用4
> 4. **缓存+分级组合使用**，成本可以降低60%+

---

### 41. 如何设计AI高可用架构？

#### 41.1 高可用要求

```
1. 多模型切换：主模型失败自动切换备选
2. 限流保护：防止突发流量打挂服务
3. 降级策略：高峰期降级非核心功能
4. 监控告警：及时发现问题
```

#### 41.2 容错设计

```python
class AIFaultTolerance:
    def __init__(self, primary_llm, fallback_llm):
        self.primary = primary_llm
        self.fallback = fallback_llm

    def chat_with_fallback(self, prompt):
        try:
            return self.primary.chat(prompt)
        except RateLimitError:
            # 触发限流，降级到备用
            return self.fallback.chat(prompt)
        except TimeoutError:
            # 超时降级
            return self.fallback.chat(prompt)
        except Exception as e:
            logger.error(f"AI服务异常: {e}")
            return self.fallback.chat(prompt)
```

#### 41.3 降级策略

| 级别 | 触发条件 | 动作 | 影响 |
|------|---------|------|------|
| 正常 | - | GPT-4 | 无 |
|黄色|GPT-4延迟>10s|GPT-3.5|部分受影响 |
|橙色| GPT-4失败 | 缓存+3.5 | 明显降级 |
|红色| 全量失败 |FAQ+固定回复 | 影响很大 |

---

## 附录：常见AI面试追问

1. **你用过哪些Embedding模型？效果对比如何？**
   - 回答：BGE-large-zh在中文场景效果最好

2. **RAG和Fine-tune怎么选？**
   - 回答：RAG成本低、实时性好；Fine-tune针对性强

3. **如何评估RAG效果？**
   - 回答：召回率、准确率、半主观评估

4. **GPU显存不够怎么处理？**
   - 回答：模型量化、batch切分、升级显

5. **LLM未来发展趋势？**
   - 回答：多模态、长context、Agent生态、端侧

---

## 参考资料

- OpenAI官方文档：https://platform.openai.com/docs/
- LangChain文档：https://python.langchain.com/
- Claude文档：https://docs.anthropic.com/
- RAG综述论文：2024年初至今

---

> 整理by Claude Code | Go后端AI应用开发面试高频题