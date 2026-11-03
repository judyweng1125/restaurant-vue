<script setup>
import { ref, computed } from 'vue'
import ProductCard from './components/ProductCard.vue'

// 1. 保留你原本的台灣小吃資料
const mockMenuItems = ref([
  {
    id: 1,
    name: '經典滷肉飯',
    price: 50,
    tag: '人氣招牌',
    image: 'https://tokyo-kitchen.icook.network/uploads/recipe/cover/391516/4a15cb3a501a0c4e.jpg',
    description: '肥瘦相間的嚴選豬肉，慢火熬煮淋上獨門醬汁，在地好滋味。'
  },
  {
    id: 2,
    name: '香酥排骨飯',
    price: 95,
    tag: '經典美味',
    image: 'https://img.shoplineapp.com/media/image_clips/65d7103b092a780017684f7a/original.jpg',
    description: '現炸大塊排骨，外皮金黃酥脆內層多汁，搭配三樣每日配菜。'
  },
  {
    id: 3,
    name: '黃金炸豆腐',
    price: 45,
    tag: '必點小菜',
    image: 'https://tokyo-kitchen.icook.network/uploads/recipe/cover/178791/6af55680021a192f.jpg',
    description: '外酥內嫩的現炸雞蛋豆腐，撒上特製胡椒鹽，必點招牌小菜。'
  }
])

// 2. 狀態管理
const cart = ref([])

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + (item.finalPrice * item.quantity), 0)
})

const cartTotalQuantity = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0)
})

// 3. 客製化選項視窗狀態
const showOptionModal = ref(false)
const currentProduct = ref(null)
const selectedSize = ref('正常')
const selectedSpicy = ref('不辣')
const remarkText = ref('')

function handleSelect(item) {
  currentProduct.value = item
  // 重置預設選項
  selectedSize.value = '正常'
  selectedSpicy.value = '不辣'
  remarkText.value = ''
  showOptionModal.value = true
}

const confirmAddToCart = () => {
  const extraPrice = selectedSize.value.includes('大碗') ? 20 : 0
  const finalPrice = currentProduct.value.price + extraPrice

  cart.value.push({
    id: Date.now(),
    name: currentProduct.value.name,
    size: selectedSize.value,
    spicy: selectedSpicy.value,
    remark: remarkText.value,
    finalPrice: finalPrice,
    quantity: 1 // 加入購物車時預設數量為 1
  })

  showOptionModal.value = false
}

// 4. 購物車操作
const showCartModal = ref(false)

const increaseQty = (item) => item.quantity++
const decreaseQty = (item) => {
  if (item.quantity > 1) item.quantity--
}
const removeItem = (id) => {
  cart.value = cart.value.filter(item => item.id !== id)
}

// 5. 結帳與成功畫面
const showSuccessModal = ref(false)
const orderNumber = ref('')
const orderTime = ref('')

const submitOrder = () => {
  if (cart.value.length === 0) return
  
  // 產生隨機取餐號與時間
  orderNumber.value = 'A-' + Math.floor(100 + Math.random() * 900)
  const now = new Date()
  orderTime.value = now.toLocaleTimeString('zh-TW', { hour: '2-digit', minute: '2-digit' })
  
  showCartModal.value = false
  showSuccessModal.value = true
}

const completeOrder = () => {
  cart.value = []
  showSuccessModal.value = false
}
</script>

