<script setup lang="ts">
import type { HTMLAttributes } from "vue";
import { cn } from "@/lib/utils";

interface SeparatorProps {
  orientation?: "horizontal" | "vertical";
  class?: HTMLAttributes["class"];
  label?: string;
}

const props = withDefaults(defineProps<SeparatorProps>(), {
  orientation: "horizontal",
});
</script>

<template>
  <div
    :class="
      cn(
        'shrink-0',
        orientation === 'horizontal' ? 'h-[1px] w-full' : 'h-full w-[1px]',
        label ? 'relative flex items-center' : 'bg-border',
        props.class,
      )
    "
  >
    <template v-if="label">
      <span class="bg-border h-[1px] flex-1" />
      <span class="px-2 text-xs text-muted-foreground">{{ label }}</span>
      <span class="bg-border h-[1px] flex-1" />
    </template>
  </div>
</template>
