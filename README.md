
# 数道智融应用产品 说明

## 注意: shudaodao 版本
> **v0.1.58**

### v0.1.8 关键变化
- 1.增加了 shudaodao_acm
  - 表结构初步定型
- 2.修正 代码生成器 bug 
  - 过滤了 数据库的 pid 本表指向 id 的约束，它不再会出现在SQLModel类的约束中
- 3.清理了 shudaodao_web 工程
  - 引入了 Art Design Pro
  - 执行了 pnpm clean:dev

## 一、开发环境配置

### 使用 uv 更高效 
```text
1. 初始化 
uv sync

2. 安装 shudaodao 
pip install shudaodao
- 有 .c 文件，需要本地编译，如果编译失败，大概是本地缺少C++编译器，请自行安装

3. 更新 shudaodao
pip install --upgrade --index-url https://pypi.org/simple/ shudaodao
```

### pyproject.toml
```toml
dependencies = [
    "uvicorn>=0.35.0",
]
```
- 原则上只需要 uvicorn + pip install shudaodao
- 多余部分其实是 shudaodao 的依赖
- 因为有时 要从 pypi.org 强制更新，避免依赖包从pypi下载，速度极慢


## 二、版本更新

### v0.1.7 关键变化
- 1.支持 refresh_token
  - 可通过刷新接口 依据 refresh_token 无需登录获取 access_token
- 2.修正 代码生成器 bug 
    - 字段（field）同时指定了 default 和 default_factory 参数
- 3.修正 Application 启动逻辑
    - 过早的加载了各种Engine
- 4.修正 Swagger API 文档的资源本地化

### v0.1.6 关键变化

1.代码生成器,生成的FastAPI路由代码更简洁
```text
# 为指定的数据模型自动生成标准 CRUD+Q（查询）接口
# 以下是完整的代码

TEnumSchema_Router = AuthController.create(
    # prefix=f"/v1/{get_schema_name()}/t_enum_schema",
    prefix="/v1/enums/schema",
    db_config_name=get_engine_name(),  # 配置文件中的数据库连接名称
    schema_name=get_schema_name(),
    table_name="t_enum_schema",
    # 模型设置: 数据库模型、创建模型、更新模型、响应模型
    model_class=TEnumSchema,
    create_schema=TEnumSchemaCreate,
    update_schema=TEnumSchemaUpdate,
    response_schema=TEnumSchemaResponse,
)
```
2.增加主要对象类的Google风格文档注释

### v0.1.5 关键变化

1.支持多租户

2.query 接口更新 

    新的参数 format : list page tree

    支持 1对多 数据库表的 二级目录查询

```json
{
    "fields": [
        "group_id as id",
        "group_name as name"
    ],
    "type": "AND",
    "conditions": [],
    "orderby": [],
    "format": "tree",
    "tree": {
        "field_tag": "Group",
        "field_id": "group_id",
        "field_pid": "group_pid",
        "field_children": "enum_fields",
        "children_tag": "Field",
        "children_fields": [
            "field_id as id",
            "field_label as name"
        ]
    }
}
```

### v0.1.5 关键变化

1.开发环境变化
- 增加组件 "pyjwt>=2.10.1", 用户产品使用授权
- 优化了代码生成器
- 更新的模版文件

2.后端API生成器
- 增加了SQLModel 约束关系的生成
- 生成 schema_name 目录 __init__.py 文件中多了配置管理
- 优化 雪花算法ID，后端bigint ,前端 str
- 优化 日期类字段的支持，前后端保持一致（但没有采用有时区的UTC，出于老库兼容考虑）
  
3.增加了产品使用授权功能
  config 目录 增加了2个文件

    1、许可文件：license.jwt
    2、公钥文件：public.pem

  目前是开发许可，**60天** 过期，定期更新

## v0.1.1 关键变化
  > FastAPI 的路由，支持配置文件加载 
```yaml
  #路由配置
  routers:
    # name 是包名「可以直接写子包，但是要写全」，注意: 不是目录名
    - { name: "shudaodao_core", "prefix": "/api" }
    - { name: "shudaodao_common", "prefix": "/api", "tags": [ ] }
    - { name: "shudaodao_omni", "prefix": "/api", "tags": [ "AI 应用" ] }
    - { name: "shudaodao_generate.shudao_acm", "prefix": "/api" , "tags": [ "访问控制系统" ] }

```

## 注意事项

### 1.资源 根目录
- 目录 web 标记为  **资源根目录**

### 2.源代码 根目录
- 所有 src 目录 需要标记为 **源代码根目录**
  - src
  - packages/****/src

### 3.默认目录
- 配置都在目录 **config** 
- 模型文件目录 **models**

