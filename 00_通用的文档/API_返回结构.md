
## 格式
```text
{
  "code": 200/400/500,
  "msg": "提示消息",
  "data": "any",
  "error": "any"
}
```

## 成功  
```json
{
  "code": 200,
  "msg": "登录成功",
  "data": {
    "key1": "any",
    "key2": {
      "children": "any"
    },
    "key3": [
      "any_value1",
      "any_value2"
    ]
  }
}
```

```json
{
    "code": 200,
    "msg": "操作成功",
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
    "code": 200,
    "msg": "查询成功",
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

## 错误 

```json
{
    "code": 403,
    "msg": "非法访问，没有权限授权",
    "error": "拒绝 -> lf_shao - shudao_acm.t_staff - edit"
}
```

```json
{
    "code": 422,
    "msg": "验证失败",
    "error": [
        {
            "field": "staff_entry_date",
            "message": "输入无效"
        }
    ]
}
```

```json
{
    "code": 405,
    "message": "方法不允许",
    "error": "405: Method Not Allowed"
}
```


