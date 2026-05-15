# 前端 UI/UX 开发规范

前端 UI 组件开发时必须遵循以下规则。

## 无障碍 (CRITICAL)

- 颜色对比度 >= 4.5:1（正常文本）/ >= 3:1（大文本 18px+）
- 所有可交互元素必须有可见 focus 环（2-4px outline）
- 图标按钮必须有 `aria-label` 标注
- Tab 导航顺序必须匹配视觉顺序

## 触控交互 (CRITICAL)

- 触摸目标最小 44x44px
- 触控目标间距 >= 8px
- 禁止仅依赖 hover 交互

## 性能 (HIGH)

- 图片使用 WebP/AVIF 格式
- 响应式图片：`srcset` + `sizes` 属性
- 图片懒加载，声明 `width`/`height` 防止 CLS < 0.1
- 列表 >= 50 项必须虚拟化

## 组件风格 (HIGH)

- 统一使用 SVG 图标（Heroicons / Lucide）
- 禁止 emoji 做功能图标
- 同项目统一图标风格和描边粗细

## 布局 & 响应式 (HIGH)

- 移动优先设计
- 禁止移动端水平滚动
- 间距系统：4/8dp 增量
- 固定元素（header/navbar）预留 safe padding

## 颜色 & 暗色模式 (MEDIUM)

- 定义语义化 color token 使用，禁止组件内直接写 hex 色值
- 暗色模式：去饱和化浅色变体，非简单颜色反转

## 动画 (MEDIUM)

- 微交互时长 150-300ms，复杂过渡 <= 400ms
- 仅可使用 `transform` / `opacity` 动画
- 禁止动画 `width` / `height` / `top` / `left`
- 必须尊重 `prefers-reduced-motion`：`@media (prefers-reduced-motion: reduce)`

## 表单 (MEDIUM)

- 每个输入必须有可见 label，禁止纯 placeholder
- 错误信息放在对应字段下方
- 提交后自动聚焦首个错误字段
