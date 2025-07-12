<template>
  <div @click="closeDropdowns">
    <AppHeader
      ref="headerRef"
      :totalPrice="cartTotal"
      :isContentShifted="isContentShifted"
      @toggle-cart="toggleCart"
    />
    <div class="content-wrapper">
      <div class="content-flex">
        <transition name="slide">
          <div class="page-container" :class="{ 'cart-open': isContentShifted }">
            <div class="food-content">
              <header class="page-header">
                <div class="header-content">
                  <h1>
                    🍕 ПИЦЦА & СУШИ 🍣
                    <div class="highlight">Доставка за 60 минут</div>
                  </h1>
                  <p class="slogan">Блюда от шеф-повара прямо к вашему столу</p>
                </div>
              </header>
            </div>

            <section class="food-categories">
              <button
                v-for="category in categories"
                :key="category.id"
                class="category-btn"
                :class="{
                  active: activeCategory === category.id,
                  pressed: pressedCategory === category.id,
                }"
                @mousedown="pressedCategory = category.id"
                @mouseup="pressedCategory = null"
                @mouseleave="pressedCategory = null"
                @click="setActiveCategory(category.id)"
              >
                <span class="icon">{{ category.icon }}</span>
                {{ category.name }}
              </button>
            </section>

            <transition-group name="food-list" tag="div" class="food-list">
              <div v-for="item in filteredFood" :key="item.id" class="food-card">
                <div class="food-image" :style="{ background: item.color }">
                  <div class="image-placeholder">{{ item.icon }}</div>
                </div>
                <div class="food-info">
                  <h3>{{ item.name }}</h3>
                  <p class="description">{{ item.description }}</p>
                  <div class="price-add">
                    <span class="price">{{ item.price }} ₸</span>
                  </div>
                </div>
              </div>
            </transition-group>

            <div class="promo-banner">
              <div class="promo-content">
                <div class="promo-icon">🎉</div>
                <div class="promo-text">
                  <h3>АКЦИЯ! Закажите 2 пиццы и получите роллы в подарок</h3>
                  <p>Только до конца месяца</p>
                </div>
              </div>
            </div>
          </div></transition
        >
        <div
          class="cart-animation-container"
          :class="{ open: isCartOpen, closing: isClosing }"
          ref="cartContainer"
        >
          <CartSidebar :show="isCartOpen" :cartItems="cartItems" @close="startCloseAnimation" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'
import AppHeader from './AppHeader.vue'
import CartSidebar from './CartSidebar.vue'

const categories = ref([
  { id: 'all', name: 'Все', icon: '🍽️' },
  { id: 'pizza', name: 'Пицца', icon: '🍕' },
  { id: 'sushi', name: 'Суши', icon: '🍣' },
  { id: 'breakfast', name: 'Завтраки', icon: '🥞' },
  { id: 'main', name: 'Основное', icon: '🍔' },
  { id: 'desserts', name: 'Десерты', icon: '🍰' },
])

const cartItems = ref([
  {
    id: 1,
    name: 'Пепперони',
    description: 'Острая салями, томатный соус, сыр моцарелла',
    price: 599,
    category: 'pizza',
    icon: '🍕',
    color: 'linear-gradient(135deg, #ff9a9e, #fad0c4)',
    quantity: 1,
  },
  {
    id: 4,
    name: 'Филадельфия',
    description: 'Лосось, сливочный сыр, огурец, рис',
    price: 450,
    category: 'sushi',
    icon: '🍣',
    color: 'linear-gradient(135deg, #d4fc79, #96e6a1)',
    quantity: 2,
  },
])

