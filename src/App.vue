<script setup lang="ts">
import { onMounted, ref } from "vue";
import { useAppStore } from "@/stores/app";
import { invoke } from "@tauri-apps/api/core";
import AppLayout from "@/components/AppLayout.vue";
import InitialSetup from "@/components/InitialSetup.vue";
import Sonner from "@/components/ui/Sonner.vue";
import type { AppConfig } from "@/stores/app";

const store = useAppStore();
const loading = ref(true);

onMounted(async () => {
  try {
    const initialized: boolean = await invoke("is_initialized_cmd");
    store.isInitialized = initialized;

    if (initialized) {
      const config: AppConfig = await invoke("get_config_cmd");
      store.config = config;
      store.applyTheme(config.theme as "light" | "dark");
    }
  } catch (e) {
    console.error("Failed to load config:", e);
  } finally {
    loading.value = false;
  }
});

function onSetupComplete(config: AppConfig) {
  store.config = config;
  store.isInitialized = true;
  store.applyTheme(config.theme as "light" | "dark");
}
</script>

<template>
  <div v-if="loading" class="flex h-screen items-center justify-center">
    <p class="text-muted-foreground">正在加载...</p>
  </div>
  <template v-else>
    <InitialSetup v-if="!store.isInitialized" @complete="onSetupComplete" />
    <AppLayout v-else />
    <Sonner />
  </template>
</template>
