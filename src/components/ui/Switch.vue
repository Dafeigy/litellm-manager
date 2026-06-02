<script setup lang="ts">
import type { HTMLAttributes } from "vue";
import { cn } from "@/lib/utils";
import { SwitchRoot, SwitchThumb } from "radix-vue";

const props = withDefaults(defineProps<{
  modelValue?: boolean;
  class?: HTMLAttributes["class"];
  disabled?: boolean;
}>(), {
  modelValue: false,
  disabled: false,
});

const emit = defineEmits<{ "update:modelValue": [value: boolean] }>();
</script>

<template>
  <SwitchRoot
    :checked="props.modelValue"
    @update:checked="emit('update:modelValue', $event)"
    :disabled="props.disabled"
    :class="
      cn(
        'peer inline-flex h-[24px] w-[44px] shrink-0 cursor-pointer items-center rounded-full border-2 border-transparent transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 focus-visible:ring-offset-background disabled:cursor-not-allowed disabled:opacity-50',
        props.modelValue ? 'bg-primary' : 'bg-input',
        props.class,
      )
    "
  >
    <SwitchThumb
      :class="
        cn(
          'pointer-events-none block h-5 w-5 rounded-full bg-background shadow-lg ring-0 transition-transform',
          props.modelValue ? 'translate-x-5' : 'translate-x-0',
        )
      "
    />
  </SwitchRoot>
</template>