const foodItems = ref([
  {
    id: 1,
    name: 'Пепперони',
    description: 'Острая салями, томатный соус, сыр моцарелла',
    price: 599,
    category: 'pizza',
    icon: '🍕',
    color: 'linear-gradient(135deg, #ff9a9e, #fad0c4)',
  },
  {
    id: 2,
    name: 'Маргарита',
    description: 'Томаты, сыр моцарелла, свежий базилик',
    price: 549,
    category: 'pizza',
    icon: '🍕',
    color: 'linear-gradient(135deg, #a1c4fd, #c2e9fb)',
  },
  {
    id: 3,
    name: '4 Сыра',
    description: 'Моцарелла, пармезан, горгонзола, фета',
    price: 649,
    category: 'pizza',
    icon: '🍕',
    color: 'linear-gradient(135deg, #fbc2eb, #a6c1ee)',
  },
  {
    id: 4,
    name: 'Филадельфия',
    description: 'Лосось, сливочный сыр, огурец, рис',
    price: 450,
    category: 'sushi',
    icon: '🍣',
    color: 'linear-gradient(135deg, #d4fc79, #96e6a1)',
  },
  {
    id: 5,
    name: 'Калифорния',
    description: 'Краб, авокадо, огурец, икра тобико',
    price: 420,
    category: 'sushi',
    icon: '🍣',
    color: 'linear-gradient(135deg, #e0c3fc, #8ec5fc)',
  },
  {
    id: 6,
    name: 'Запеченный ролл с угрем',
    description: 'Угорь, сливочный сыр, соус унаги',
    price: 580,
    category: 'sushi',
    icon: '🍣',
    color: 'linear-gradient(135deg, #f6d365, #fda085)',
  },
  {
    id: 7,
    name: 'Сырники с вареньем',
    description: 'Нежные творожные сырники с вишневым вареньем',
    price: 320,
    category: 'breakfast',
    icon: '🥞',
    color: 'linear-gradient(135deg, #ffecd2, #fcb69f)',
  },
  {
    id: 8,
    name: 'Омлет с овощами',
    description: 'Пышный омлет со свежими овощами и зеленью',
    price: 280,
    category: 'breakfast',
    icon: '🍳',
    color: 'linear-gradient(135deg, #ff9a9e, #fad0c4)',
  },
  {
    id: 9,
    name: 'Бургер классический',
    description: 'Говяжья котлета, сыр, салат, помидор, соус',
    price: 450,
    category: 'main',
    icon: '🍔',
    color: 'linear-gradient(135deg, #f6d365, #fda085)',
  },
  {
    id: 10,
    name: 'Стейк из лосося',
    description: 'Филе лосося с овощами на гриле',
    price: 780,
    category: 'main',
    icon: '🐟',
    color: 'linear-gradient(135deg, #a1c4fd, #c2e9fb)',
  },
  {
    id: 11,
    name: 'Тирамису',
    description: 'Классический итальянский десерт с кофе',
    price: 380,
    category: 'desserts',
    icon: '🍰',
    color: 'linear-gradient(135deg, #e0c3fc, #8ec5fc)',
  },
  {
    id: 12,
    name: 'Чизкейк Нью-Йорк',
    description: 'Нежный чизкейк с ягодным соусом',
    price: 350,
    category: 'desserts',
    icon: '🍮',
    color: 'linear-gradient(135deg, #d4fc79, #96e6a1)',
  },
  {
    id: 13,
    name: 'Панна Котта',
    description: 'Итальянский сливочный десерт с карамелью',
    price: 320,
    category: 'desserts',
    icon: '🍨',
    color: 'linear-gradient(135deg, #fbc2eb, #a6c1ee)',
  },
  {
    id: 14,
    name: 'Мексиканская пицца',
    description: 'Острая говядина, перец халапеньо, кукуруза',
    price: 650,
    category: 'pizza',
    icon: '🍕',
    color: 'linear-gradient(135deg, #ff9a9e, #fad0c4)',
  },
  {
    id: 15,
    name: 'Ролл "Дракон"',
    description: 'Креветка в темпуре, авокадо, соус спайси',
    price: 520,
    category: 'sushi',
    icon: '🍣',
    color: 'linear-gradient(135deg, #a1c4fd, #c2e9fb)',
  },
])

const activeCategory = ref('all')
const pressedCategory = ref(null)
const headerRef = ref(null)
const isContentShifted = ref(false)
const isCartOpen = ref(false)
const isClosing = ref(false)
const cartContainer = ref(null)

const toggleCart = () => {
  if (isCartOpen.value) {
    startCloseAnimation()
  } else {
    isCartOpen.value = true
    isContentShifted.value = true
  }
}
const startCloseAnimation = () => {
  isClosing.value = true
  isContentShifted.value = false
  nextTick(() => {
    cartContainer.value.addEventListener(
      'animationend',
      () => {
        isCartOpen.value = false
        isClosing.value = false
      },
      { once: true },
    )
  })
}
const cartTotal = computed(() => 14500)
const filteredFood = computed(() =>
  activeCategory.value === 'all'
    ? foodItems.value
    : foodItems.value.filter((item) => item.category === activeCategory.value),
)

const setActiveCategory = (id) => (activeCategory.value = id)

const handlePageClick = (event) => {
  if (headerRef.value?.closeDropdowns) {
    headerRef.value.closeDropdowns()
  }
}

onMounted(() => document.addEventListener('click', handlePageClick))
onBeforeUnmount(() => document.removeEventListener('click', handlePageClick))
</script>

<style scoped>
* {
  transition: 0.3s ease;
  box-sizing: border-box;
}

