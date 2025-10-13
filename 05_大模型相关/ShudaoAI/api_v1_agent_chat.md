## ShuDaodao 对话智能体 接口

主要通过 **agent/chat API** 实现

- 适用于智能体的问答或文本生成任务。
- 适用于公司产品化，方便Agent的调用

### **1. 接口基本信息**

- **API地址**:
    - /api/v1/agent/chat
- **请求方法**:
    - POST
- **认证方式**:
    - 自定义
- **适用模型**:
    - 所有开源语言模型「language model」

### **2. 请求参数（JSON Body）**

从标准版接口删减了暂时用不上的参数

**增加了1个核心参数**

    **enable_thinking**:用于关闭Qwen3这类可关闭思考模式的模型

**强化了 user 的价值**

    强化了user的用途，系统的核心关联参数

```json
{
  "id": "对话上下文标识",
  "model": "Qwen3-0.6B/DeepSeek",
  "messages": [
    {
      "role": "system",
      "content": "你是一个有帮助的助手。"
    },
    {
      "role": "user",
      "content": "你好！"
    }
  ],
  "user": "user123",
  "stream": false,
  "enable_thinking": true,
  "top_p": 0.5,
  "temperature": 0.5,
  "max_tokens": 512
}

```

```python
from typing import Optional, List
from pydantic import BaseModel, Field
from shudaodao_omni.schema.language_request import LanguageModelMessage


class GenerateRequest(BaseModel):
  """ 生成的请求 """
  id: str = Field("", description="持久上下文ID")
  model: str = Field(..., description="使用的模型名称")  # 如 Qwen3-32B 等
  messages: List[LanguageModelMessage] = Field(..., description="对话消息列表")
  user: Optional[str] = Field(..., description="用户标识")
  stream: Optional[bool] = Field(False, description="是否流式输出")
  enable_thinking: bool = Field(False, description="是否深度思考")
  temperature: Optional[float] = Field(0.3, ge=0, le=2, description="采样温度，0-2之间")
  top_p: Optional[float] = Field(1, ge=0, le=1, description="核心采样概率")
  max_tokens: Optional[int] = Field(None, ge=1, description="最大token数")

  @property
  def kwargs(self) -> dict:
    _kwargs = dict()
    _kwargs["enable_thinking"] = self.enable_thinking
    _kwargs["temperature"] = self.temperature
    _kwargs["top_p"] = self.top_p
    _kwargs["max_tokens"] = self.max_tokens
    return _kwargs
```

### **3. 响应示例**

### 成功响应 (HTTP 200)

```json
{
  "id": "对话上下文标识",
  "object": "generate",
  "created": 1677652288,
  "model": "Qwen3-0.6B/DeepSeek",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！有什么我可以帮助你的吗？",
        "reasoning_content": "推理思考的内容"
      },
      "finish_reason": "stop"
    }
  ]
}
```

#### 响应字段说明

| 字段        | 类型      | 描述                       |
|-----------|---------|--------------------------|
| `id`      | string  | 本次对话的唯一ID                |
| `object`  | string  | 对象类型 ("chat.completion") |
| `created` | integer | 创建时间戳                    |
| `model`   | string  | 使用的模型                    |
| `choices` | array   | 生成的回复选项                  |

### Choices 对象

| 字段              | 类型      | 描述                                        |
|-----------------|---------|-------------------------------------------|
| `index`         | integer | 选项索引                                      |
| `message`       | object  | 生成的消息                                     |
| `finish_reason` | string  | 停止原因 ("stop", "length", "content_filter") |

### 流式响应

当 `stream: true` 时，响应为多个 SSE (Server-Sent Events) 数据块:

```
data: {
  "id":"对话上下文标识",
  "object":"generate.chunk",
  "created":1694268190,
  "model": "Qwen3-0.6B/DeepSeek",
  "choices":[
    {
      "index":0,
      "delta":
        {
          "role":"thinking|answer",
          "content":"Hello"
        },
      "finish_reason":null
  }]
}

data: [DONE]
```

**新增节点:**

    data --> choices[] --> delta --> role

| 字段值      | 描述          |
|----------|-------------|
| thinking | 输出内容 — 思考部分 | 
| answer   | 输出内容 — 答案部分 |

### **4. Python 调用示例**

``` python
@chats_router.post("/generate")
async def generate(generate_request: GenerateRequest):
    if generate_request.stream:
        return StreamingResponse(
            ContextManager.create().generate_stream(generate_request), media_type="application/json")
    return StreamingResponse(
        ContextManager.create().generate(generate_request).model_dump_json(), media_type="application/json")
```
