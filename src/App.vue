<script setup lang="ts">
  import AppSelect from './components/common/AppSelect.vue'
  import { ref } from 'vue'

  const countries = [
    { label: 'Vietnam', value: 'vn' },
    { label: 'China', value: 'cn' },
    { label: 'United State of American', value: 'usa', disabled: true },
    { label: 'Nguyen Van Hung', value: 'nvh' },
    { label: 'Home Sweet Home', value: 'home' },
  ]

  const selected = ref(null)
</script>

<template>
  <div class="wrap">
    <AppSelect v-model="selected" :options="countries" :show-search="true" :allow-clear="true" />
    <AppSelect v-model="selected" :options="countries" :show-search="true" :multiple="true" />
    <AppSelect v-model="selected" :options="countries" />
    <AppSelect v-model="selected" :options="countries" />
    <AppSelect v-model="selected" :options="countries" />
  </div>
</template>

<style scoped>
  .wrap {
    display: flex;
    gap: 20px;
  }
</style>