<template>
  <div class="app-layout">
    <!-- 頂部標題區 -->
    <header class="header-section">
      <h1 class="page-title">台灣美食線上點餐</h1>
      <p class="page-subtitle">嚴選在地食材，為您現點現做</p>
    </header>

    <!-- 菜單列表：保留你原本的網格架構 -->
    <main class="menu-grid">
      <ProductCard 
        v-for="item in mockMenuItems" 
        :key="item.id" 
        :item="item" 
        @select="handleSelect" 
      />
    </main>

    <!-- 底部固定結帳按鈕 -->
    <div class="checkout-btn-wrapper">
      <button class="checkout-btn" @click="showCartModal = true">
        🛒 查看購物車 / 結帳 ({{ cartTotalQuantity }} 件)
      </button>
    </div>

    <!-- ================= 彈窗 1：選項規格 (按鈕式) ================= -->
    <div class="modal-overlay" v-if="showOptionModal" @click.self="showOptionModal = false">
      <div class="modal-content option-modal">
        <div class="modal-header">
          <h2>{{ currentProduct.name }} - 規格選擇</h2>
          <button class="close-btn" @click="showOptionModal = false">✕</button>
        </div>

        <div class="options-body">
          <!-- 份量 -->
          <div class="option-group">
            <label>份量大小</label>
            <div class="button-group">
              <button :class="{ active: selectedSize === '正常' }" @click="selectedSize = '正常'">正常</button>
              <button :class="{ active: selectedSize === '大碗 (+NT$ 20)' }" @click="selectedSize = '大碗 (+NT$ 20)'">大碗 (+NT$ 20)</button>
            </div>
          </div>

          <!-- 辣度 -->
          <div class="option-group">
            <label>辣度調整</label>
            <div class="button-group grid-3">
              <button :class="{ active: selectedSpicy === '不辣' }" @click="selectedSpicy = '不辣'">不辣</button>
              <button :class="{ active: selectedSpicy === '微辣' }" @click="selectedSpicy = '微辣'">微辣</button>
              <button :class="{ active: selectedSpicy === '大辣' }" @click="selectedSpicy = '大辣'">大辣</button>
            </div>
          </div>

          <!-- 備註 -->
          <div class="option-group">
            <label>特殊需求備註</label>
            <input type="text" v-model="remarkText" class="remark-input" placeholder="例如：飯少一點、不要滷汁...">
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-cancel" @click="showOptionModal = false">取消</button>
          <button class="btn-submit" @click="confirmAddToCart">加入購物車</button>
        </div>
      </div>
    </div>

    <!-- ================= 彈窗 2：購物車清單 (加減數量版) ================= -->
    <div class="modal-overlay" v-if="showCartModal" @click.self="showCartModal = false">
      <div class="modal-content cart-modal">
        <div class="modal-header">
          <h2>您的點餐清單</h2>
          <button class="close-btn" @click="showCartModal = false">✕</button>
        </div>

        <div v-if="cart.length === 0" class="empty-state">
          <p>購物車空空的，去點些好吃的吧！</p>
        </div>

        <div class="cart-items-wrapper" v-else>
          <div class="cart-item-card" v-for="item in cart" :key="item.id">
            <div class="item-header">
              <h3>{{ item.name }}</h3>
              <button class="btn-delete" @click="removeItem(item.id)">🗑️ 刪除</button>
            </div>
            <p class="item-spec">{{ item.size.split(' ')[0] }} / {{ item.spicy }}</p>
            <p class="item-remark" v-if="item.remark">備註：{{ item.remark }}</p>
            
            <div class="item-bottom">
              <span class="item-price">NT$ {{ item.finalPrice }}</span>
              <div class="qty-control">
                <button @click="decreaseQty(item)">-</button>
                <span>{{ item.quantity }}</span>
                <button @click="increaseQty(item)">+</button>
              </div>
            </div>
          </div>
        </div>

        <div class="cart-footer" v-if="cart.length > 0">
          <div class="cart-summary">
            <span>總金額</span>
            <span class="total-price">NT$ {{ cartTotal }}</span>
          </div>
          <button class="btn-checkout" @click="submitOrder">確認送出訂單</button>
        </div>
      </div>
    </div>

    <!-- ================= 彈窗 3：下單成功 ================= -->
    <div class="modal-overlay" v-if="showSuccessModal">
      <div class="modal-content success-modal">
        <div class="success-icon">✓</div>
        <h2>下單成功！</h2>
        <p class="success-desc">廚房已收到您的點餐，請稍後片刻</p>
        
        <div class="order-ticket">
          <span class="ticket-label">取餐號碼</span>
          <span class="ticket-number">{{ orderNumber }}</span>
        </div>

        <div class="order-details">
          <div class="detail-header">
            <span>餐點明細</span>
            <span>時間：{{ orderTime }}</span>
          </div>
          <div class="detail-row">
            <span>{{ cart[0]?.name }} 等 {{ cartTotalQuantity }} 項餐點</span>
            <span>NT$ {{ cartTotal }}</span>
          </div>
          <div class="detail-total">
            <span>合計</span>
            <span>NT$ {{ cartTotal }}</span>
          </div>
        </div>

        <button class="btn-complete" @click="completeOrder">完成 / 回到菜單</button>
      </div>
    </div>

  </div>
