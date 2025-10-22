[return](/ruby/index)

### 学习型方向，rails8主流框架
```
Hotwire + Tailwind CSS（Rails 官方推荐，近年最流行）
技术组合：Rails 8 内置的 Hotwire（包含 Turbo 和 Stimulus）+ Tailwind CSS（实用优先的 CSS 框架）。
特点：
Hotwire 通过 Turbo Streams 实现页面局部更新（无需写大量 AJAX），Stimulus 处理轻量交互（如表单验证、动态显示），替代传统 jQuery；
Tailwind 通过原子类快速构建自定义 UI，配合 Rails 的 ERB 模板或 Slim，开发效率极高。
优势：无需引入 React/Vue 等重型框架，保持 Rails “前后端一体” 的开发体验，同时支持现代交互，是当前 Rails 社区的主流趋势。
适用场景：需要定制化 UI、中等复杂度交互（如实时数据刷新、动态表单）的后台。

```