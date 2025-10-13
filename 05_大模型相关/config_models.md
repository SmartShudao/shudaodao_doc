## 模型管理

### 0.配置文件 /config/base.yaml

```
1.生成模型
language_models:
  - model
    其他配置项
    
  - model

2.嵌入模型
embedding_models:
  - model
    其他配置项
    
  - model

3.重排序模型
reranker_models:
  - model
    其他配置项
    
  - model
```

### 1.生成模型「language_models」

```
language_models:
  - model: "Qwen3-0.6B"                     任意你想发布成的名字
    is_think: false                         如何模型支持（如Qwen3支持开关）
    proxy: "local"                          本地模型还是远程模型
    local:                                  本地模型配置
      engine: "transformers"                本地加载方式 transformers/vllm 
      path: "models/Qwen/Qwen3-0.6B"        模型路径（根目录的models下）
      kwargs:                               传递给模型启动的参数
        device_map: "cpu"                   模型本身支持的参数设置
        ...
      generate:                             generate 方法参数配置
        temperature: 0.2                    温度
        kwargs:                             enerate 方法本身支持的参数
          temperature: 0.2                  也可以写在这里，优先级低
    remote:                                 远程模型配置（通过WebAPI发布的）
      model: ""                             接口的模型名字
      url: "https://ip:port/api/..."        接口的地址
      key: "api-key"                        接口的密钥
      

```


### 2.嵌入模型「embedding_models」

```
embedding_models:
```

### 3.重排序模型「reranker_models」

```
reranker_models:
```