</template>

<style>
/* 全域設定 */
body {
  margin: 0; padding: 0;
  background-color: #f8fafc;
  font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif;
  color: #333;
}
/* 捲軸美化 */
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-thumb { background: #ddd; border-radius: 10px; }
</style>

<style scoped>
.app-layout {
  max-width: 1050px;
  margin: 0 auto;
  padding: 40px 20px 120px 20px;
}

.header-section {
  text-align: center;
  margin-bottom: 40px;
  animation: fadeInDown 0.6s ease-out backwards;
}

.page-title {
  margin: 0 0 10px 0;
  color: #1e293b;
  font-size: 2.5rem;
  font-weight: 900;
  letter-spacing: 2px;
}

.page-subtitle {
  margin: 0;
  color: #64748b;
  font-size: 1.1rem;
  letter-spacing: 1px;
}

/* 菜單網格：保留原本架構 */
.menu-grid {
  display: flex;
  gap: 28px;
  flex-wrap: wrap;
  justify-content: center;
}

/* 底部購物車按鈕 */
.checkout-btn-wrapper {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  width: calc(100% - 40px);
  max-width: 480px;
  z-index: 100;
}

.checkout-btn {
  width: 100%;
  background-color: rgba(234, 88, 12, 0.95);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 18px;
  border-radius: 40px;
  font-size: 1.15rem;
  font-weight: 800;
  box-shadow: 0 8px 24px rgba(234, 88, 12, 0.4);
  cursor: pointer;
  transition: all 0.2s ease;
}
.checkout-btn:hover { background-color: rgba(217, 70, 10, 0.98); transform: translateY(-2px); }
.checkout-btn:active { transform: translateY(2px); }

/* ================= 彈窗共用 ================= */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex; justify-content: center; align-items: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
  animation: fadeIn 0.2s;
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

.modal-content {
  background: white;
  border-radius: 24px;
  width: 90%;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 40px rgba(0,0,0,0.15);
  animation: slideUp 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}
@keyframes slideUp { from { transform: translateY(50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }

.modal-header {
  padding: 24px;
  display: flex; justify-content: space-between; align-items: center;
  border-bottom: 1px solid #f0f0f0;
}
.modal-header h2 { margin: 0; font-size: 1.4rem; color: #1e293b; }
.close-btn { background: none; border: none; font-size: 1.5rem; color: #999; cursor: pointer; }

.modal-footer {
  padding: 20px 24px;
  display: flex; gap: 12px;
  border-top: 1px solid #f0f0f0;
  background: white;
  border-bottom-left-radius: 24px;
  border-bottom-right-radius: 24px;
}
.btn-cancel, .btn-submit, .btn-checkout, .btn-complete {
  padding: 16px;
  border-radius: 12px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  border: none;
  transition: opacity 0.2s;
}
.btn-cancel { flex: 1; background: #f1f5f9; color: #64748b; }
.btn-submit { flex: 2; background: #e63946; color: white; }
.btn-submit:hover, .btn-checkout:hover, .btn-complete:hover { opacity: 0.9; }

/* ================= 規格選擇 ================= */
.option-modal { max-width: 500px; }
.options-body { padding: 24px; overflow-y: auto; display: flex; flex-direction: column; gap: 24px; }
.option-group { display: flex; flex-direction: column; gap: 12px; text-align: center; }
.option-group label { font-weight: bold; color: #475569; }

.button-group { display: flex; gap: 12px; }
.button-group.grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); }
.button-group button {
  flex: 1;
  padding: 14px;
  border: 2px solid transparent;
  background: #f8f9fa;
  border-radius: 12px;
  font-size: 1rem;
  color: #64748b;
  cursor: pointer;
  transition: all 0.2s;
}
.button-group button.active {
  background: #fd7e14;
  color: white;
  border-color: #fd7e14;
  font-weight: bold;
  box-shadow: 0 4px 10px rgba(253, 126, 20, 0.2);
}

.remark-input {
  padding: 16px;
  border: 1px solid #cbd5e1;
  border-radius: 12px;
  font-size: 1rem;
  background: #fafafa;
  outline: none;
}
.remark-input:focus { border-color: #fd7e14; background: white; }

/* ================= 購物車 ================= */
.cart-modal { max-width: 600px; }
.cart-items-wrapper { padding: 24px; overflow-y: auto; background: #fafafa; }
.empty-state { padding: 60px 0; text-align: center; color: #94a3b8; font-weight: bold; }

.cart-item-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.04);
}
.item-header { display: flex; justify-content: space-between; align-items: flex-start; }
.item-header h3 { margin: 0 0 8px 0; font-size: 1.2rem; color: #334155; }
.btn-delete { background: none; border: none; color: #adb5bd; cursor: pointer; font-size: 0.9rem; }
.item-spec { margin: 0; color: #94a3b8; font-size: 0.95rem; }
.item-remark { margin: 8px 0 0 0; color: #fd7e14; font-size: 0.9rem; }

.item-bottom {
  display: flex; justify-content: space-between; align-items: center;
  margin-top: 16px; padding-top: 16px; border-top: 1px solid #f1f5f9;
}
.item-price { font-size: 1.3rem; color: #e63946; font-weight: bold; }

.qty-control { display: flex; align-items: center; border: 1px solid #e2e8f0; border-radius: 30px; overflow: hidden; }
.qty-control button { width: 36px; height: 36px; border: none; background: white; color: #e63946; font-size: 1.2rem; cursor: pointer; }
.qty-control button:active { background: #f8f9fa; }
.qty-control span { width: 40px; text-align: center; font-weight: bold; }

.cart-footer { padding: 24px; background: white; border-top: 1px solid #eee; border-radius: 0 0 24px 24px; }
.cart-summary { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; font-weight: bold; font-size: 1.2rem; color: #64748b; }
.total-price { color: #e63946; font-size: 1.8rem; font-weight: 900; }
.btn-checkout { width: 100%; background: #e63946; color: white; padding: 18px; border-radius: 12px; font-size: 1.2rem; }

/* ================= 成功畫面 ================= */
.success-modal { max-width: 400px; padding: 40px 30px; align-items: center; text-align: center; }
.success-icon { width: 80px; height: 80px; background: #e8f5e9; color: #4caf50; font-size: 40px; display: flex; justify-content: center; align-items: center; border-radius: 50%; margin-bottom: 20px; }
.success-modal h2 { margin: 0 0 10px 0; font-size: 1.8rem; color: #1e293b; }
.success-desc { color: #64748b; margin: 0 0 30px 0; }

.order-ticket { background: #fff8f0; width: 100%; padding: 20px; border-radius: 16px; border: 1px solid #ffe8cc; margin-bottom: 24px; display: flex; flex-direction: column; gap: 8px; }
.ticket-label { color: #fd7e14; font-weight: bold; }
.ticket-number { font-size: 2.5rem; color: #e67e22; font-weight: 900; }

.order-details { width: 100%; background: #f8f9fa; padding: 20px; border-radius: 12px; margin-bottom: 30px; text-align: left; }
.detail-header { display: flex; justify-content: space-between; color: #475569; font-weight: bold; margin-bottom: 16px; font-size: 0.95rem; }
.detail-row { display: flex; justify-content: space-between; color: #64748b; margin-bottom: 12px; }
.detail-total { display: flex; justify-content: space-between; color: #e63946; font-weight: bold; padding-top: 12px; border-top: 1px dashed #cbd5e1; margin-top: 12px; }

.btn-complete { width: 100%; background: #e63946; color: white; padding: 16px; border-radius: 12px; font-size: 1.1rem; }
</style>