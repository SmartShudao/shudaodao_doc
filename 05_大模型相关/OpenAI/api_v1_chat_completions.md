# OpenAI Chat API 接口规划

## 1. 概述

OpenAI 的 Chat API 允许开发者通过 HTTP 请求与 OpenAI 的语言模型进行交互，获取智能回复。以下是详细的接口规划说明。

## 2. 基础信息

- **API 端点**: `https://api.openai.com/v1/chat/completions`
- **请求方法**: POST
- **认证方式**: Bearer Token (API Key)
- **支持的模型**: gpt-4, gpt-4-turbo, gpt-3.5-turbo 等

## 3. 请求接口规范

### 请求头 (Headers)

| 字段                    | 类型     | 必填 | 描述                                  |
|-----------------------|--------|----|-------------------------------------|
| `Authorization`       | string | 是  | Bearer 加 API Key，如: `Bearer sk-...` |
| `Content-Type`        | string | 是  | 固定为 `application/json`              |
| `OpenAI-Organization` | string | 否  | 组织ID (企业账户使用)                       |

### 请求体 (JSON Body)

```json
{
  "model": "gpt-3.5-turbo",
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
  "temperature": 0.7,
  "top_p": 1,
  "n": 1,
  "stream": false,
  "stop": null,
  "max_tokens": 1000,
  "presence_penalty": 0,
  "frequency_penalty": 0,
  "logit_bias": null,
  "user": "user123"
}
```

#### 参数说明

| 参数                  | 类型           | 必填 | 描述                  |
|---------------------|--------------|----|---------------------|
| `model`             | string       | 是  | 使用的模型ID             |
| `messages`          | array        | 是  | 消息对象数组，描述对话历史       |
| `temperature`       | number       | 否  | 采样温度 (0-2)，值越高输出越随机 |
| `top_p`             | number       | 否  | 核心采样概率 (0-1)        |
| `n`                 | integer      | 否  | 生成多少个回复选项           |
| `stream`            | boolean      | 否  | 是否流式返回结果            |
| `stop`              | string/array | 否  | 停止生成的标记             |
| `max_tokens`        | integer      | 否  | 生成的最大token数         |
| `presence_penalty`  | number       | 否  | 主题重复惩罚 (-2.0到2.0)   |
| `frequency_penalty` | number       | 否  | 单词重复惩罚 (-2.0到2.0)   |
| `logit_bias`        | object       | 否  | 特定token的偏差调整        |
| `user`              | string       | 否  | 用户标识，用于滥用检测         |

### Messages 格式

每个消息对象包含:

- `role`: "system"、"user" 或 "assistant"
- `content`: 消息内容
- `name` (可选): 参与者名称

## 4. 响应接口规范

### 成功响应 (HTTP 200)

```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-3.5-turbo",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！有什么我可以帮助你的吗？"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
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
| `usage`   | object  | token使用统计                |

### Choices 对象

| 字段              | 类型      | 描述                                        |
|-----------------|---------|-------------------------------------------|
| `index`         | integer | 选项索引                                      |
| `message`       | object  | 生成的消息                                     |
| `finish_reason` | string  | 停止原因 ("stop", "length", "content_filter") |

### 流式响应

当 `stream: true` 时，响应为多个 SSE (Server-Sent Events) 数据块:

```
data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-3.5-turbo","choices":[{"index":0,"delta":{"content":"Hello"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-3.5-turbo","choices":[{"index":0,"delta":{"content":"!"},"finish_reason":null}]}

data: [DONE]
```

## 5. 错误响应

### HTTP 状态码

| 状态码 | 描述              |
|-----|-----------------|
| 400 | 错误请求 - 参数无效     |
| 401 | 未授权 - API Key无效 |
| 429 | 请求过多 - 达到速率限制   |
| 500 | 服务器内部错误         |

### 错误响应体

```json
{
  "error": {
    "message": "Invalid API Key",
    "type": "invalid_request_error",
    "param": null,
    "code": "invalid_api_key"
  }
}
```

## 6. 使用示例

### cURL 示例

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "解释一下量子计算"}],
    "temperature": 0.7
  }'
```

### Python 示例

```
import openai

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你是一个有帮助的助手。"},
        {"role": "user", "content": "解释一下量子计算"}
    ]
)

print(response.choices[0].message.content)
```

## 7. 最佳实践

1. 始终从系统消息开始对话，设置助手行为
2. 合理设置 temperature 值平衡创造性和一致性
3. 监控 usage 字段控制成本
4. 对于长对话，注意 token 限制并及时截断
5. 使用 user 字段标识不同用户以改进滥用检测

## 8. 限制

- 最大 token 限制取决于模型 (如 gpt-3.5-turbo 通常为 4096 tokens)
- 请求速率限制取决于账户类型
- 内容过滤可能阻止某些请求或修改响应