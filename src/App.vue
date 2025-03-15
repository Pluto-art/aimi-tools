<script setup>
import DataGenerator from './components/DataGenerator.vue'
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <div class="app-logo">
        <img src="/data-generator-icon.svg" alt="测试数据生成工具" width="40" height="40">
        <h1>测试数据生成工具</h1>
      </div>
    </header>
    <main class="app-content">
      <DataGenerator />
    </main>
  </div>
</template>

<style>
.app-container {
  max-width: 1600px;
  margin: 0 auto;
  padding: 1rem;
}

.app-header {
  margin-bottom: 2rem;
}

.app-logo {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.app-logo h1 {
  margin: 0;
  font-size: 1.8rem;
  color: #42b883;
}
</style>
