<script setup lang="ts">
import { ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Card from "@/components/ui/Card.vue";
import { toast } from "vue-sonner";
import type { AppConfig } from "@/stores/app";

const emit = defineEmits<{
  complete: [config: AppConfig];
}>();

const apiKey = ref("");
const loading = ref(false);

async function handleSave() {
  if (!apiKey.value.trim()) {
    toast.error("请输入 API Key");
    return;
  }
  loading.value = true;
  try {
    const config: AppConfig = await invoke("get_config_cmd");
    config.apiKey = apiKey.value.trim();
    await invoke("save_config_cmd", { config });
    toast.success("API Key 已保存");
    emit("complete", config);
  } catch (e) {
    toast.error(`保存失败: ${e}`);
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="flex h-screen items-center justify-center bg-background">
    <Card class="w-full max-w-md p-8">
      <div class="mb-6 text-center">
        <div class="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-xl bg-secondary text-xl font-bold text-primary-foreground">
          🧊
        </div>
        <h1 class="text-xl font-bold">欢迎使用 Pontus</h1>
        <p class="text-muted-foreground text-sm"></p>
        <p class="mt-2 text-sm text-muted-foreground">
          初次使用，请设置您的 Litellm API Key 以开始管理用户。
        </p>
      </div>

      <div class="space-y-4">
        <div class="space-y-2">
          <Label for="apikey">API Key</Label>
          <Input
            id="apikey"
            v-model="apiKey"
            type="password"
            placeholder="sk-..."
            @keyup.enter="handleSave"
          />
          <p class="text-xs text-muted-foreground">
            此 Key 将安全存储在本机，后续可在设置页面修改。
          </p>
        </div>

        <Button class="w-full" :disabled="loading" @click="handleSave">
          {{ loading ? "保存中..." : "开始使用" }}
        </Button>
      </div>
    </Card>
  </div>
</template>
