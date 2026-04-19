<script setup lang="ts">
defineProps<{
  tabs: Array<{ key: string; label: string; path: string; icon: string }>
  activeTab: string
}>()
</script>

<template>
  <nav class="tabs">
    <RouterLink
      v-for="tab in tabs"
      :key="tab.key"
      :to="tab.path"
      class="tab"
      :class="{ on: activeTab === tab.key }"
    >
      <span class="tab-ico">{{ tab.icon }}</span>
      <span class="tab-lbl">{{ tab.label }}</span>
    </RouterLink>
  </nav>
</template>
