<template>
  <!-- Main app container -->
  <div :class="['app', { dark: darkMode }]">

    <!-- Header section -->
    <header class="topbar">
      <div class="brand-block">
        <h1>MERCADO NEGRO D.C</h1>
        <p>Todo la moda a la palma de la mano</p>
      </div>

      <!-- Header controls -->
      <div class="top-controls">

        <!-- Product search input -->
        <input
          v-model="search"
          type="text"
          placeholder="Buscar productos..."
          class="search-input"
        />

        <!-- Category filter -->
        <select v-model="selectedCategory" class="category-select">
          <option value="Todos">Todos</option>
          <option v-for="cat in categories" :key="cat" :value="cat">
            {{ cat }}
          </option>
        </select>

        <!-- Dark mode toggle -->
        <button class="theme-btn" @click="toggleTheme">
          {{ darkMode ? 'Claro' : 'Oscuro' }}
        </button>

        <!-- Cart toggle button -->
        <button class="cart-btn" @click="showCart = !showCart">
          🛒 {{ cartCount }}
        </button>

      </div>
    </header>

    <!-- Main layout -->
    <main class="main-layout">

      <!-- Products section -->
      <section class="products-section">

        <div class="products-grid">
          <ProductCard
            v-for="product in filteredProducts"
            :key="product.id"
            :product="product"
            @add-to-cart="addToCart"
          />
        </div>

        <!-- Message when no products match filters -->
        <p v-if="filteredProducts.length === 0" class="empty-message">
          No se encontraron productos.
        </p>

      </section>

      <!-- Cart panel -->
      <aside v-if="showCart" class="cart-panel">

        <div class="cart-panel-header">
          <h2>Carrito</h2>

          <!-- Clear cart -->
          <button v-if="cart.length" class="clear-cart-btn" @click="clearCart">
            Vaciar
          </button>
        </div>

        <!-- Empty cart message -->
        <div v-if="cart.length === 0" class="cart-empty">
          El carrito está vacío.
        </div>

        <!-- Cart items -->
        <div v-else class="cart-list">

          <div v-for="item in cart" :key="item.cartKey" class="cart-item">

            <img
              :src="item.selectedImage"
              :alt="item.name"
              class="cart-item-image"
            />

            <div class="cart-item-info">
              <h4>{{ item.name }}</h4>
              <p>{{ item.category }}</p>

              <p v-if="item.selectedColor">
                Color: {{ item.selectedColor }}
              </p>

              <strong>$ {{ formatPrice(item.price) }}</strong>

              <!-- Quantity controls -->
              <div class="qty-controls">
                <button @click="decreaseQty(item.cartKey)">-</button>
                <span>{{ item.qty }}</span>
                <button @click="increaseQty(item.cartKey)">+</button>
              </div>

            </div>

            <!-- Remove item -->
            <button class="remove-btn" @click="removeFromCart(item.cartKey)">
              ✕
            </button>

          </div>

          <!-- Cart summary -->
          <div class="cart-summary">
            <p>Total: <strong>$ {{ formatPrice(cartTotal) }}</strong></p>

            <!-- Checkout button -->
            <button class="buy-btn" @click="checkout">
              Comprar
            </button>
          </div>

        </div>
      </aside>
    </main>
  </div>
</template>

<script>

// Product component
import ProductCard from './components/ProductCard.vue'

