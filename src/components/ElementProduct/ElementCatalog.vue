<template>
  <div class="catalog-items-products"  v-if="product.active == 1">
    <!--product quantity checked-->
    <div class="catalog-element products">
      <transition name="fade">
        <div class="error-mesage" v-if="isVisible">
          <p class="title-3 text-centered">
            Укажите пожалуйста цену и вес товара
          </p>
        </div>
      </transition>
      <div class="catalog-items-products-img">
        <img
          :src="'http://shop-dev.zdmail.ru' + product.image_path.slice('2')"
          :alt="product.name"
          class="catalog-element-img"
        />
      </div>
      <p class="paragraph bold text-centered">{{ product.name }}</p>
      <img
        src="/allImage/Icons/que.svg"
        alt="Описание продукта"
        class="img-question"
        @mouseover="description = !description"
      />
      <div class="v-select width-200" v-click-outside="onClickOutside" >
        <p class="input" @click="areOptionsVisible = !areOptionsVisible">
          {{ selected.name }}
        </p>
        <div class="options-product" v-if="areOptionsVisible" v-click-outside="onClickOutside">
          <div v-for="price in PRICES" :key="price.id">
            <a
              class="input bold search"
              v-if="(price.product_id == product.id) && (price.active == 1)"
              @click="selectOption(price)"
            >
              {{ price.item_price }}&nbsp;₽ | 1 &nbsp;{{ price.item_measure }}
            </a>
          </div>
        </div>
      </div>
        <div class="cart-element">
        <input type="button" class="btn margin-0-10" @click="minusAmount" value="-" />
        <input
          type="number"
          v-model="amount"
          class="input max-width-50"
          step="1"
          min="1"
          x-data="{}"
          @keydown="
            if (['+', '-', 'e'].includes($event.key)) $event.preventDefault();
          "
        />
        <input
          type="button"
          class="btn margin-0-10"
          @click="summAmount"
          value="+"
        />
      </div>
      <button type="submit" @click="addToCart" class="btn centered">
        В корзину
        <img
          src="/allImage/Icons/buybuttone.png"
          class="element-catalog-bottom-img-cart"
        />
      </button>
    </div>
    <transition name="description">
      <div
        class="products description pointer"
        v-if="description"
        @mouseleave="description = false"
      >
        <div class="description-text">
          <div>
            <p class="paragraph-small text-centered bold">
              {{ product.description }}
            </p>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'ElementCatalog',
  data() {
    return {
      productAmount: '',
      productАvailability: false,
      units: [
        { value: 0, text: 'шт' },
        { value: 1, text: 'кг' },
        { value: 2, text: 'л' },
      ],
      productId: '',
      inStockQuantity: '',
      productInStock: false,
      description: false,
      choice: false,
      areOptionsVisible: false,
      selected: {name: 'Выбрать упаковку'},
      isVisible: false,
      amount: 1
    };
  },
  props: {
    adminLogin: {
      type: Object,
      default() {
        return {};
      },
    },
    product: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  computed: {
    ...mapGetters(['PRODUCTS', 'CATEGORIES', 'PRICES']),
  },
  methods: {
    summAmount() {
      this.amount++
      this.product.amount = this.amount 
    },
    minusAmount() {
      this.amount--
      if (this.amount == 0) {
        this.amount = 1
      this.product.amount = this.amount 
      } 
      this.product.amount = this.amount 
    },
    onClickOutside() {
      this.areOptionsVisible = false;
    },
    selectOption(PRICES) {
      this.areOptionsVisible = false;
      this.selected.name = PRICES.item_price + ' ₽ | 1 ' + PRICES.item_measure;
      this.selected.id = PRICES.id
      this.selectedMeasure = PRICES.item_measure;
    },
    quantityProduct() {
      this.inStockQuantity = this.product.inStockQuantity;
      return this.inStockQuantity;
    },
    isEmpty() {
      if (this.product.inStock == 0) {
        return false;
      }
      return true;
    },
    addToCart() {
      if (this.selected.name == 'Выбрать упаковку') {
        setTimeout(() => {
          this.isVisible = !this.isVisible;
        }, 2000);
        return (this.isVisible = !this.isVisible);
      }
      this.product.amount = this.amount 
      this.$emit('addToCart', this.product, this.selected);
    },
  },
};
</script>
<style>
.description-enter-active,
.description-leave-active {
  transition: opacity 0.5s ease;
}

.description-enter-from,
.description-leave-to {
  opacity: 0;
}
</style>
