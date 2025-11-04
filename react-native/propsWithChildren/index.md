[return](../index)


`propsWithChildren` 是 React 内置的工具类型，核心作用是给自定义的 Props 类型自动添加 `children` 属性类型，无需手动声明即可让组件支持子元素传递。


### 一、核心概念
`propsWithChildren` 本质是一个泛型类型，接收一个自定义 Props 类型作为参数，返回一个包含该自定义 Props + `children` 属性的新类型。

- **来源**：从 `react` 库中直接导入，无需额外安装依赖。
- **作用**：简化组件 Props 类型定义，避免每次都手动写 `children?: React.ReactNode`。
- **基础示例**：
  ```
  import { propsWithChildren } from 'react';
  import React from 'react';

  // 1. 定义自定义 Props 类型（不含 children）
  type CardProps = {
    title: string; // 卡片标题（自定义属性）
    size?: 'small' | 'medium' | 'large'; // 卡片尺寸（可选自定义属性）
  };

  // 2. 使用 propsWithChildren 包装，自动获得 children 属性
  type CardWithChildrenProps = propsWithChildren<CardProps>;

  // 3. 组件使用（可直接接收 title、size 和 children）
  const Card = (props: CardWithChildrenProps) => {
    const { title, size = 'medium', children } = props;
    return (
      <div className={`card card--${size}`}>
        <h3 className="card__title">{title}</h3>
        <div className="card__content">{children}</div>
      </div>
    );
  };

  // 4. 调用组件（传递子元素）
  const App = () => {
    return (
      <Card title="我的卡片" size="large">
        {/* 这里的内容会作为 children 传递给 Card 组件 */}
        <p>这是卡片内部的内容</p>
        <button>点击按钮</button>
      </Card>
    );
  };
  ```
  **返回结果**（组件渲染后）：
  ```html
  <div class="card card--large">
    <h3 class="card__title">我的卡片</h3>
    <div class="card__content">
      <p>这是卡片内部的内容</p>
      <button>点击按钮</button>
    </div>
  </div>
  ```


### 二、常见使用场景
`propsWithChildren` 主要用于需要承载子元素的组件，以下是两个典型场景：

#### 场景 1：布局组件（如 Layout、Container）
```
import { propsWithChildren } from 'react';

// 布局组件 Props（仅需定义自定义属性，children 由 propsWithChildren 自动添加）
type LayoutProps = {
  hasHeader?: boolean; // 是否显示头部
  hasFooter?: boolean; // 是否显示底部
};

const Layout = (props: propsWithChildren<LayoutProps>) => {
  const { hasHeader = true, hasFooter = true, children } = props;
  return (
    <div className="layout">
      {hasHeader && <header className="layout__header">网站头部</header>}
      <main className="layout__main">{children}</main> {/* 承载页面内容（子元素） */}
      {hasFooter && <footer className="layout__footer">网站底部</footer>}
    </div>
  );
};

// 调用布局组件
const HomePage = () => {
  return (
    <Layout hasFooter={false}>
      <h1>首页</h1>
      <p>首页的具体内容...</p>
    </Layout>
  );
};
```
**返回结果**（组件渲染后）：
```html
<div class="layout">
  <header class="layout__header">网站头部</header>
  <main class="layout__main">
    <h1>首页</h1>
    <p>首页的具体内容...</p>
  </main>
  <!-- hasFooter 为 false，不渲染底部 -->
</div>
```

#### 场景 2：容器组件（如 Modal、Dropdown）
```
import { propsWithChildren, useState } from 'react';

// 弹窗组件 Props
type ModalProps = {
  isOpen: boolean; // 是否显示弹窗
  onClose: () => void; // 关闭弹窗的回调
  title: string; // 弹窗标题
};

const Modal = (props: propsWithChildren<ModalProps>) => {
  const { isOpen, onClose, title, children } = props;
  if (!isOpen) return null;

  return (
    <div className="modal__backdrop">
      <div className="modal__content">
        <div className="modal__header">
          <h2>{title}</h2>
          <button onClick={onClose}>&times;</button>
        </div>
        <div className="modal__body">{children}</div> {/* 承载弹窗内容（子元素） */}
      </div>
    </div>
  );
};

// 调用弹窗组件
const App = () => {
  const [modalOpen, setModalOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setModalOpen(true)}>打开弹窗</button>
      <Modal isOpen={modalOpen} onClose={() => setModalOpen(false)} title="提示">
        <p>这是弹窗里的内容，需要用户确认</p>
        <button onClick={() => setModalOpen(false)}>确认</button>
      </Modal>
    </div>
  );
};
```
**返回结果**（点击“打开弹窗”后）：
```html
<div>
  <button>打开弹窗</button>
  <div class="modal__backdrop">
    <div class="modal__content">
      <div class="modal__header">
        <h2>提示</h2>
        <button>&times;</button>
      </div>
      <div class="modal__body">
        <p>这是弹窗里的内容，需要用户确认</p>
        <button>确认</button>
      </div>
    </div>
  </div>
</div>
```


### 三、与普通 Props 类型的对比
使用 `propsWithChildren` 可显著简化代码，以下是它与“手动声明 children”的对比：

```
| 对比维度         | 使用 propsWithChildren                | 手动声明 children                     |
| ---------------- | ------------------------------------- | ------------------------------------- |
| children 支持    | 自动添加，无需手动写类型              | 需手动声明 `children?: React.ReactNode` |
| 类型安全性       | 与 React 内置类型对齐，避免类型错误   | 需自行保证 children 类型正确          |
| 使用场景         | 所有需要承载子元素的组件              | 仅适用于简单组件，复杂场景易遗漏      |
| 代码示例         | type Props = propsWithChildren<{a: number}> | type Props = {a: number; children?: React.ReactNode} |
```


要不要我帮你整理一份 **`propsWithChildren` 常用场景代码模板**，包含布局、弹窗、卡片等组件的完整示例，方便你直接复制到项目中使用？