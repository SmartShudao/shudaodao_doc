### 列出可用模型清单

```
方式 : get
地址 : api/v1/models
```

#### 输入

    无

#### 输出

```Json
{
  "models": [
    {
      "id": "模型名字",
      "is_think": true
    }
  ]
}
```

数据来自配置文件 /config/base.yaml

| 属性       | 说明                   | 值类型    | 数据来源                                 |
|:---------|:---------------------|:-------|:-------------------------------------|
| id       | 模型名称                 | string | language_models.model 如：'Qwen3-0.6B' |
| is_think | 思考模式，Qwen3可关闭，其他隐藏输出 | bool   | language_models.is_think             |
