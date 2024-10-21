---
title: 如何优雅的设计RESTful API
tags: [Java,RESTful API]
date: 2024-10-21
categories: 后端
---

```
思考几个问题​
1. 为什么要有规范？​
2. 如何让前后端快速开发、联调？
```

## 一些概念​
### URI和URL的区别​
### http状态码的含义​
### Http method的含义​

## 什么是RESTful​
### 定义​
### 设计原则​
#### URI设计​
#### 状态码的使用​
#### 请求和响应体的设计​
#### 异常处理​
#### 安全性的考虑​
#### 文档​
#### 超媒体驱动的 API（HATEOAS）​

## 场景举例​
### 查询用户列表（带条件）​
### 获取用户​
### 删除用户​
### 修改用户状态​
### 批量修改某一个用户的属性​
### 查询用户关联的机器狗​

## 附录​
### 优秀案例​
1. GitHub​
    - [https://docs.github.com/zh/rest​](https://docs.github.com/zh/rest​)
    - [https://api.github.com/​](https://api.github.com/​)
2. Swagger​

### 工具​
1. Spring HATEOAS：[点我](https://spring.io/projects/spring-hateoas)

### 示例代码