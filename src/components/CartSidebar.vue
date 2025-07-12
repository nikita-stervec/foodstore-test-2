<template>
  <div v-if="show" class="cart-sidebar" @click.stop>
    <div class="cart-dropdown-content">
      <div class="cart-header">
        <h3>Мой заказ:</h3>
        <button class="close-cart" @click="$emit('close')">
          <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
            <path d="M1 1L13 13M13 1L1 13" stroke="currentColor" stroke-width="2" />
          </svg>
        </button>
      </div>
      <div class="cart-content">
        <div class="delivery-options">
          <button
            class="delivery-option"
            :class="{ active: deliveryMethod === 'delivery' }"
            @click.stop="deliveryMethod = 'delivery'"
          >
            Доставка
          </button>
          <button
            class="delivery-option"
            :class="{ active: deliveryMethod === 'pickup' }"
            @click.stop="deliveryMethod = 'pickup'"
          >
            Самовывоз
          </button>
        </div>

        <div v-if="deliveryMethod === 'delivery'" class="address-container">
          <svg class="location-icon" width="20" height="20" viewBox="0 0 24 24" fill="none">
            <path
              d="M12 2C8.13 2 5 5.13 5 9C5 14.25 12 22 12 22C12 22 19 14.25 19 9C19 5.13 15.87 2 12 2ZM12 11.5C10.62 11.5 9.5 10.38 9.5 9C9.5 7.62 10.62 6.5 12 6.5C13.38 6.5 14.5 7.62 14.5 9C14.5 10.38 13.38 11.5 12 11.5Z"
              fill="currentColor"
            />
          </svg>
          <span class="address">ул. Коркыт Ата (аул. Косшы) 18/1</span>
        </div>

        <div class="cart-items-container">
          <div v-for="(item, index) in cartItems" :key="index" class="cart-item">
            <div class="item-image-container">
              <div class="item-image" :style="{ background: item.color }">
                <div class="food-icon">{{ item.icon }}</div>
              </div>
            </div>
            <div class="item-contorl">
              <div class="item-name">{{ item.name }}</div>
              <div class="item-compound">{{ item.description }}</div>
              <div class="item-summary-block">
                <div class="quantity-control">
                  <button class="quantity-btn" @click.stop="$emit('decrease', index)">-</button>
                  <span class="quantity">{{ item.quantity }}</span>
                  <button class="quantity-btn" @click.stop="$emit('increase', index)">+</button>
                </div>
                <div class="item-price">
                  {{ (item.price * item.quantity).toLocaleString('ru-RU') }} ₸
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="promo-container">
          <input
            type="text"
            v-model="promoCode"
            @click.stop
            placeholder="Промокод"
            class="promo-input"
          />
        </div>

        <div class="order-summary">
          <div class="summary-row">
            <span
              >Товары в заказе <span class="order-quantity">{{ totalItemsCount }} шт.</span></span
            >
            <span>{{ totalItemsPrice.toLocaleString('ru-RU') }} ₸</span>
          </div>
          <div v-if="deliveryMethod === 'delivery'" class="summary-row">
            <span>Доставка</span>
            <span>1 000 ₸</span>
          </div>
          <div class="summary-row total">
            <span class="total-text">Итого</span>
            <span class="total-price" :class="{ changing: isTotalChanging }">
              {{ cartTotalPrice.toLocaleString('ru-RU') }} ₸
            </span>
          </div>
        </div>

        <button class="order-button">Заказать</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  cartItems: Array,
})

const emit = defineEmits(['close', 'increase', 'decrease'])

const deliveryMethod = ref('delivery')
const promoCode = ref('')
const isTotalChanging = ref(false)

const totalItemsPrice = computed(() => {
  return props.cartItems.reduce((total, item) => total + item.price * item.quantity, 0)
})

const cartTotalPrice = computed(() => {
  const deliveryFee = deliveryMethod.value === 'delivery' ? 1000 : 0
  return totalItemsPrice.value + deliveryFee
})

