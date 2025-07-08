<template>
  <div class="app-container" :class="{ 'cart-open': showCartDropdown }" @click="closeAllDropdowns">
    <header class="header" :class="{ scrolled: isScrolled }">
      <div class="header-container">
        <div class="left-controls">
          <button
            class="nav-button"
            :class="{ active: activeIcon === 'menu' }"
            @click="setActiveIcon('menu')"
          >
            <MenuIcon :class="{ 'icon-active': activeIcon === 'menu' }" />
          </button>

          <div class="lang-wrapper" @click.stop>
            <button class="lang-button" @click="toggleLangDropdown" ref="langButton">
              <span class="lang-text">{{ currentLang }}</span>
            </button>

            <transition name="slide-down">
              <div v-if="showLangDropdown" class="lang-dropdown">
                <div
                  v-for="lang in languages"
                  :key="lang"
                  class="lang-option"
                  :class="{ active: currentLang === lang }"
                  @click="selectLang(lang)"
                >
                  {{ lang }}
                </div>
              </div>
            </transition>
          </div>

          <div class="search-container" @click.stop>
            <button
              class="nav-button search-button"
              :class="{ active: showSearch }"
              @click="toggleSearch"
            >
              <SearchIcon :class="{ 'icon-active': showSearch }" />
            </button>

            <transition name="slide-fade">
              <div v-if="showSearch" class="search-box">
                <input
                  type="text"
                  placeholder="Поиск..."
                  v-model="searchQuery"
                  ref="searchInput"
                  @click.stop
                />
                <button class="search-submit">
                  <SearchIcon />
                </button>
              </div>
            </transition>
          </div>
        </div>

        <div class="right-controls">
          <button class="cart-button" :class="{ active: isCartOpen }" @click.stop="toggleCart">
            <CartIcon class="cart-icon" />
            <span class="balance" :class="{ animating: isAnimating }">
              {{ formattedBalance }}
            </span>
          </button>
        </div>
      </div>
    </header>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue'
import CartIcon from './icons/CartIcon.vue'
import MenuIcon from './icons/MenuIcon.vue'
import SearchIcon from './icons/SearchIcon.vue'

const emit = defineEmits(['toggle-cart'])
const props = defineProps({
  totalPrice: { type: Number, default: 0 },
  isCartOpen: Boolean,
})

const animatedBalance = ref(0)
const animationStart = ref(0)
const animationDuration = 300
const animationFrameId = ref(null)
const activeIcon = ref(null)
const isAnimating = ref(false)
const isScrolled = ref(false)
const showLangDropdown = ref(false)
const showSearch = ref(false)
const searchQuery = ref('')
const currentLang = ref('RU')
const langButton = ref(null)
const searchInput = ref(null)
const langButtonWidth = ref(50)

const languages = ['RU', 'KZ', 'EN']

const formattedBalance = computed(
  () => Math.floor(animatedBalance.value).toLocaleString('ru-RU') + ' ₸',
)

const startCounterAnimation = () => {
  if (animationFrameId.value) cancelAnimationFrame(animationFrameId.value)

  const startValue = animatedBalance.value
  const endValue = props.totalPrice
  animationStart.value = performance.now()

  const animate = (timestamp) => {
    const elapsed = timestamp - animationStart.value
    const progress = Math.min(elapsed / animationDuration, 1)
    animatedBalance.value = startValue + (endValue - startValue) * progress

    if (progress < 1) {
      animationFrameId.value = requestAnimationFrame(animate)
    } else {
      animatedBalance.value = endValue
      animationFrameId.value = null
    }
  }

  isAnimating.value = true
  animationFrameId.value = requestAnimationFrame(animate)
  setTimeout(() => (isAnimating.value = false), 300)
}

watch(() => props.totalPrice, startCounterAnimation)

const closeDropdowns = () => {
  showLangDropdown.value = false
  showSearch.value = false
}

const closeOtherDropdowns = (current) => {
  if (current !== 'lang') showLangDropdown.value = false
  if (current !== 'search') showSearch.value = false
}

const closeAllDropdowns = () => {
  showLangDropdown.value = false
  showSearch.value = false
}

const setActiveIcon = (iconName) => {
  activeIcon.value = activeIcon.value === iconName ? null : iconName
}

const toggleCart = (e) => {
  e.stopPropagation()
  closeOtherDropdowns('cart')
  emit('toggle-cart')
}

const toggleLangDropdown = (e) => {
  e.stopPropagation()
  closeOtherDropdowns('lang')
  showLangDropdown.value = !showLangDropdown.value
}

const selectLang = (lang) => {
  currentLang.value = lang
  showLangDropdown.value = false
}

const toggleSearch = (e) => {
  e.stopPropagation()
  closeOtherDropdowns('search')
  showSearch.value = !showSearch.value
  if (showSearch.value) {
    nextTick(() => searchInput.value.focus())
  }
}

