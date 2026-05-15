# Go 后端开发规范

Go 后端所有代码必须遵循以下规则。

## 并发 & Context

- 所有阻塞/IO 操作必须传入 `context.Context`
- 协程 (goroutine) 必须有明确生命周期管理，禁止创建无 lifecycle 的裸 goroutine
- 使用 `errgroup` 或 `sync.WaitGroup` 管理并发任务

### Context 传播规则

- **Handler 层**：从 `c.Request.Context()` 获取请求上下文，传递给 Service 层
- **Service 层**：接收 `ctx context.Context` 作为第一个参数，传递给 Repository 层和外部调用
- **Repository 层**：接收 `ctx context.Context` 作为第一个参数，使用 `r.db.WithContext(ctx)` 执行数据库查询
- **中间件层**：使用 `c.Request.Context()` 而非 `context.Background()`，保证请求上下文在整个链路中传播
- 禁止在 Service/Repository/Handler 中使用 `context.Background()` 或 `context.TODO()`（仅允许在 `init()` 或测试中使用）
- 传递上下文时，仅使用 `ctx` 变量名，禁止 `context` 等歧义命名

## 错误处理

- 所有错误必须显式处理，禁止 `_` 忽略
- 错误传播使用 `fmt.Errorf("%w", err)` 包装，保持完整错误链
- 业务错误统一使用 `internal/pkg/errors` 定义错误码

## 全局异常处理

- 所有中间件和 Handler 中的错误统一使用 `c.Error(err)` 挂载到 Gin Context，由全局 Error 中间件统一捕获并格式化响应
- 业务代码禁止直接 `c.AbortWithStatusJSON()` 拼装响应，必须通过 `internal/pkg/response` 和 `internal/pkg/errors` 返回
- Error 中间件规则：
  - 作为 Gin 链的**最后一个**中间件注册（在所有路由之后）
  - 遍历 `c.Errors` 取出每条错误，按错误码映射 HTTP 状态码
  - 错误码格式：`{ "code": <业务错误码>, "message": "<多语言消息>", "error": { ... } }`
  - HTTP 状态码按错误码区间映射：`1xxxx→400`, `2xxxx→401`, `3xxxx→403`, `4xxxx→404`, `5xxxx→500`
- 自定义中间件（auth/rbac/cors/限流）验出错误时使用 `c.Abort()` + `c.Error(err)` 终止链，不得直接写入响应
- 业务代码禁止用 `panic`，仅允许在 `init()` 或不可恢复的致命场景使用
- 单独的 Recovery 中间件（gin.Recovery 或自定义）作为链**最前**兜底 panic，返回统一错误码 50000

## 代码规范

- 所有导出函数、类型、包必须有 GoDoc 注释
- 代码必须通过 `gofmt` 和 `golangci-lint` 检查
- 文件名小写下划线，命名驼峰（导出大写）
- 日志使用 `zap.L()` 结构化日志，禁止 `fmt.Println`

## 测试

- 必须写表驱动测试 (table-driven tests)
- `-race` 竞态检测必须通过
- 覆盖率目标 >= 80%
- 文件命名 `*_test.go`

## 泛型 & 反射

- 泛型使用 `X | Y` union 约束（Go 1.18+）
- 禁止无性能依据地使用反射

## 配置

- 配置禁止硬编码
- 使用 functional options 模式或环境变量注入
- 加载优先级：环境变量 > .env > config.yaml > 默认值

## 架构

- 分层单向依赖：Handler → Service → Repository → Model
- 禁止混合同步和异步模式
- 接口定义在使用方，实现在提供方