const totalItemsCount = computed(() => {
  return props.cartItems.reduce((total, item) => total + item.quantity, 0)
})

watch(cartTotalPrice, () => {
  isTotalChanging.value = true
  setTimeout(() => (isTotalChanging.value = false), 600)
})
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family:
    'NotoSans',
    -apple-system,
    sans-serif;
}

.cart-sidebar {
  margin-top: 85px;
  top: 0;
  right: 0;
  bottom: 0;
  height: 92.5vh;
  width: 375px;
  background: white;
  overflow-y: auto;
}

.cart-dropdown-content {
  padding: 20px;
  height: 100%;
  display: flex;
  flex-direction: column;
  border-left: 1px solid #eee;
}

.cart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.close-cart {
  background: none;
  border: none;
  cursor: pointer;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background-color 0.2s;
}

.close-cart:hover {
  background-color: #f5f5f5;
}

.cart-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
  overflow-y: auto;
  padding-top: 15px;
}

.delivery-options {
  display: flex;
  gap: 10px;
}

.delivery-option {
  flex: 1;
  padding: 12px 0;
  border-radius: 10px;
  border: 1px solid #e0e0e0;
  background: white;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.delivery-option.active {
  background: #f5f5f7;
  border-color: #0071e35d;
  color: #000;
}

.address-container {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: #f9f9f9;
  border-radius: 10px;
  font-size: 0.95rem;
}

.location-icon {
  flex-shrink: 0;
  color: #666;
}

.cart-items-container {
  flex: 1;
  overflow-y: auto;
  padding-right: 8px;
}

.cart-items-container::-webkit-scrollbar {
  width: 6px;
}

.cart-items-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.cart-items-container::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 10px;
}

.cart-items-container::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

.cart-item {
  padding: 16px 0;
  display: flex;
  border-bottom: 1px solid #eee;
}

.item-image-container {
  width: 80px;
  height: 80px;
  border-radius: 10px;
  overflow: hidden;
  flex-shrink: 0;
}

.item-image {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.food-icon {
  font-size: 2rem;
  opacity: 0.8;
}

.item-contorl {
  flex: 1;
  padding-left: 15px;
  display: flex;
  flex-direction: column;
}

.item-name {
  font-weight: 600;
  margin-bottom: 5px;
}

.item-compound {
  color: #888;
  font-size: 0.85rem;
  margin-bottom: 10px;
  flex: 1;
}

.item-summary-block {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.quantity-control {
  display: flex;
  align-items: center;
  gap: 10px;
}

.quantity-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: 1px solid #e0e0e0;
  background: white;
  font-size: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.2s;
}

.quantity-btn:hover {
  background: #f5f5f5;
}

.quantity {
  min-width: 30px;
  text-align: center;
  font-weight: 500;
}

.item-price {
  font-weight: 700;
  font-size: 1.1rem;
}

.promo-container {
  padding: 10px 0;
}

.promo-input {
  width: 100%;
  padding: 12px 15px;
  border: 1px solid #e0e0e0;
  border-radius: 10px;
  font-size: 1rem;
  outline: none;
}

.order-summary {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 15px 6px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
}

.summary-row.total {
  position: relative;
  margin-top: 10px;
  padding-top: 15px;
  border-top: 1px dashed #ccc;
  font-size: 1.1rem;
}

.total-text {
  font-weight: bold;
}

.total-price {
  font-weight: bold;
  font-size: 1.3rem;
}

.order-button {
  width: 100%;
  padding: 16px;
  background: #282828;
  color: white;
  border: none;
  border-radius: 10px;
  font-weight: 600;
  font-size: 1.05rem;
  cursor: pointer;
  transition: background 0.2s;
  margin-top: auto;
}

.order-button:hover {
  background: #1d1d1f;
}

.total-price.changing {
  animation: highlightTotal 0.6s ease;
}

@keyframes highlightTotal {
  0% {
    transform: scale(1);
    color: #000;
  }
  50% {
    transform: scale(1.05);
    color: #e53935;
  }
  100% {
    transform: scale(1);
    color: #000;
  }
}
</style>
