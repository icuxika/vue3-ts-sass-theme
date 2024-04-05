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