const handleScroll = () => {
  isScrolled.value = window.scrollY > 200
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
  startCounterAnimation()

  nextTick(() => {
    if (langButton.value) {
      langButtonWidth.value = langButton.value.offsetWidth
    }
  })
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll)
  if (animationFrameId.value) cancelAnimationFrame(animationFrameId.value)
})

defineExpose({
  closeDropdowns,
})
</script>

<style scoped>
:root {
  --bg-primary: #ffffff;
  --button-bg: #f9f9f9;
  --accent-color: #0071e3;
  --contrast-color: #0071e35d;
  --active-color: #282828;
  --text-primary: #1d1d1f;
  --border-radius: 10px;
  --transition-duration: 0.3s;
}

* {
  transition: 0.3s ease;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family:
    'NotoSans',
    -apple-system,
    sans-serif;
}

.app-container {
  position: relative;
  overflow-x: hidden;
  transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1);
}

.header {
  background-color: var(--bg-primary);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  transition: transform 0.4s ease;
  padding: 0;
  height: 85px;
  border-color: #eee;
}

.header.scrolled {
  border-bottom: 1px solid #eee;
  height: 85px;
  transition: 0.4s ease;
}

.header-container {
  max-width: 1140px;
  height: 85px;
  transition: 0.4s ease;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0 auto;
  padding: 24px 0;
  transition: padding 0.4s ease;
}

.left-controls {
  display: flex;
  align-items: center;
  gap: 28px;
  flex: 1;
}

.right-controls {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  flex: 1;
}

.search-container {
  position: relative;
}

.search-box {
  border: 1px solid #eee;
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 8px;
  display: flex;
  width: 320px;
  background: white;
  border-radius: 10px;
  z-index: 200;
}

.search-box input {
  flex: 1;
  padding: 12px 16px;
  border: none;
  border-radius: 8px 0 0 8px;
  outline: none;
  font-size: 0.95rem;
}

.search-submit {
  background: #0071e35d;
  border: none;
  padding: 0 16px;
  border-radius: 0 8px 8px 0;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: background 0.2s;
}

.search-submit:hover {
  background: #e0e0e0;
}

.search-submit svg {
  width: 18px;
  height: 18px;
}

.nav-button {
  background-color: var(--button-bg);
  border: none;
  width: 50px;
  height: 50px;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all var(--transition-duration) ease;
}

.nav-button:hover,
.nav-button.active {
  background-color: #e0e0e0;
}

.nav-button svg {
  width: 20px;
  height: 20px;
  transition: fill var(--transition-duration) ease;
}

.icon-active {
  fill: var(--accent-color);
}

.lang-wrapper {
  position: relative;
  display: inline-block;
}

.lang-button {
  background-color: var(--button-bg);
  border: none;
  width: 50px;
  height: 50px;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all var(--transition-duration) ease;
  font-weight: 600;
  font-size: 0.9rem;
}

.lang-button:hover {
  background-color: #e0e0e0;
}

.lang-dropdown {
  width: 56px;
  border: 1px solid #eee;
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  background: white;
  border-radius: 10px;
  overflow: hidden;
  z-index: 200;
}

.lang-option {
  padding: 12px 16px;
  cursor: pointer;
  transition: background-color 0.2s;
  font-size: 0.9rem;
  text-align: center;
}

.lang-option:hover {
  background-color: #f5f5f7;
}

.lang-option.active {
  color: var(--accent-color);
  font-weight: 500;
}

.item-image-container {
  width: 112px;
  height: 112px;
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
  font-size: 3rem;
  opacity: 0.8;
}

.cart-items-container {
  max-height: 400px;
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

.cart-button {
  background-color: var(--button-bg);
  border: none;
  height: 50px;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  padding: 0 14px;
  gap: 8px;
  transition: all var(--transition-duration) ease;
  color: var(--text-primary);
}

.cart-button:hover {
  background-color: #e0e0e0;
}

.cart-icon {
  width: 20px;
  height: 20px;
  color: inherit;
}

.cart-button.active {
  background-color: #282828 !important;
  color: #ffffff !important;
}

.cart-button.active .cart-icon {
  color: inherit;
}

.cart-button.active .balance {
  color: #ffffff !important;
}

.balance {
  font-weight: 500;
  font-size: 16px;
  transition: transform 0.1s ease-out;
}

.balance {
  font-weight: 500;
  font-size: 16px;
  transition: transform 0.3s ease;
  font-variant-numeric: tabular-nums;
}

@keyframes counterTick {
  0% {
    transform: translateY(-2px);
    opacity: 0.8;
  }
  50% {
    transform: translateY(0);
    opacity: 1;
  }
  100% {
    transform: translateY(2px);
    opacity: 0.9;
  }
}

.animating {
  animation: counterTick 0.1s infinite;
  color: #4caf50;
  font-weight: 700;
}

main {
  transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1);
  display: flex;
  justify-content: center;
  margin: 0;
}

.cart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
}

