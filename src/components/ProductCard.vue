<template>
  <article class="product-card">
    <div class="stock-badge">Stock: {{ product.stock }}</div>

    <div class="product-image-box">
      <img :src="currentImage" :alt="product.name" class="product-image" />
    </div>

    <div class="product-body">
      <h3 class="product-title">{{ product.name }}</h3>
      <p class="product-category">{{ product.category }}</p>

      <div class="product-price-row">
        <span class="product-price">$ {{ formatPrice(product.price) }}</span>
        <span class="product-currency">COP</span>
      </div>

      <div v-if="product.colors && product.colors.length" class="color-section">
        <p class="color-label">Color</p>

        <div class="color-options">
          <button
            v-for="color in product.colors"
            :key="color.name"
            class="color-circle"
            :class="{ active: selectedColor === color.name }"
            :style="{ backgroundColor: color.hex }"
            @click="selectColor(color)"
            :title="color.name"
          ></button>
        </div>

        <p class="selected-color">{{ selectedColor }}</p>
      </div>

      <button
        class="add-btn"
        :disabled="product.stock <= 0"
        @click="addProduct"
      >
        Agregar
      </button>
    </div>
  </article>
</template>

<script>
export default {
  name: 'ProductCard',
  props: {
    product: {
      type: Object,
      required: true,
    },
  },
  emits: ['add-to-cart'],
  data() {
    return {
      selectedColor:
        this.product.colors && this.product.colors.length
          ? this.product.colors[0].name
          : '',
      currentImage:
        this.product.colors && this.product.colors.length
          ? this.product.colors[0].image
          : this.product.image,
    }
  },
  methods: {
    selectColor(color) {
      this.selectedColor = color.name
      this.currentImage = color.image
    },
    addProduct() {
      this.$emit('add-to-cart', {
        ...this.product,
        selectedColor: this.selectedColor,
        selectedImage: this.currentImage,
      })
    },
    formatPrice(value) {
      return value.toLocaleString('es-CO')
    },
  },
}
</script>