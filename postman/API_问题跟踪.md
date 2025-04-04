# API问题跟踪

本文档用于跟踪在API测试过程中发现的问题和异常行为。

## 1. 高优先级问题

### 1.1 用户角色更新不生效

**问题描述**:
- 接口调用返回200成功，但角色实际未更新
- 更新操作: PUT `/user/:id` 包含角色字段

**重现步骤**:
1. 执行"管理员-更新用户角色"请求，尝试将用户角色更改为"admin"
2. 执行"验证角色更新"请求，确认实际角色
3. 观察到角色值仍为原始值（通常是"user"）

**技术日志**:
```
[GIN] 2025/04/01 - 12:15:38 | 200 | 4.176833ms | ::1 | PUT "/user/106"
```

**请求数据**:
```json
{
  "nickname": "更新的用户名",
  "role": "admin",
  "email": "updated@example.com"
}
```

**响应数据**:
```json
{
  "message": "更新用户成功"
}
```

**验证请求**:
```
GET {{base_url}}/user/106
```

**验证响应**:
```json
{
  "user": {
    "CreatedAt": "2025-03-30T16:27:35.596+08:00",
    "UpdatedAt": "2025-04-01T12:15:38.807+08:00",
    "DeletedAt": null,
    "ID": 106,
    "Email": "updated@example.com",
    "Role": "user"
  }
}
```

**影响**:
- 使用API无法提升用户权限或更改角色
- 管理功能受限

**可能解决方案**:
1. 与后端团队确认是否故意禁用角色更新
2. 检查后端代码中对角色字段的处理
3. 考虑创建专门的角色管理API

**当前状态**: 待解决

### 1.2 用户更新邮箱时可能引发500错误

**问题描述**:
- 更新用户信息时，如果邮箱地址已被其他用户使用，会导致服务器返回500错误
- 服务器未提供适当的错误处理，而是直接返回内部服务器错误

**重现步骤**:
1. 使用管理员权限更新用户信息
2. 将邮箱地址设置为系统中已存在的邮箱
3. 发送请求并观察500错误响应

**技术日志**:
```
2025/04/01 12:33:33 /Users/hxc/Desktop/zonghe1/Stage5-wenda/services/user_server.go:82 Error 1062 (23000): Duplicate entry 'huangxiaochuan2020@163.com' for key 'users.uni_users_email'
[4.031ms] [rows:0] UPDATE `users` SET `email`='huangxiaochuan2020@163.com',`updated_at`='2025-04-01 12:33:33.214' WHERE id = 106
[GIN] 2025/04/01 - 12:33:33 | 500 | 5.267584ms | ::1 | PUT "/user/106"
```

**请求数据**:
```json
{
  "nickname": "更新的用户名",
  "email": "huangxiaochuan2020@163.com"
}
```

**影响**:
- 用户界面可能会显示通用错误消息，无法提供有意义的反馈
- 管理员无法理解为什么更新操作失败
- 可能导致用户混淆和支持请求增加

**可能解决方案**:
1. 增强后端错误处理，对唯一约束冲突返回更具体的400错误
2. 在前端实现邮箱唯一性验证，在发送请求前验证
3. 更新API文档，说明邮箱地址必须是唯一的

**当前状态**: 待解决

## 2. 中等优先级问题

// 此处可以添加其他中等优先级问题

## 3. 低优先级问题

// 此处可以添加其他低优先级问题