.content-wrapper {
  position: relative;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 0;
}

.content-flex {
  display: flex;
  position: relative;
  width: 100%;
  max-width: 1200px;
  transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1);
}

.page-container {
  flex: 1;
  transition:
    transform 0.4s cubic-bezier(0.23, 1, 0.32, 1),
    padding 0.4s cubic-bezier(0.23, 1, 0.32, 1);
  max-width: 1200px;
  padding: 20px;
  margin: 0 auto;
}

.page-container.cart-open {
  transform: translateX(-15%);
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1);
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}

.cart-animation-container {
  position: absolute;
  top: 0;
  right: 0;
  width: 375px;
  height: 100%;
  transform: translateX(100%);
  z-index: 99;
}

.cart-animation-container.open {
  animation: slide-in 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
}

.cart-animation-container.closing {
  animation: slide-out 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
}

.food-content {
  padding: 20px 0;
  font-family:
    'NotoSans',
    -apple-system,
    sans-serif;
  background-color: #fff;
}

.page-header {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 96px;
  margin-bottom: 30px;
  padding: 20px 0;
  border-radius: 12px;
  background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
  box-shadow: 0 4px 20px rgba(255, 154, 158, 0.3);
  color: #fff;
  transition: transform 0.3s ease;
}

.page-header:hover {
  transform: translateY(-3px);
}

.header-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.header-content h1 {
  display: flex;
  flex-direction: row;
  margin: 0;
  font-size: 2.2rem;
  font-weight: 800;
  transition: transform 0.3s ease;
}

.header-content h1:hover {
  transform: scale(1.02);
}

.highlight {
  background-color: #e53935;
  padding: 5px 10px;
  border-radius: 8px;
  font-size: 1.8rem;
  font-weight: 800;
  transition: transform 0.2s ease;
}

.highlight:hover {
  transform: scale(1.05);
}

.slogan {
  margin-top: 8px;
  font-size: 1.1rem;
  opacity: 0.9;
  font-weight: 400;
}

.food-categories {
  display: flex;
  gap: 12px;
  margin-bottom: 30px;
  flex-wrap: wrap;
  justify-content: center;
}

.category-btn {
  padding: 12px 20px;
  border: none;
  border-radius: 12px;
  background: #f8f9fa;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  font-size: 1.05rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  transition:
    all 0.3s cubic-bezier(0.68, -0.55, 0.27, 1.55),
    transform 0.1s ease;
}

.category-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.category-btn.active,
.category-btn.active:hover {
  background: #ff7043;
  color: white;
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(255, 112, 67, 0.4);
}

.category-btn.pressed {
  transform: scale(0.95) translateY(-1px);
  opacity: 0.9;
}

.icon {
  font-size: 1.4rem;
  transition: transform 0.2s ease;
}

.category-btn:hover .icon {
  transform: scale(1.1);
}

.food-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 30px;
  margin-bottom: 40px;
}

.food-card {
  border: 1px solid #eee;
  border-radius: 16px;
  overflow: hidden;
  background: white;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.03);
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease,
    opacity 0.3s ease;
  transform: scale(0.98);
  opacity: 0.95;
}

.food-card:hover {
  transform: translateY(-8px) scale(1.01);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
  opacity: 1;
}

.food-image {
  height: 180px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.3s ease;
}

.food-card:hover .food-image {
  transform: scale(1.03);
}

.image-placeholder {
  font-size: 5rem;
  opacity: 0.8;
  transition: transform 0.3s ease;
}

.food-card:hover .image-placeholder {
  transform: scale(1.1);
}

.food-info {
  padding: 20px;
}

.food-info h3 {
  margin-top: 0;
  margin-bottom: 12px;
  color: #333;
  font-size: 1.4rem;
  transition: color 0.2s ease;
}

.food-card:hover h3 {
  color: #e53935;
}

.description {
  color: #666;
  font-size: 0.95rem;
  min-height: 60px;
  margin-bottom: 20px;
  line-height: 1.5;
  transition: color 0.2s ease;
}

.food-card:hover .description {
  color: #444;
}

