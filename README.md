# Luclin REST API Guidelines

使用该方案实现的接口所使用的接口协议可称为`Luclin Restful Protocol`，简称`LRP`

## Uri Defining

### 基本结构

```shell
scheme://domain/service/version/endpoint
```

- `scheme` 一般为`https`
- `domain` 业务访问域名
- `service` 服务名，用于网关路由
- `version` 服务版本，同样用于网关路由或灰度策略
- `endpoint` 业务路径，可能包括多条`/`

### 命名规范

在`Uri`命名中，仅使用全小写加上`-`连接符的形式，同时保持命名精简，禁止重复

例如：

```shell
https://portal.cthun.com/passport/v1.0/user/1/oauth-token
```

### 常见范例

取单个资源

```shell
GET https://portal.cthun.com/passport/v1.0/user/1
```

多个资源

```shell
GET https://portal.cthun.com/passport/v1.0/users
```

## Special Named

通过特殊命名机制，使得基于`LRP`的实现可以摆脱返回结果必包一层`data`，`code`之类的一级域的问题，让交互信息变得精简明晰

### 约定字 contract `_*`

用于通讯数据体中传递标准数据之外的通用约定信息

例如在上行中可以用`_token`域在`header`之外传递token，可以在下行中使用`_error`来传递错误信息

### 修饰器 decorator `@*`

在通讯数据体中用于对已经有的域补充额外的信息时用到，比如回传的域为`user`，则可以在同级的`@user`域名补充其数据结构`scheme`（如果使用`compact`压缩格式的话），提供分页信息`paginate`等

### 操作符 operator `$*`

操作符用于在endpoint中表达非ID类节点，例如：

```shell
# 修改用户1
PATCH https://portal.cthun.com/passport/v1.0/user/{user}

# 变更用户token
PATCH https://portal.cthun.com/passport/v1.0/user/$token
```

> 当然，合理的`endpoint`规划能避免使用操作符，但这仍然是一种有效的逃跑方案

## Request

### Allow Methods

- POST `create` or `trigger`（后者在定义时应尽量避免）
- PATCH `upsert` or `update`
- DELETE `remove`
- GET `fetch`

## Response

### 根域优先

将所有数据直接放在数据体根下的子域中

### 列表优先

凡是作为数据体的域，无论返回1个还是多个，一律放在一个列表中以`array`类型表达（即使只有1个单元）

## Deprecated Features

### 显式Endpoint

Endpoint不再需要显式以`.`结尾

```shell
# Before
https://domain/service/version/endpoint.

# Now
https://domain/service/version/endpoint
```

### Path参数后置

```shell
# Before
https://domain/service/version/resource/sub-resource./{resourceId}/{subResourceId}

# Now
https://domain/service/version/resource/{resourceId}/sub-resource/{subResourceId}
```

现在，除了最后一个Path参数外的路径部分均被视作是Endpoint的一部分

### Endpoint后带.后跟返回类型

```shell
# Before
https://domain/service/version/endpoint.json

# Now
https://domain/service/version/endpoint
```

现在不再需要支持该方法，客户端可传递`Accept`头来指定期望服务端返回的数据类型

目前支持的有：

- `application/json`
- `application/vnd.msgpack`

### 使用`PUT`方法更新数据

目前版本下`PUT`方法被淘汰
