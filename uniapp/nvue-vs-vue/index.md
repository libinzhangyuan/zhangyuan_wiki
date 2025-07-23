[返回](/uniapp/index)

# Uniapp 中的 nvue 介绍

## 什么是 nvue？

nvue 是 Uniapp 中一种特殊的页面文件类型，全称是 **native vue**，它是针对原生渲染优化的页面开发方式。

### 主要特点：

1. **原生渲染**：使用原生组件而非 WebView 渲染，性能更高
2. **样式限制**：样式写法有限制，更接近 CSS 的子集
3. **平台差异**：在不同平台上有不同的表现
4. **性能优势**：适合复杂列表、高性能要求的场景

## nvue 与 vue 页面的对比

```
| 特性            | nvue 页面               | 普通 vue 页面           |
|----------------|------------------------|------------------------|
| 渲染方式        | 原生组件渲染             | WebView 渲染            |
| 性能           | 更高，适合复杂列表        | 一般                    |
| 样式支持        | 受限的 CSS 子集          | 完整 CSS 支持           |
| 开发体验        | 需要适应平台差异          | 统一开发体验            |
| 适用场景        | 高性能需求页面            | 普通页面                |
| 组件支持        | 部分组件不支持            | 支持所有组件            |
| 跨平台一致性    | 需要处理平台差异          | 一致性较好              |
```

## nvue 使用示例

### 1. 创建 nvue 页面

在 pages.json 中配置：

```json
{
  "pages": [
    {
      "path": "pages/nvuePage",
      "style": {
        "navigationBarTitleText": "nvue 示例",
        "renderer": "native"
      }
    }
  ]
}
```

### 2. nvue 文件示例 (nvuePage.nvue)

```html
<template>
  <view class="container">
    <text class="title">这是一个 nvue 页面</text>
    <list class="list">
      <cell v-for="(item, index) in items" :key="index">
        <text class="item-text">{{ item }}</text>
      </cell>
    </list>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        items: ['项目1', '项目2', '项目3', '项目4']
      }
    }
  }
</script>

<style>
  .container {
    flex: 1;
    background-color: #f5f5f5;
  }
  .title {
    font-size: 36rpx;
    color: #333;
    padding: 20rpx;
  }
  .list {
    flex: 1;
  }
  .item-text {
    font-size: 28rpx;
    padding: 20rpx;
    border-bottom-width: 1px;
    border-bottom-color: #eee;
  }
</style>
```

## nvue 样式限制

```
| CSS 特性       | nvue 支持情况            | 备注                     |
|---------------|-------------------------|--------------------------|
| flex 布局      | 完全支持                 | 推荐使用 flex 布局        |
| position      | 仅支持 relative 和 absolute |                        |
| 盒模型         | 部分支持                 | 不支持 margin 合并        |
| 选择器         | 有限支持                 | 不支持属性选择器等复杂选择器 |
| 过渡动画        | 部分支持                 | 使用 uni.createAnimation  |
| 伪类伪元素      | 不支持                  |                          |
| z-index       | 不支持                  |                          |
```

## 何时使用 nvue

1. 需要高性能长列表滚动
2. 需要复杂动画或手势交互
3. 需要高性能的拖拽操作
4. 需要更高的渲染性能

## 注意事项

1. nvue 页面不能直接使用 Vue 的过渡动画
2. 样式写法有限制，需要适应
3. 不同平台可能有差异，需要测试
4. 不是所有 Uniapp 组件都支持 nvue

希望这个介绍对您理解 Uniapp 中的 nvue 有所帮助！如需更详细的信息，可以参考 Uniapp 官方文档中关于 nvue 的部分。