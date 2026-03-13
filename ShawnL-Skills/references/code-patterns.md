# 代码模式

## Vue 组件模式

### 标准 SFC

```vue
<template>
    <div class="feature-page">
        <div class="feature-page__header">
            <span class="feature-page__header-title">{{ title }}</span>
        </div>
        <div class="feature-page__content">
            <slot />
        </div>
    </div>
</template>

<script setup lang="ts">
defineOptions({ name: 'FeaturePage' });

const props = withDefaults(defineProps<{
    title: string;
    loading: boolean;
}>(), {
    loading: false,
});

const emit = defineEmits<{
    (e: 'close'): void;
    (e: 'submit', data: Record<string, any>): void;
}>();

const isVisible = defineModel('isVisible', { required: true, type: Boolean });
</script>

<style lang="scss" scoped>
.feature-page {
    &__header {
        display: flex;
        align-items: center;
    }
    &__header-title {
        font-weight: bold;
    }
    &__content {
        padding: 16px;
    }
}
</style>
```

## Composable 模式

```ts
import { ref, watch, onUnmounted, type Ref } from 'vue';

export function useFeature(param: Ref<string>) {
    const isLoading = ref(false);
    const hasError = ref(false);
    const data = ref<FeatureData | null>(null);

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

    watch(() => param.value, (newVal) => {
        if (newVal) execute();
    }, { immediate: true });

    onUnmounted(() => {
        // 清理资源
    });

    return { isLoading, hasError, data, execute };
}
```

## Pinia Store 模式

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

    const selectItem = (id: string) => {
        selectedId.value = id;
    };

    return { list, selectedId, selectedItem, setList, selectItem };
});
```

组件中使用：

```ts
import { storeToRefs } from 'pinia';
import useFeatureStore from '@/stores/useFeatureStore';

const featureStore = useFeatureStore();
const { list, selectedItem } = storeToRefs(featureStore);
featureStore.selectItem('123');
```

## 路由模式

```ts
import { createRouter, createWebHashHistory } from 'vue-router';
import Home from '@/views/Home/index.vue';

const router = createRouter({
    history: createWebHashHistory(import.meta.env.BASE_URL),
    routes: [
        { path: '/', name: 'Home', component: Home },
        {
            path: '/detail',
            name: 'Detail',
            component: () => import('@/views/Detail/index.vue'),
        },
    ],
});

export default router;
```

## Vite 配置模式

```ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import AutoImport from 'unplugin-auto-import/vite';
import Components from 'unplugin-vue-components/vite';

export default defineConfig({
    plugins: [
        vue(),
        AutoImport({ resolvers: [/* UI 库 resolver */] }),
        Components({ resolvers: [/* UI 库 resolver */] }),
    ],
    resolve: {
        alias: {
            '@': '/src',
        },
    },
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['vue', 'pinia'],
                    vueRouter: ['vue-router'],
                },
            },
        },
    },
});
```

## 全局 SCSS Mixins

```scss
@mixin flex($direction: row, $justify: flex-start, $align: flex-start) {
    display: flex;
    flex-direction: $direction;
    justify-content: $justify;
    align-items: $align;
}
@mixin flex-row-center {
    display: flex;
    justify-content: center;
    align-items: center;
}
@mixin flex-column-center {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
```

使用方式：

```scss
.container {
    @include flex(row, space-between, center);
}
.centered {
    @include flex-row-center;
}
```
