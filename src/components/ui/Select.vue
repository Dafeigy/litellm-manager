<script setup lang="ts">
import type { HTMLAttributes } from "vue";
import { cn } from "@/lib/utils";
import {
  SelectContent,
  SelectRoot,
  SelectTrigger,
  SelectValue,
} from "radix-vue";
import { ChevronDown } from "lucide-vue-next";

const props = withDefaults(defineProps<{
  modelValue?: string;
  placeholder?: string;
  class?: HTMLAttributes["class"];
  disabled?: boolean;
}>(), {
  placeholder: "请选择",
  disabled: false,
});

const emit = defineEmits<{ "update:modelValue": [value: string] }>();
</script>

<template>
  <SelectRoot :model-value="props.modelValue" @update:model-value="emit('update:modelValue', $event)" :disabled="props.disabled">
    <SelectTrigger
      :class="
        cn(
          'flex h-10 w-full items-center justify-between rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 [&>span]:line-clamp-1',
          props.class,
        )
      "
    >
      <SelectValue :placeholder="props.placeholder" />
      <ChevronDown class="h-4 w-4 opacity-50" />
    </SelectTrigger>
    <SelectContent
      class="relative z-50 max-h-96 min-w-[8rem] overflow-hidden rounded-md border bg-popover text-popover-foreground shadow-md"
    >
      <slot />
    </SelectContent>
  </SelectRoot>
</template>
