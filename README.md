# Vue 3 + TypeScript + Vite 通过 SASS 切换主题

### 安装依赖
```
yarn add -D sass @types/node
```

### 定义主题
`src\assets\stylesheets\themes.scss`
```
$themes: (
    light: (
        color-primary: #6750a4,
        color-secondary: #625b71,
    ),
    dark: (
        color-primary: #d0bcff,
        color-secondary: #ccc2dc,
    ),
);

$currentTheme: light;

@mixin useTheme() {
    @each $key, $value in $themes {
        $currentTheme: $key !global;
        html[data-theme="#{$key}"] & {
            @content;
        }
    }
}

@function getVar($key) {
    @return map-get(map-get($themes, $currentTheme), $key);
}

```

### 主题动态切换
`src\hooks\use-theme.ts`
```
import { ref, watchEffect } from "vue";

type Theme = "light" | "dark" | "os";
const LOCAL_KEY = "__theme__";
const theme = ref<Theme>((localStorage.getItem(LOCAL_KEY) as Theme) || "light");
const isDark = ref(theme.value == "dark");

const match = matchMedia("(prefers-color-scheme: dark)");

function followOSTheme() {
    if (match.matches) {
        document.documentElement.dataset.theme = "dark";
    } else {
        document.documentElement.dataset.theme = "light";
    }
    isDark.value = match.matches;
}

watchEffect(() => {
    localStorage.setItem(LOCAL_KEY, theme.value);

    if (theme.value == "os") {
        followOSTheme();
        match.addEventListener("change", followOSTheme);
    } else {
        document.documentElement.dataset.theme = theme.value;
        isDark.value = theme.value == "dark";
        match.removeEventListener("change", followOSTheme);
    }
});

export const useTheme = () => {
    return { theme, isDark };
};

```

### vite 配置
`vite.config.ts`
```
import vue from "@vitejs/plugin-vue";
import * as path from "path";
import { defineConfig } from "vite";

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()],
    resolve: {
        alias: {
            "@": path.resolve(__dirname, "./src"),
        },
    },

    css: {
        preprocessorOptions: {
            scss: {
                additionalData: `@import "@/assets/stylesheets/themes.scss";`,
            },
        },
    },
});

```

### 使用
`src\App.vue`
```
<script setup lang="ts">
import { useTheme } from "./hooks/use-theme";
const { theme } = useTheme();
</script>

<template>
    <div class="app">
        <div>
            <select v-model="theme">
                <option value="os">OS</option>
                <option value="light">Light</option>
                <option value="dark">Dark</option>
            </select>
        </div>
        <div class="container"></div>
    </div>
</template>

<style lang="scss" scoped>
.app {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.container {
    width: 400px;
    height: 400px;
    @include useTheme {
        background-color: getVar(color-primary);
    }
}
</style>

```