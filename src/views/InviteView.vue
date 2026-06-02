<script setup lang="ts">
import { ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Card from "@/components/ui/Card.vue";
import Select from "@/components/ui/Select.vue";
import { SelectItem, SelectItemText } from "radix-vue";
import { toast } from "vue-sonner";
import { Send, Loader2 } from "lucide-vue-next";

const userAlias = ref("");
const userEmail = ref("");
const userRole = ref("internal_user_viewer");
const loading = ref(false);

const roleOptions = [
  { value: "proxy_admin", label: "网关管理员 (proxy_admin)" },
  { value: "proxy_admin_viewer", label: "审计管理员 (proxy_admin_viewer)" },
  { value: "internal_user", label: "普通用户 (internal_user)" },
  { value: "internal_user_viewer", label: "普通只读用户 (internal_user_viewer)" },
];

async function handleInvite() {
  if (!userEmail.value.trim()) {
    toast.error("请输入用户邮箱");
    return;
  }
  if (!userAlias.value.trim()) {
    toast.error("请输入用户名");
    return;
  }

  // Simple email validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(userEmail.value.trim())) {
    toast.error("请输入有效的邮箱地址");
    return;
  }

  loading.value = true;
  try {
    const result = await invoke("invite_user", {
      userEmail: userEmail.value.trim(),
      userAlias: userAlias.value.trim(),
      userRole: userRole.value,
    });
    const r = result as any;
    toast.success("邀请成功！邮件已发送");
    console.log("Invite result:", r);
    // Clear form
    userAlias.value = "";
    userEmail.value = "";
  } catch (e) {
    toast.error(`邀请失败: ${e}`);
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="flex h-full items-center justify-center px-4">
    <Card class="w-full max-w-lg p-8">
      <div class="mb-6 text-center">
        <div class="mx-auto mb-3 flex h-10 w-10 items-center justify-center rounded-lg bg-primary/10">
          <Send class="h-5 w-5 text-primary" />
        </div>
        <h2 class="text-lg font-semibold">邀请新用户</h2>
        <p class="mt-1 text-sm text-muted-foreground">
          填写用户信息，系统将自动创建账号并发送邀请邮件
        </p>
      </div>

      <div class="space-y-4">
        <!-- Username -->
        <div class="space-y-2">
          <Label for="alias">用户名</Label>
          <Input
            id="alias"
            v-model="userAlias"
            placeholder="请输入用户备注名"
            :disabled="loading"
          />
        </div>

        <!-- Email -->
        <div class="space-y-2">
          <Label for="email">用户邮箱</Label>
          <Input
            id="email"
            v-model="userEmail"
            type="email"
            placeholder="user@example.com"
            :disabled="loading"
            @keyup.enter="handleInvite"
          />
        </div>

        <!-- Role -->
        <div class="space-y-2">
          <Label>用户角色</Label>
          <Select v-model="userRole" placeholder="选择用户角色" :disabled="loading">
            <SelectItem
              v-for="opt in roleOptions"
              :key="opt.value"
              :value="opt.value"
              class="relative flex cursor-pointer select-none items-center rounded-sm px-2 py-1.5 text-sm outline-none hover:bg-accent hover:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
            >
              <SelectItemText>{{ opt.label }}</SelectItemText>
            </SelectItem>
          </Select>
        </div>

        <!-- Submit -->
        <Button class="w-full" size="lg" :disabled="loading" @click="handleInvite">
          <Loader2 v-if="loading" class="mr-2 h-4 w-4 animate-spin" />
          <Send v-else class="mr-2 h-4 w-4" />
          {{ loading ? "邀请发送中..." : "发送邀请" }}
        </Button>
      </div>
    </Card>
  </div>
</template>
