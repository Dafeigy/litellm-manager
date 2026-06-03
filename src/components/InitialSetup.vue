<script setup lang="ts">
import { ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Card from "@/components/ui/Card.vue";
import Separator from "@/components/ui/Separator.vue";
import { toast } from "vue-sonner";
import type { AppConfig } from "@/stores/app";

const emit = defineEmits<{
  complete: [config: AppConfig];
}>();

const apiKey = ref("");
const litellmHost = ref("http://localhost:4000");
const smtpHost = ref("");
const smtpPort = ref("465");
const smtpSenderEmail = ref("");
const smtpUsername = ref("");
const smtpPassword = ref("");
const loading = ref(false);

function buildConfig(): AppConfig {
  return {
    apiKey: apiKey.value.trim(),
    litellmHost: litellmHost.value.trim() || "http://localhost:4000",
    smtpHost: smtpHost.value.trim(),
    smtpPort: Number(smtpPort.value) || 465,
    smtpSenderEmail: smtpSenderEmail.value.trim(),
    smtpUsername: smtpUsername.value.trim(),
    smtpPassword: smtpPassword.value,
    theme: "light",
  };
}

async function handleSave() {
  if (!apiKey.value.trim()) {
    toast.error("请输入 API Key");
    return;
  }
  loading.value = true;
  try {
    const config = buildConfig();
    await invoke("save_config_cmd", { config });
    toast.success("配置已保存");
    emit("complete", config);
  } catch (e) {
    toast.error(`保存失败: ${e}`);
  } finally {
    loading.value = false;
  }
}

async function handleSkipSmtp() {
  if (!apiKey.value.trim()) {
    toast.error("请输入 API Key");
    return;
  }
  loading.value = true;
  try {
    const config = buildConfig();
    await invoke("save_config_cmd", { config });
    toast.success("配置已保存");
    toast.info("SMTP 未配置，可稍后在设置页面完成", { duration: 5000 });
    emit("complete", config);
  } catch (e) {
    toast.error(`保存失败: ${e}`);
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="flex h-screen items-center justify-center bg-background p-4">
    <Card class="w-full max-w-lg p-8">
      <div class="mb-6 text-center">
        <div class="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-xl bg-secondary text-xl font-bold text-primary-foreground">
          🧊
        </div>
        <h1 class="text-xl font-bold">欢迎使用 Pontus</h1>
        <p class="mt-2 text-sm text-muted-foreground">
          初次使用，请配置 Litellm 连接与 SMTP 邮件服务
        </p>
      </div>

      <div class="space-y-5">
        <!-- Litellm 连接配置 -->
        <div class="space-y-3">
          <h3 class="text-sm font-semibold">Litellm 连接配置</h3>
          <div class="space-y-2">
            <Label for="litellm-host">Litellm 服务地址</Label>
            <Input
              id="litellm-host"
              v-model="litellmHost"
              placeholder="http://localhost:4000"
              :disabled="loading"
            />
          </div>
          <div class="space-y-2">
            <Label for="apikey">API Key</Label>
            <Input
              id="apikey"
              v-model="apiKey"
              type="password"
              placeholder="sk-..."
              :disabled="loading"
              @keyup.enter="handleSave"
            />
            <p class="text-xs text-muted-foreground">
              此 Key 将安全存储在本机
            </p>
          </div>
        </div>

        <Separator />

        <!-- SMTP 邮件配置（可选） -->
        <div class="space-y-3">
          <h3 class="text-sm font-semibold">SMTP 邮件配置 <span class="font-normal text-muted-foreground">（可选）</span></h3>
          <div class="grid grid-cols-2 gap-3">
            <div class="space-y-2">
              <Label for="smtp-host">SMTP 服务器</Label>
              <Input
                id="smtp-host"
                v-model="smtpHost"
                placeholder="smtp.example.com"
                :disabled="loading"
              />
            </div>
            <div class="space-y-2">
              <Label for="smtp-port">端口</Label>
              <Input
                id="smtp-port"
                v-model="smtpPort"
                type="number"
                placeholder="465"
                :disabled="loading"
              />
            </div>
          </div>
          <div class="space-y-2">
            <Label for="smtp-sender">发送者邮箱</Label>
            <Input
              id="smtp-sender"
              v-model="smtpSenderEmail"
              type="email"
              placeholder="noreply@example.com"
              :disabled="loading"
            />
          </div>
          <div class="grid grid-cols-2 gap-3">
            <div class="space-y-2">
              <Label for="smtp-username">SMTP 用户名</Label>
              <Input
                id="smtp-username"
                v-model="smtpUsername"
                placeholder="user@example.com"
                :disabled="loading"
              />
            </div>
            <div class="space-y-2">
              <Label for="smtp-password">SMTP 密码</Label>
              <Input
                id="smtp-password"
                v-model="smtpPassword"
                type="password"
                placeholder="••••••••"
                :disabled="loading"
              />
            </div>
          </div>
        </div>

        <!-- Actions -->
        <div class="flex gap-3 pt-2">
          <Button
            variant="outline"
            class="flex-1"
            :disabled="loading"
            @click="handleSkipSmtp"
          >
            {{ loading ? "保存中..." : "跳过 SMTP 设置" }}
          </Button>
          <Button
            class="flex-1"
            :disabled="loading"
            @click="handleSave"
          >
            {{ loading ? "保存中..." : "完成设置" }}
          </Button>
        </div>

        <p class="text-center text-xs text-muted-foreground">
          所有配置可在设置页面随时修改
        </p>
      </div>
    </Card>
  </div>
</template>
