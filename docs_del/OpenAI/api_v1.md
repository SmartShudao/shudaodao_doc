以下是OpenAI官网提供的主要API接口分类及详细说明，基于最新搜索结果整理：

---

### **1. 对话类接口**

#### **Chat Completions API**

- **用途**：多轮对话交互，适用于聊天机器人、客服等场景。
- **请求地址**：`POST https://api.openai.com/v1/chat/completions`
- **参数**：
    - `model`（如 `gpt-3.5-turbo`、`gpt-4`）
    - `messages`（包含 `role` 和 `content` 的对话历史）
    - 可选参数：`temperature`、`max_tokens`、`stream` 等。
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "Hello!"}]}'
  ```

#### **Completions API**

- **用途**：单轮文本生成，适用于文章、代码补全等。
- **请求地址**：`POST https://api.openai.com/v1/completions`
- **参数**：
    - `model`（如 `text-davinci-003`）
    - `prompt`（输入文本）
    - 可选参数：`max_tokens`、`temperature`、`stop` 等。
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"model": "text-davinci-003", "prompt": "Say this is a test", "max_tokens": 7}'
  ```

---

### **2. 私有化模型训练类**

#### **Embeddings API**

- **用途**：将文本转换为向量表示，用于语义搜索、聚类等。
- **请求地址**：`POST https://api.openai.com/v1/embeddings`
- **参数**：
    - `model`（如 `text-embedding-ada-002`）
    - `input`（待向量化的文本）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/embeddings" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"input": "The food was delicious...", "model": "text-embedding-ada-002"}'
  ```

#### **Fine-tunes API**

- **用途**：微调GPT模型，适用于定制化任务。
- **请求地址**：
    - 创建微调任务：`POST https://api.openai.com/v1/fine-tunes`
    - 获取微调列表：`GET https://api.openai.com/v1/fine-tunes`
    - 取消微调：`POST https://api.openai.com/v1/fine-tunes/{fine_tune_id}/cancel`
- **参数**：
    - `training_file`（训练数据集文件ID）
    - `model`（如 `ada`、`babbage`、`curie`）
    - `n_epochs`（训练轮次）
    - `batch_size`（批次大小）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/fine-tunes" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"training_file": "file-XGinujblHPwGLSztz8cPS8XY"}'
  ```

---

### **3. 通用类接口**

#### **Models API**

- **用途**：获取可用模型列表或模型详情。
- **请求地址**：
    - 获取模型列表：`GET https://api.openai.com/v1/models`
    - 获取模型详情：`GET https://api.openai.com/v1/models/{model}`
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/models" \
    -H "Authorization: Bearer $OPENAI_API_KEY"
  ```

#### **Files API**

- **用途**：上传文件用于微调或分析。
- **请求地址**：`POST https://api.openai.com/v1/files`
- **参数**：
    - `file`（文件内容）
    - `purpose`（如 `fine-tune`）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/files" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -F purpose="fine-tune" \
    -F file="@mydata.jsonl"
  ```

---

### **4. 图片 & 音频类**

#### **Images API**

- **用途**：生成或编辑图片（如 DALL·E）。
- **请求地址**：
    - 生成图片：`POST https://api.openai.com/v1/images/generations`
    - 编辑图片：`POST https://api.openai.com/v1/images/edits`
- **参数**：
    - `prompt`（图片描述）
    - `n`（生成数量）
    - `size`（图片尺寸）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/images/generations" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"prompt": "A cute cat", "n": 1, "size": "1024x1024"}'
  ```

#### **Audio API**

- **用途**：语音转文本（如 Whisper）。
- **请求地址**：
    - 语音转文本：`POST https://api.openai.com/v1/audio/transcriptions`
- **参数**：
    - `file`（音频文件）
    - `model`（如 `whisper-1`）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/audio/transcriptions" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -F file="@audio.mp3" \
    -F model="whisper-1"
  ```

---

### **5. 最新接口（2025年更新）**

#### **Responses API**

- **用途**：结合 Chat Completions 和 Assistants API，支持多工具调用（如网页搜索、文件检索）。
- **请求地址**：`POST https://api.openai.com/v1/responses`
- **功能**：
    - 内置工具（Web Search、File Search、Computer Use）
    - 流式事件处理
    - 支持 Webhook 回调
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/responses" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"model": "gpt-4o", "query": "Find the latest AI news", "tools": ["web_search"]}'
  ```

#### **o1 API**

- **用途**：增强版模型 API，支持函数调用、结构化输出、视觉处理。
- **请求地址**：`POST https://api.openai.com/v1/o1/completions`
- **功能**：
    - 自动调用外部 API
    - JSON 结构化返回
    - 视觉分析（如表格错误检测）
- **示例**：
  ```bash
  curl "https://api.openai.com/v1/o1/completions" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{"model": "o1", "prompt": "Analyze this form for errors", "image": "base64_encoded_image"}'
  ```

---

### **总结**

OpenAI 的 API 主要分为：

1. **对话类**（Chat、Completions）
2. **私有化训练类**（Embeddings、Fine-tunes）
3. **通用类**（Models、Files）
4. **多媒体类**（Images、Audio）
5. **最新增强类**（Responses、o1 API）

更详细的文档可参考 [OpenAI 官方 API 文档](https://platform.openai.com/docs/api-reference)。