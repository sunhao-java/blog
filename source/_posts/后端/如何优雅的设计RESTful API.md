---
title: 如何优雅的设计RESTful API
tags: [Java,RESTful API]
date: 2024-10-21
categories: 后端
---

> 在现代 Web 开发中，RESTful API 已经成为前后端交互的标准方式。一个设计良好的 API 不仅能提高开发效率，还能降低维护成本。本文将深入探讨如何优雅地设计 RESTful API。

```
思考几个问题​
1. 为什么要有规范？​
2. 如何让前后端快速开发、联调？
```

<!-- more -->

## 思考
### 1. 为什么要有规范？

- **提高开发效率**：统一的规范让开发者能够快速理解和使用 API
- **降低沟通成本**：减少前后端联调时的误解和返工
- **便于维护和扩展**：良好的设计使系统更容易维护和升级
- **提升用户体验**：一致的接口设计让客户端开发更加顺畅

### 2. 如何让前后端快速开发、联调？

- 制定清晰的 API 规范文档
- 使用标准化的状态码和错误处理
- 提供完整的示例和测试环境
- 建立高效的沟通机制

## 一些概念​
### URI和URL的区别​
- **URI (Uniform Resource Identifier)**：统一资源标识符，用于标识某一互联网资源
- **URL (Uniform Resource Locator)**：统一资源定位符，是 URI 的子集，不仅标识资源，还能定位资源
```text
URI: /users/123
URL: https://api.example.com/users/123
```

### http状态码的含义​

| 状态码 | 类别 | 含义 |
|--------|------|------|
| 2xx | 成功 | 请求成功处理 |
| 3xx | 重定向 | 需要后续操作 |
| 4xx | 客户端错误 | 请求有误 |
| 5xx | 服务器错误 | 服务器处理出错 |

- `200 OK`：请求成功
- `201 Created`：资源创建成功
- `400 Bad Request`：请求参数错误
- `401 Unauthorized`：未授权
- `403 Forbidden`：禁止访问
- `404 Not Found`：资源不存在
- `500 Internal Server Error`：服务器内部错误

### Http method的含义​

| 方法 | 用途 | 幂等性 | 安全性 |
|------|------|--------|--------|
| GET | 获取资源 | 是 | 是 |
| POST | 创建资源 | 否 | 否 |
| PUT | 更新资源 | 是 | 否 |
| PATCH | 部分更新 | 否 | 否 |
| DELETE | 删除资源 | 是 | 否 |

## 什么是RESTful​
### 定义​

REST (Representational State Transfer) 是一种软件架构风格，RESTful API 遵循以下约束：

1. **客户端-服务器架构**：分离关注点
2. **无状态**：每个请求包含完整信息
3. **可缓存**：响应可以被缓存
4. **统一接口**：一致的接口设计
5. **分层系统**：系统可以分层部署
6. **按需代码**（可选）：支持扩展功能

### 设计原则​
#### URI设计​

**基本原则：**
- 使用名词而非动词
- 使用复数形式
- 使用小写字母和连字符
- 层级结构清晰

**优秀示例：**
```
✅ 好的设计
GET /api/v1/users
GET /api/v1/users/123
POST /api/v1/users
PUT /api/v1/users/123
DELETE /api/v1/users/123

❌ 不好的设计
GET /api/v1/getUsers
GET /api/v1/UserManagement/getUser?id=123
POST /api/v1/createUser
```

**版本控制：**
```
/api/v1/users        # 版本1
/api/v2/users        # 版本2
```

**查询参数：**
```
GET /api/v1/users?page=1&size=10&sort=name
GET /api/v1/users?status=active&role=admin
```

#### 状态码的使用​

**成功状态码：**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "id": 123,
    "name": "张三"
  }
}
```

```http
HTTP/1.1 201 Created
Location: /api/v1/users/123
Content-Type: application/json

{
  "data": {
    "id": 123,
    "name": "张三"
  }
}
```

**错误状态码：**
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "参数验证失败",
    "details": [
      {
        "field": "email",
        "reason": "邮箱格式不正确"
      }
    ]
  }
}
```

#### 请求和响应体的设计​

**统一响应格式：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "users": [
      {
        "id": 1,
        "name": "张三",
        "email": "zhangsan@example.com"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 10,
      "total": 100
    }
  }
}
```

**错误响应格式：**
```json
{
  "code": 400,
  "message": "参数验证失败",
  "error": {
    "type": "VALIDATION_ERROR",
    "details": [
      {
        "field": "email",
        "message": "邮箱格式不正确"
      }
    ]
  }
}
```

#### 异常处理​

**全局异常处理：**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException e) {
        ErrorResponse error = ErrorResponse.builder()
            .code(400)
            .message("参数验证失败")
            .error(ErrorDetails.builder()
                .type("VALIDATION_ERROR")
                .details(e.getDetails())
                .build())
            .build();
        return ResponseEntity.badRequest().body(error);
    }
}
```

