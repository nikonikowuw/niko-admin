# React 性能优化规范

React 组件开发时必须遵循以下性能规则。

## 消除异步瀑布 (CRITICAL)

- 无依赖的异步操作用 `Promise.all()` 并发执行，禁止顺序 await
- 用 Suspense 边界包裹数据获取组件，布局先渲染（骨架屏）数据流式加载
- Server Action / API Route 中立即启动所有独立 Promise，最后真正使用时再 await

## 包体积 (CRITICAL)

- 禁止 barrel import（`export *` 的 index.js），直接从源文件导入
- 重型组件用 `next/dynamic` 懒加载（`ssr: false`）
- 第三方分析/日志库延迟到 hydration 之后加载

## 服务端安全 (CRITICAL)

- 每个 Server Action / API Route 内部独立验证认证和授权
- 不能仅依赖 middleware 或 layout 的权限检查

## 缓存 & 性能 (HIGH)

- 同请求内使用 `React.cache()` 对数据库查询去重
- 跨请求共享使用 LRU 缓存
- 静态资源（字体/Logo/配置）提升到模块级别，只加载一次

## 数据传输 (HIGH)

- RSC 边界只传递客户端实际需要的字段，禁止序列化整个对象
- 避免服务端 `.toSorted()` / `.filter()` 生成重复引用

## 重渲染优化 (MEDIUM)

- 可从 props/state 计算出的值禁止存储到 state 或 effect 中更新
- 用 `useDeferredValue` 保持输入响应性，避免阻塞用户交互
- 高频变化值用 `useRef` 存储而非 `useState`

## 渲染正确性 (MEDIUM)

- 禁止组件内部定义子组件（导致每次重渲染完全重新挂载，丢失状态）
- 条件渲染用三元 `? :` 而非 `&&`（`&&` 可能渲染出 `0` 或 `NaN`）
- `useState` 昂贵初始值必须传函数：`useState(() => computeExpensive())`