export default {

  name: 'App',

  components: {
    ProductCard,
  },

  data() {
    return {

      // UI state
      darkMode: false,
      showCart: true,

      // Search and category filters
      search: '',
      selectedCategory: 'Todos',

      // Cart storage
      cart: [],

      // Product catalog
      products: [
        {
          id: 1,
          name: 'Camiseta Oversize',
          category: 'Ropa',
          price: 45000,
          stock: 8,
          image: '/Imagenes/OversideNegra.png',

          colors: [
            {
              name: 'Negro',
              hex: '#111111',
              image: '/Imagenes/OversideNegra.png',
            },
            {
              name: 'Blanco',
              hex: '#f1f1f1',
              image: '/Imagenes/OversideBlanca.png',
            },
            {
              name: 'Azul',
              hex: '#60a5fa',
              image: '/Imagenes/OversideAzul.png',
            },
          ],
        },

        {
          id: 2,
          name: 'Audífonos Bluetooth',
          category: 'Tecnología',
          price: 120000,
          stock: 5,
          image: '/Imagenes/audifonos.png',

          colors: [
            {
              name: 'Negro',
              hex: '#111111',
              image: '/Imagenes/audifonos.png',
            },
          ],
        },

        {
          id: 3,
          name: 'Gorra',
          category: 'Accesorios',
          price: 30000,
          stock: 4,
          image: '/Imagenes/GorraNegra.png',

          colors: [
            {
              name: 'Negro',
              hex: '#111111',
              image: '/Imagenes/GorraNegra.png',
            },
            {
              name: 'Rojo',
              hex: '#ef4444',
              image: '/Imagenes/GorraRoja.png',
            },
          ],
        },

        {
          id: 4,
          name: 'Mouse Gamer',
          category: 'Tecnología',
          price: 90000,
          stock: 3,
          image: '/Imagenes/MouseGamer.png',

          colors: [
            {
              name: 'Negro',
              hex: '#111111',
              image: '/Imagenes/MouseGamer.png',
            },
          ],
        },

        {
          id: 5,
          name: 'Chaqueta Adidas',
          category: 'Ropa',
          price: 135000,
          stock: 2,
          image: '/Imagenes/ChaquetaNegra.png',

          colors: [
            {
              name: 'Negro',
              hex: '#111111',
              image: '/Imagenes/ChaquetaNegra.png',
            },
            {
              name: 'Gris',
              hex: '#9ca3af',
              image: '/Imagenes/ChaquetaGris.png',
            },
            {
              name: 'Verde',
              hex: '#22c55e',
              image: '/Imagenes/ChaquetaVerde.png',
            },
          ],
        },
      ],
    }
  },

  computed: {

    // Extract unique categories from products
    categories() {
      return [...new Set(this.products.map((p) => p.category))]
    },

    // Filter products by search and category
    filteredProducts() {
      return this.products.filter((product) => {

        const matchesSearch = product.name
          .toLowerCase()
          .includes(this.search.toLowerCase())

        const matchesCategory =
          this.selectedCategory === 'Todos' ||
          product.category === this.selectedCategory

        return matchesSearch && matchesCategory
      })
    },

    // Total number of products in cart
    cartCount() {
      return this.cart.reduce((acc, item) => acc + item.qty, 0)
    },

    // Total cart price
    cartTotal() {
      return this.cart.reduce(
        (acc, item) => acc + item.price * item.qty,
        0
      )
    },

  },

  methods: {

    // Toggle dark/light mode
    toggleTheme() {
      this.darkMode = !this.darkMode
    },

    // Add product to cart
    addToCart(product) {

      const colorKey = product.selectedColor || 'default'
      const cartKey = `${product.id}-${colorKey}`

      const existing = this.cart.find(
        (item) => item.cartKey === cartKey
      )

      if (existing) {

        if (existing.qty < product.stock) {
          existing.qty++
        }

      } else {

        this.cart.push({
          ...product,
          cartKey,
          qty: 1,
        })

      }
    },

    // Increase item quantity
    increaseQty(cartKey) {

      const item = this.cart.find(
        (p) => p.cartKey === cartKey
      )

      if (item && item.qty < item.stock) {
        item.qty++
      }

    },

    // Decrease item quantity
    decreaseQty(cartKey) {

      const item = this.cart.find(
        (p) => p.cartKey === cartKey
      )

      if (!item) return

      if (item.qty > 1) {
        item.qty--
      } else {
        this.removeFromCart(cartKey)
      }

    },

    // Remove product from cart
    removeFromCart(cartKey) {
      this.cart = this.cart.filter(
        (item) => item.cartKey !== cartKey
      )
    },

    // Clear all cart items
    clearCart() {
      this.cart = []
    },

    // Simulated checkout
    checkout() {

      if (!this.cart.length) return

      alert('Compra realizada con éxito')

      this.clearCart()
    },

    // Format price for display
    formatPrice(value) {
      return value.toLocaleString('es-CO')
    }

  }

}
</script>