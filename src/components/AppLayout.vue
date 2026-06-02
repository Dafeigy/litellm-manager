<script setup lang="ts">
import { useRoute } from "vue-router";
import { cn } from "@/lib/utils";
import {
  UserPlus,
  Settings,
  LayoutDashboard,
} from "lucide-vue-next";

const route = useRoute();

const navItems = [
  { path: "/invite", label: "邀请用户", icon: UserPlus },
  { path: "/dashboard", label: "用户看板", icon: LayoutDashboard },
  { path: "/settings", label: "设置", icon: Settings },
];

function isActive(path: string) {
  return route.path === path || (path === "/invite" && route.path === "/");
}
</script>

<template>
  <div class="flex h-screen bg-background">
    <!-- Sidebar -->
    <aside class="flex w-56 flex-col border-r bg-card">
      <div class="flex h-14 items-center gap-2 border-b px-4">
        <div class="flex h-8 w-8 items-center justify-center rounded-md bg-primary text-sm font-bold text-primary-foreground">
          LA
        </div>
        <span class="font-semibold">Litellm Admin</span>
      </div>
      <nav class="flex-1 space-y-1 p-3">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          :class="
            cn(
              'flex items-center gap-3 rounded-md px-3 py-2 text-sm font-medium transition-colors',
              isActive(item.path)
                ? 'bg-primary/10 text-primary'
                : 'text-muted-foreground hover:bg-accent hover:text-accent-foreground',
            )
          "
        >
          <component :is="item.icon" class="h-4 w-4" />
          {{ item.label }}
        </router-link>
      </nav>
      <div class="border-t p-3 text-xs text-muted-foreground">
        v0.1.0
      </div>
    </aside>

    <!-- Main content -->
    <main class="flex-1 overflow-auto">
      <router-view />
    </main>
  </div>
</template>
