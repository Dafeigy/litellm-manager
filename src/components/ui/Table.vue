<script setup lang="ts">
import { cn } from "@/lib/utils";
</script>

<template>
  <div :class="cn('w-full overflow-auto', $attrs.class as string)">
    <table :class="cn('w-full caption-bottom text-sm')">
      <slot />
    </table>
  </div>
</template>
