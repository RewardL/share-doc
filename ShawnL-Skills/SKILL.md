---
name: luoshang
description: luoshang 的个人编码规范，适用于 Vue 3 + TypeScript 项目。当编写 Vue SFC、composables、Pinia stores、API 层、SCSS/BEM 样式、Vite 配置时使用。适用于任何 Vue 3 Composition API 开发、.vue 文件、TypeScript、SCSS、pnpm monorepo 项目。
---

## 编码实践

### 代码组织

- **单一职责**：一个文件只做一件事；超过 ~300 行考虑拆分
- **类型分离**：类型定义放独立的 `types/` 目录或 `types.ts`
- **常量提取**：枚举和常量放独立文件（如 `constants.ts`）
- **导入顺序**：第三方库 → 本地模块 → 类型

### TypeScript

- 结构化对象用 `interface`（props、API 响应、store 状态）
- 固定值集合用 `const enum`
- 可选链处理不确定调用：`obj?.method?.()`
- `strict: true`

### 注释

- 避免多余注释，代码应当自解释
- 注释只解释非显而易见的意图、权衡或约束

### 格式化

- 4 空格缩进
- 单引号
- 语句末尾加分号
- 多行尾逗号：`always-multiline`
- 最大行宽：120 字符

---

## Vue 3 规范

### 组件结构

顺序：`<template>` → `<script setup lang="ts">` → `<style lang="scss" scoped>`

```vue
<template>
  <div class="module-name">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
defineOptions({ name: 'ModuleName' });

// 1. 导入
// 2. props 和 emits
// 3. 响应式状态（ref、reactive、computed）
// 4. composables 和 hooks
// 5. watchers
// 6. 生命周期钩子
// 7. 方法
// 8. defineExpose（按需）
</script>

<style lang="scss" scoped>
.module-name {
  // 样式
}
</style>
```

### Props 和 Emits

```ts
// Props：类型化 + withDefaults
const props = withDefaults(defineProps<{
    width: number;
    autoPlay: boolean;
}>(), {
    width: 120,
    autoPlay: true,
});

// Emits：接口类型风格
interface Emits {
    (e: 'ready', data: any): void;
    (e: 'error', error: Error): void;
}
const emit = defineEmits<Emits>();

// v-model：使用 defineModel
const isVisible = defineModel('isVisible', { required: true, type: Boolean });
```

### 条件渲染

- `v-show`：频繁切换的元素（弹窗、面板）
- `v-if`：条件渲染的内容（权限判断、功能开关）

### 图片

```html
<img class="block__image" src="..." alt="" />
```

`alt` 属性固定为空字符串。

### 路由

- 使用 `createWebHashHistory(import.meta.env.BASE_URL)`
- 首屏：直接导入以加快加载
- 其他页面：懒加载 `() => import('@/views/...')`
- 使用 `keep-alive` 配合 `include` 缓存关键视图

---

## Composables 和 Hooks

### 命名与结构

- 统一使用 `use` 前缀：`useImageCache`、`usePolling`
- 返回包含响应式状态和方法的普通对象
- 在 `onUnmounted` 中清理资源

```ts
export function useFeature(param: Ref<string>) {
    const isLoading = ref(false);
    const hasError = ref(false);
    const data = ref<T | null>(null);

    const execute = async () => {
        if (!param.value) return;
        isLoading.value = true;
        try {
            data.value = await fetchData(param.value);
            hasError.value = false;
        } catch (error) {
            hasError.value = true;
            console.warn('Feature error:', error);
        } finally {
            isLoading.value = false;
        }
    };

    onUnmounted(() => { /* 清理 */ });

    return { isLoading, hasError, data, execute };
}
```

### Composable 与 Hook 的区别

- **Composables**（`composables/`）：共享业务逻辑、有状态的工具函数
- **Hooks**（`hooks/`）：UI 效果、副作用较重的操作

---

## 状态管理（Pinia）

### Setup Store 模式（始终使用此模式）

```ts
import { ref, computed } from 'vue';
import { defineStore } from 'pinia';

export default defineStore('useFeatureStore', () => {
    const list = ref<Item[]>([]);
    const selectedId = ref<string>('');

    const selectedItem = computed(() =>
        list.value.find((item) => item.id === selectedId.value)
    );

    const setList = (items: Item[]) => {
        list.value = items;
    };

    return { list, selectedId, selectedItem, setList };
});
```

- store 文件使用 `export default` 导出
- 组件中使用 `storeToRefs()` 获取响应式引用
- 直接调用 action：`useFeatureStore().setList(items)`

---

## API 层

### URL 管理

```ts
export const enum ApiUrl {
    getList = '/api/module/list',
    getDetail = '/api/module/detail',
}
```

### 请求与响应

- 从响应中解构 `{ errno, data }`（或项目约定的结构）
- `errno === 0` 表示成功
- 错误反馈使用 UI 库的 Toast 组件

```ts
const fetchList = async () => {
    try {
        const { errno, data } = await getListReq({ page: 1, size: 20 });
        if (errno === 0) {
            list.value = data.list;
        } else {
            showToast('加载失败');
        }
    } catch (error) {
        console.error('请求失败:', error);
    }
};
```

---

## 样式

### SCSS + BEM

完整 BEM 规则参见 [BEM 命名规范](references/bem-conventions.md)。

核心规则：
- Element 必须是单个单词：`&__header`、`&__content`
- 一层嵌套 → subpart（单连字符）：`&__header-title`
- 多层嵌套 → 创建新的 Block
- 绝对禁止 Element 嵌套 Element：不能有 `block__element__subelement`
- 修饰符用双连字符：`&--active`、`&--disabled`
- 动态状态用 `is-*` 类：`.is-active`、`.is-loading`

```scss
.feature-card {
    &__header {
        display: flex;
    }
    &__header-title {
        font-weight: bold;
    }
    &__content {
        padding: 16px;
    }
    &__content--highlighted {
        background: #f0f0f0;
    }
}
```

### 样式习惯

- 尽可能使用 flex 布局
- 使用 `px` 单位
- 在 `<style>` 中使用 `v-bind()` 绑定动态值
- 始终使用 `scoped` 样式

---

## 工具链偏好

### Vite

- 使用 `unplugin-auto-import` + `unplugin-vue-components` 实现自动导入
- 手动分包：vendor（vue, pinia）、vueRouter、UI 库

### pnpm

- 优先使用 pnpm 作为包管理器
- monorepo 用 pnpm workspace

### 常用库

- **UI**：Vant（移动端）或按项目选择
- **工具**：VueUse、lodash-es、dayjs
- **动画**：GSAP
- **安全**：DOMPurify、xss

---

## 参考文档

| 主题 | 说明 | 文件 |
|------|------|------|
| BEM 命名规范 | 完整 BEM 规则、决策树、示例 | [bem-conventions](references/bem-conventions.md) |
| 代码模式 | 组件、composable、store、路由模式 | [code-patterns](references/code-patterns.md) |
