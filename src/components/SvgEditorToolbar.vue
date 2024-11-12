<template>
  <div class="p-2 flex flex-col gap-2 shadow-2xl">
    <Button
      icon="pi pi-arrow-up-right"
      :variant="mode === 'selection' ? undefined : 'outlined'"
      @click="mode = 'selection'"
    />
    <Button icon="pi pi-stop" :variant="mode === 'rectangle' ? undefined : 'outlined'" @click="mode = 'rectangle'" />
    <Button icon="pi pi-circle" :variant="mode === 'circle' ? undefined : 'outlined'" @click="mode = 'circle'" />
  </div>
</template>

<script setup lang="ts">
  import Button from "primevue/button";

  const mode = defineModel<string>();
</script>
