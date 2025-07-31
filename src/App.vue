<script setup lang="ts">
import AppTheme from './components/AppTheme.vue'
import { ref } from 'vue'
import { theme } from 'ant-design-vue';
const initialTheme =
  localStorage.getItem("theme") ??
  (globalThis.matchMedia("(prefers-color-scheme: dark)").matches
    ? "dark"
    : "light");
const currentTheme = ref(initialTheme)
function toggleTheme() {
  currentTheme.value = currentTheme.value === 'dark' ? 'light' : 'dark'
}
</script>

<template>
  <a-config-provider :theme="{
    algorithm: currentTheme === 'dark' ? theme.darkAlgorithm : theme.defaultAlgorithm,
  }">
    <div className="container m-2 md:mx-auto text-center w-[95%] h-screen">
      <header className="flex justify-between items-center text-xl font-semibold h-20 dark:text-neutral-100">
        <div className="flex m-4 items-center space-x-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
            <mask id="lineMdCheckAll0">
              <g fill="none" stroke="#fff" stroke-dasharray="24" stroke-dashoffset="24" stroke-linecap="round"
                stroke-linejoin="round" stroke-width="2">
                <path d="M2 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" dur="0.4s" values="24;0" />
                </path>
                <path stroke="#000" strokeWidth={6} d="M7.5 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.4s" dur="0.4s" values="24;0" />
                </path>
                <path d="M7.5 13.5l4 4l10.75 -10.75">
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.4s" dur="0.4s" values="24;0" />
                </path>
              </g>
            </mask>
            <rect width="24" height="24" fill="currentColor" mask="url(#lineMdCheckAll0)" />
          </svg>
          <h1>TODO App</h1>
        </div>
        <AppTheme :theme="currentTheme" @toggle-theme="toggleTheme" />
      </header>
    </div>
  </a-config-provider>
</template>

<style scoped></style>