.price-add {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price {
  font-weight: bold;
  font-size: 1.4rem;
  color: #e53935;
  transition: transform 0.2s ease;
}

.food-card:hover .price {
  transform: scale(1.05);
}

.promo-banner {
  background: linear-gradient(135deg, #5ee7df 0%, #b490ca 100%);
  border-radius: 15px;
  padding: 25px;
  margin-top: 30px;
  color: white;
  box-shadow: 0 4px 20px rgba(94, 231, 223, 0.3);
  transition: transform 0.3s ease;
}

.promo-banner:hover {
  transform: translateY(-5px);
}

.promo-content {
  display: flex;
  align-items: center;
  gap: 20px;
}

.promo-icon {
  font-size: 3.5rem;
  transition: transform 0.3s ease;
}

.promo-banner:hover .promo-icon {
  transform: scale(1.1);
}

.promo-text h3 {
  margin: 0 0 8px 0;
  font-size: 1.6rem;
  transition: transform 0.2s ease;
}

.promo-banner:hover h3 {
  transform: translateX(3px);
}

.promo-text p {
  margin: 0;
  font-size: 1.1rem;
  opacity: 0.9;
}

.food-list-move,
.food-list-enter-active,
.food-list-leave-active {
  transition: all 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55);
}

.food-list-enter-from,
.food-list-leave-to {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

.food-list-leave-active {
  position: absolute;
}

@keyframes slide-in {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(50%);
    opacity: 1;
  }
}

@keyframes slide-out {
  0% {
    transform: translateX(50%);
    opacity: 1;
  }
  100% {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@media (min-width: 1140px) and (max-width: 1200px) {
  /* Анимации оставляем без изменений */
  @keyframes slide-in-low {
    0% {
      transform: translateX(-100%);
      opacity: 0;
    }
    100% {
      transform: translateX(-40%);
      opacity: 1;
    }
  }

  @keyframes slide-out-low {
    0% {
      transform: translateX(-40%);
      opacity: 1;
    }
    100% {
      transform: translateX(-100%);
      opacity: 0;
    }
  }

  .header-content h1 {
    align-items: center;
    flex-direction: row;
    margin: 0;
    font-size: 1.5rem;
    font-weight: 500;
    transition: transform 0.3s ease;
  }

  .highlight {
    font-size: 1.5rem;
  }

  .content-flex {
    justify-content: center;
  }

  /* ФИКС: Корректируем позицию корзины */
  .cart-animation-container {
    right: 0; /* Прижимаем к правому краю */
    left: auto; /* Отключаем левое позиционирование */
    width: 25%; /* Оптимальная ширина для этого разрешения */
  }

  .cart-animation-container.open {
    animation: slide-in-low 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
    transform: translateX(-5%); /* Фиксируем конечную позицию */
  }

  .cart-animation-container.closing {
    animation: slide-out-low 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  }

  /* ФИКС: Уменьшаем смещение контента */
  .page-container.cart-open {
    transform: translateX(-25%); /* Снижено с -25% */
    max-width: 730px; /* Оптимальная ширина */
  }
  .page-container {
    margin: auto 72px;
  }
}

@media (min-width: 1201px) and (max-width: 1400px) {
  @keyframes slide-in-lite {
    0% {
      transform: translateX(-100%);
      opacity: 0;
    }
    100% {
      transform: translateX(-1%);
      opacity: 1;
    }
  }
  /* меню в лево */
  @keyframes slide-out-lite {
    0% {
      transform: translateX(-1%);
      opacity: 1;
    }
    100% {
      transform: translateX(-100%);
      opacity: 0;
    }
  }

  .cart-animation-container.open {
    animation: slide-in-lite 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  }

  .cart-animation-container.closing {
    animation: slide-out-lite 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  }

  .page-container.cart-open {
    transform: translateX(-21.75%);
    max-width: 798px;
  }

  .content-flex {
    justify-content: center;
  }

  .page-container {
    margin: auto 64px;
  }
}

@media (min-width: 1401px) and (max-width: 1600px) {
  @keyframes slide-in-lite {
    0% {
      transform: translateX(-100%);
      opacity: 0;
    }
    100% {
      transform: translateX(25%);
      opacity: 1;
    }
  }

  @keyframes slide-out-lite {
    0% {
      transform: translateX(25%);
      opacity: 1;
    }
    100% {
      transform: translateX(-100%);
      opacity: 0;
    }
  }

  .page-container.cart-open {
    transform: translateX(-15%);
    max-width: 943px;
    padding-right: 28px;
  }

  .content-flex {
    justify-content: center;
  }

  .cart-animation-container.open {
    animation: slide-in-lite 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  }

  .cart-animation-container.closing {
    animation: slide-out-lite 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  }

  .page-container {
    margin: auto 48px;
  }
}

@media (min-width: 1601px) and (max-width: 1920px) {
  .page-container.cart-open {
    transform: translateX(-15%);
    padding-right: 24px;
  }

  .content-flex {
    justify-content: center;
  }
}

@media (min-width: 1921px) {
  .page-container.cart-open {
    transform: translateX(-15%);
    padding-right: 24px;
  }

  .content-flex {
    justify-content: center;
  }
}
</style>
