<script setup>
// 接收父元件傳來的整包 item 物件
defineProps({
  item: {
    type: Object,
    required: true
  }
})

// 定義點擊選購時觸發的事件
defineEmits(['select'])
</script>

<template>
  <div class="product-card">
    <div class="image-wrapper">
      <!-- 若該餐點有 tag 屬性，就顯示左上角小標籤 -->
      <span v-if="item.tag" class="tag">{{ item.tag }}</span>
      <img :src="item.image" :alt="item.name" class="product-image" />
    </div>
    
    <div class="card-content">
      <h3 class="title">{{ item.name }}</h3>
      <p class="desc">{{ item.description }}</p>
      
      <div class="card-footer">
        <span class="price">NT$ {{ item.price }}</span>
        <!-- 點擊時觸發 select 事件，並把這項餐點的資料傳回去 -->
        <button class="add-btn" @click="$emit('select', item)">
          ＋ 選購
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 卡片進場動畫 */
@keyframes cardFadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.product-card {
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.06);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  width: 100%;
  max-width: 310px;
  transition: transform 0.3s cubic-bezier(0.25, 0.8, 0.25, 1), box-shadow 0.3s ease;
  animation: cardFadeIn 0.6s ease-out backwards;
}

.product-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.12);
}

.image-wrapper {
  position: relative;
  width: 100%;
  height: 220px;
  overflow: hidden;
}

.tag {
  position: absolute;
  top: 14px;
  left: 14px;
  background-color: #ff5252;
  color: white;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 900;
  z-index: 10;
  box-shadow: 0 4px 8px rgba(255, 82, 82, 0.3);
  letter-spacing: 1px;
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.product-card:hover .product-image {
  transform: scale(1.08);
}

.card-content {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  flex-grow: 1;
}

.title {
  margin: 0;
  font-size: 1.35rem;
  color: #1e293b;
  font-weight: 800;
}

.desc {
  margin: 0;
  font-size: 0.95rem;
  color: #64748b;
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-footer {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16px;
  border-top: 1px solid #f1f5f9;
}

.price {
  font-size: 1.4rem;
  font-weight: 900;
  color: #ea580c;
}

.add-btn {
  background-color: #3b82f6;
  color: white;
  border: none;
  padding: 10px 22px;
  border-radius: 30px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
  transition: all 0.2s ease;
  box-shadow: 0 4px 10px rgba(59, 130, 246, 0.3);
}

.add-btn:hover {
  background-color: #2563eb;
  transform: scale(1.05);
}
.add-btn:active {
  transform: scale(0.95);
}
</style>