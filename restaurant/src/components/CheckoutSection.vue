<script setup>
defineProps({
  title: String,
  content: String
});
</script>

<template>
  <section class="checkout-section">
    <h2>{{ title }}</h2>
    <div class="section-content" v-html="content"></div>
  </section>
</template>

<style scoped>
.checkout-section { background: #f8f9fa; padding: 15px; border-radius: 8px; margin-bottom: 25px; border: 1px solid #eee; }
.checkout-section h2 { margin-top: 0; font-size: 1.2rem; border-bottom: 2px solid #ddd; padding-bottom: 10px; color: #333; }
:deep(ul) { padding-left: 20px; color: #555; line-height: 1.8; }
:deep(p) { margin: 10px 0; font-size: 1.1rem; }
</style>