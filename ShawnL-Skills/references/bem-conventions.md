# BEM CSS 命名规范

## 核心规则

### 规则 1：Element 必须是单个单词

```scss
// ✅ 正确
.block__header { }
.block__content { }
.block__button { }

// ❌ 错误：Element 不能是两个单词连接
.block__main-title { }
.block__nav-item { }
```

### 规则 2：DOM 嵌套处理策略

**一层嵌套 → subpart（单连字符）**

```html
<div class="card">
  <div class="card__header">
    <h1 class="card__header-title">标题</h1>
    <span class="card__header-subtitle">副标题</span>
  </div>
</div>
```

```scss
.card {
  &__header { }
  &__header-title { }
  &__header-subtitle { }
}
```

**多层嵌套 → 创建新的 Block**

```html
<div class="product-list">
  <div class="product-list__container">
    <div class="product-card">
      <div class="product-card__image">
        <img class="product-card__image-thumbnail" src="...">
      </div>
      <div class="product-card__content">
        <h3 class="product-card__content-title">商品名</h3>
      </div>
    </div>
  </div>
</div>
```

### 规则 3：绝对禁止 Element 嵌套 Element

```scss
// ❌ 绝对禁止
.block {
  &__element {
    &__subelement { } // 会生成 block__element__subelement
  }
}

// ✅ 正确：使用 subpart 或新 Block
.block {
  &__element { }
  &__element-subpart { }
}
```

### 规则 4：连接符号

| 符号 | 用途 | 示例 |
|------|------|------|
| `__`（双下划线） | Block → Element | `.card__header` |
| `--`（双连字符） | 修饰符 | `.card__header--active` |
| `-`（单连字符） | Element 的 subpart | `.card__header-title` |

## 决策树

```
DOM 有嵌套？
├─ 是 → 只有一层嵌套？
│   ├─ 是 → block__element-subpart ✅
│   └─ 否 → 创建新 Block ✅
└─ 否 → block__element ✅
```

## 推荐的 Element 命名

```scss
.block {
  &__header { }     // 头部
  &__content { }    // 内容
  &__footer { }     // 底部
  &__title { }      // 标题
  &__text { }       // 文本
  &__image { }      // 图片
  &__button { }     // 按钮
  &__icon { }       // 图标
  &__list { }       // 列表
  &__item { }       // 项目
  &__container { }  // 容器
  &__wrapper { }    // 包装器
}
```

## 状态类

- JavaScript 控制的动态状态用 `is-*`
- 静态样式变体用 `--modifier`

```scss
.page-btn__item { display: none; }
.page-btn__item.is-active { display: block; }
```

| 使用场景 | 状态类 | BEM 修饰符 |
|----------|--------|-----------|
| JS 控制 | ✅ `is-active` | ❌ |
| 静态变体 | ❌ | ✅ `button--large` |
| 动态状态 | ✅ `is-loading` | ❌ |
| 组件变体 | ❌ | ✅ `card--primary` |

## SCSS 语法

```scss
.block {
  // Block 基础样式

  &__element {
    // Element 样式
    &--modifier { }
  }

  &__element-subpart {
    // Subpart 样式
    &--modifier { }
  }

  &--modifier {
    // Block 修饰符样式
  }
}
```

## 汇总

| 模式 | 推荐 | 示例 |
|------|:----:|------|
| `block__element` | ✅ | `.card__header` |
| `block__element--modifier` | ✅ | `.card__header--active` |
| `block__element-subpart` | ✅ | `.card__header-title` |
| `block__element-subpart--modifier` | ✅ | `.card__header-title--bold` |
| `block__main-title`（两个单词的 element） | ❌ | 改用 subpart 或新 Block |
| `block__element__subelement`（嵌套） | ❌ | 禁止使用 |
