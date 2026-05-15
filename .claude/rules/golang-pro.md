# Go 后端开发规范

Go 后端所有代码必须遵循以下规则。

## 并发 & Context

- 所有阻塞/IO 操作必须传入 `context.Context`
- 协程 (goroutine) 必须有明确生命周期管理，禁止创建无 lifecycle 的裸 goroutine
- 使用 `errgroup` 或 `sync.WaitGroup` 管理并发任务

## 错误处理

- 所有错误必须显式处理，禁止 `_` 忽略
- 禁止用 `panic` 做常规错误处理
- 错误传播使用 `fmt.Errorf("%w", err)` 包装，保持完整错误链
- 业务错误统一使用 `internal/pkg/errors` 定义错误码

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
