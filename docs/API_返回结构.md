
## 格式
```text
{
  "success": "true/false"
  "code": 200/400/500,
  "message": "提示消息",
  "data": "any",
  "error": {
    "type": "错误类型",
    "details": "any"
  }
}
```

## 成功  - 示例
```json
{
    "success": true,
    "code": 200,
    "name": "auth_login",
    "message": "登录成功",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJsZl9zaGFvIiwiaWQiOiI5MDgwMjI0NjYxNDI1NzY2NCIsImV4cCI6MTc1ODYyMTY4OX0.CRg4fcSHliNcucd0hojTogIszIOS29SAl2cvoZ_iiaY"
    }
}
```

```json
{
    "success": true,
    "code": 200,
    "name": "AuthUserResponse",
    "message": "操作成功",
    "data": {
        "auth_user_id": 90802246614257664,
        "username": "lf_shao",
        "email": "lf_shao@163.com",
        "is_active": true
    }
}
```

```json
{
    "success": true,
    "code": 200,
    "name": "Paging",
    "message": "查询成功",
    "data": {
        "total": 1,
        "page": 1,
        "size": 12,
        "pages": 1,
        "rows": [
            {
                "staff_id": "96084030667100160",
                "staff_no": null,
                "staff_name": "名字1",
                "staff_spell": null,
                "staff_sex": null,
                "staff_birthday": "2025-09-23",
                "staff_mobile": null,
                "staff_email": null,
                "staff_address": null,
                "staff_status": null,
                "staff_entrydate": "2025-09-23",
                "staff_leavedate": "2025-09-23",
                "create_person": null,
                "create_date": "2025-09-23T11:23:38.066683",
                "modify_person": null,
                "modify_date": "2025-09-23T11:23:38.066684",
                "remark": null,
                "tenant_id": "None"
            }
        ]
    }
}
```

## 错误 - 示例

```json
{
    "success": false,
    "code": 403,
    "name": "PermissionError (http://localhost:8000/api/v1/shudao_acm/t/staff)",
    "message": "非法访问，没有权限授权",
    "error": "拒绝 -> lf_shao - shudao_acm.t_staff - edit"
}
```

```json
{
    "success": false,
    "code": 422,
    "name": "RequestValidationError (http://localhost:8000/api/v1/shudao_acm/t/staff)",
    "message": "验证失败",
    "error": [
        {
            "field": "staff_entrydate",
            "message": "输入无效"
        }
    ]
}
```

```json
{
    "success": false,
    "code": 405,
    "name": "HTTPException (http://localhost:8000/api/v1/shudao_acm/t/staff1)",
    "message": "方法不允许",
    "error": "405: Method Not Allowed"
}
```