.cart-header h3 {
  font-weight: 600;
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

.empty-cart {
  padding: 24px 0;
  text-align: center;
  color: #888;
}

.empty-cart svg {
  margin-bottom: 16px;
  color: #ccc;
}

.empty-cart p {
  font-size: 1.1rem;
  font-weight: 500;
}

.cart-content {
  padding: 0 20px 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.delivery-options {
  display: flex;
  gap: 10px;
  margin-top: 4px;
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
  border-color: var(--contrast-color);
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

.cart-item {
  padding: 16px 0;
  display: flex;
  flex-direction: row;
}

.item-image-container {
  width: 112px;
  height: 112px;
  border-radius: 10px;
  overflow: hidden;
  flex-shrink: 0;
}

.item-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-contorl {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
}

.item-summary-block {
  gap: 30px;
  display: flex;
  flex-direction: row-reverse;
  justify-content: center;
  align-items: center;
}

.item-name {
  display: flex;
  text-align: center;
  font-weight: 600;
  font-size: 18px;
  padding-bottom: 4px;
}

.item-compound {
  font-size: 12px;
  text-align: center;
  padding: 0 24px;
  color: #888888;
}

.item-price {
  font-size: 14px;
  font-weight: 700;
  color: #1d1d1f;
}

.quantity-control {
  display: flex;
  align-items: center;
}

.quantity-btn {
  width: 24px;
  height: 24px;
  border-radius: 50;
  border: 0px solid #e0e0e0;
  background: white;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.2s;
}

.quantity {
  min-width: 30px;
  text-align: center;
  font-weight: 500;
}

.promo-input {
  width: 100%;
  border: 0px;
  font-size: 1rem;
}

.promo-input:focus {
  outline: none;
}

.order-summary {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
}

.summary-row.total {
  position: relative;
  z-index: 1;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px dashed #ccc;
}

.total-text {
  font-size: 20px;
  font-weight: bold;
}

.total-price {
  transition: 0.3s ease;
  display: inline-block;
  transform: rotate(0deg);
  transform-origin: center;
  background: linear-gradient(to right, #fff5e6, #ffebcc);
  padding: 5px 10px;
  border-radius: 10px;
}

.total-price {
  background: linear-gradient(to left, #fff5e6, #ffebcc);
}

@keyframes highlightTotal {
  0% {
    transition: 0.3s ease;
    transform: rotate(3deg) scale(1);
  }
  50% {
    transition: 0.3s ease;
    transform: rotate(3deg) scale(1.1);
  }
  100% {
    transition: 0.3s ease;
    transform: rotate(3deg) scale(1);
  }
}

.total-price {
  transition: all 0.3s ease;
}

.total-price.changing {
  animation: highlightTotal 0.6s ease;
}

.order-quantity {
  color: #acacac;
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
  margin-top: 10px;
}

.order-button:hover {
  background: #1d1d1f;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1);
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition:
    transform 0.3s cubic-bezier(0.23, 1, 0.32, 1),
    opacity 0.3s ease;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(0.165, 0.84, 0.44, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@media (min-width: 1600px) {
  .header-container {
    transition: 0.4s ease;
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    min-width: 1300px;
    max-width: 1300px;
  }

  .header.scrolled .header-container {
    transition: 0.4s ease;
    padding: 18px 24px;
    max-width: 1548px;
  }
}

@media (max-width: 1139px) {
  .header-container {
    padding: 0 24px;
  }

  .nav-button,
  .lang-button,
  .search-button {
    width: 44px;
    height: 44px;
  }

  .cart-button {
    height: 44px;
    padding: 0 12px;
  }

  .balance {
    font-size: 0.85rem;
  }

  main {
    margin-top: 76px;
  }
}

@media (max-width: 768px) {
  .header-container {
    padding: 0 16px;
  }

  .left-controls {
    gap: 16px;
  }

  .lang-wrapper {
    display: none;
  }

  .nav-button,
  .lang-button,
  .search-button {
    width: 44px;
    height: 44px;
  }

  .cart-button {
    height: 40px;
  }

  .balance {
    display: none;
  }

  .search-box {
    width: 280px;
    right: 0;
    left: auto;
  }

  main {
    margin-top: 72px;
    padding: 16px;
  }
}

@media (max-width: 480px) {
  .header-container {
    padding: 12px 20px;
  }

  .left-controls {
    gap: 12px;
  }

  .search-button {
    display: none;
  }

  .cart-sidebar {
    width: 100%;
  }

  .cart-open .header,
  .cart-open main {
    transform: none;
  }

  .nav-button,
  .lang-button,
  .search-button {
    width: 36px;
    height: 36px;
  }

  .cart-button {
    height: 36px;
  }

  main {
    margin-top: 60px;
    padding: 12px;
  }
}

@media (max-width: 360px) {
  .header-container {
    padding: 10px 12px;
  }

  .nav-button {
    width: 34px;
    height: 34px;
  }

  .cart-button {
    padding: 0 8px;
  }
}
</style>