#### 安全性的考虑​

**认证和授权：**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**输入验证：**
```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
    // 参数自动验证
    User user = userService.create(request);
    return ResponseEntity.created(URI.create("/users/" + user.getId())).body(user);
}
```

**防止注入攻击：**
- 使用参数化查询
- 输入过滤和转义
- 限制请求频率

#### 文档​

**API 文档应该包含：**
- 接口地址和方法
- 请求参数说明
- 响应格式示例
- 错误码说明
- 调用示例

**推荐工具：**
- Swagger/OpenAPI
- Postman
- Apiary

#### 超媒体驱动的 API（HATEOAS）​

**响应中包含相关链接：**
```json
{
  "data": {
    "id": 123,
    "name": "张三",
    "links": {
      "self": "/api/v1/users/123",
      "update": "/api/v1/users/123",
      "delete": "/api/v1/users/123",
      "orders": "/api/v1/users/123/orders"
    }
  }
}
```

## 场景举例​
### 查询用户列表（带条件）​

```http
GET /api/v1/users?status=active&role=admin&page=1&size=20&sort=created_at,desc
```

**响应：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "users": [
      {
        "id": 1,
        "name": "张三",
        "email": "zhangsan@example.com",
        "status": "active",
        "role": "admin",
        "created_at": "2024-01-01T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 150,
      "total_pages": 8
    }
  }
}
```

### 获取用户​

```http
GET /api/v1/users/123
```

**响应：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "user": {
      "id": 123,
      "name": "张三",
      "email": "zhangsan@example.com",
      "profile": {
        "avatar": "https://cdn.example.com/avatar/123.jpg",
        "bio": "软件工程师"
      },
      "links": {
        "self": "/api/v1/users/123",
        "orders": "/api/v1/users/123/orders"
      }
    }
  }
}
```

### 删除用户​

```http
DELETE /api/v1/users/123
```

**响应：**
```http
HTTP/1.1 204 No Content
```

### 修改用户状态​

```http
PATCH /api/v1/users/123/status
Content-Type: application/json

{
  "status": "inactive"
}
```

**响应：**
```json
{
  "code": 200,
  "message": "用户状态更新成功",
  "data": {
    "user": {
      "id": 123,
      "name": "张三",
      "status": "inactive"
    }
  }
}
```

### 批量修改某一个用户的属性​

```http
PATCH /api/v1/users/123
Content-Type: application/json

{
  "name": "李四",
  "email": "lisi@example.com",
  "profile": {
    "bio": "高级软件工程师"
  }
}
```

**响应：**
```json
{
  "code": 200,
  "message": "用户信息更新成功",
  "data": {
    "user": {
      "id": 123,
      "name": "李四",
      "email": "lisi@example.com",
      "profile": {
        "bio": "高级软件工程师"
      }
    }
  }
}
```

### 查询用户关联的机器狗​

```http
GET /api/v1/users/123/robots?page=1&size=10
```

**响应：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "robots": [
      {
        "id": 1,
        "name": "机器狗一号",
        "model": "Model-X",
        "status": "active"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 10,
      "total": 3
    }
  }
}
```

## 附录​
### 优秀案例​

#### GitHub
- 文档地址：[https://docs.github.com/zh/rest](https://docs.github.com/zh/rest)
- API 地址：[https://api.github.com/](https://api.github.com/)

GitHub 的 API 设计非常规范，是学习 RESTful API 设计的优秀范例。

#### Swagger
Swagger 提供了完整的 API 设计和文档解决方案，广泛应用于现代 Web 开发。

### 工具​

1. Spring HATEOAS 是 Spring 生态中实现 HATEOAS 规范的框架，可以帮助开发者轻松构建超媒体驱动的 RESTful API。
2. Spring HATEOAS：[点我](https://spring.io/projects/spring-hateoas)

### 示例代码

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @GetMapping
    public ResponseEntity<PagedResponse<User>> getUsers(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String status) {
        
        Page<User> users = userService.findUsers(page, size, status);
        return ResponseEntity.ok(PagedResponse.of(users));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(UserResponse.of(user));
    }
    
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.create(request);
        URI location = URI.create("/api/v1/users/" + user.getId());
        return ResponseEntity.created(location).body(UserResponse.of(user));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> updateUser(
            @PathVariable Long id, 
            @Valid @RequestBody UpdateUserRequest request) {
        User user = userService.update(id, request);
        return ResponseEntity.ok(UserResponse.of(user));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

通过遵循这些设计原则和最佳实践，我们可以构建出优雅、易用、可维护的 RESTful API，为前后端协作提供良好的基础。