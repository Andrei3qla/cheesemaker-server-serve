<template>
  <div class="catalog-items">
    <div class="catalog-element">
      <img
        :src="'http://shop-dev.zdmail.ru' + product.image_path.slice('2')"
        :alt="product.name"
        class="catalog-element-img"
      />
      <div class="text-align-right">
        <h3 class="title-3 text-centered">{{ product.name }}</h3>
      </div>
      <p class="paragraph-small margin-10-0 text-centered">
        Введите кол-во приготовленного продукта:
      </p>
      <input class="input" type="number" v-model="form.amount" step="0.01" />
      <select class="input" v-model="form.item_measure">
        <option>кг</option>
        <option>л</option>
        <option>шт</option>
      </select>
      <select class="input" v-model="selectStore">
        <option disabled>Выбрать склад</option>
        <option
          v-for="storage in STORAGES"
          :key="storage.id"
          :value="storage.id"
        >
          {{ storage.name }}
        </option>
      </select>
      <input type="date" class="input" v-model="form.date" />
      <button class="btn" @click="addProductProduced()">Добавить</button>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import axios from 'axios';
import config from '@/config.js'

export default {
  name: 'ElementCatalogCheesemaker',
  data() {
    return {
      units: [
        { value: 0, text: 'шт' },
        { value: 1, text: 'кг' },
        { value: 2, text: 'л' },
      ],
      form: {
        amount: 0.01,
        manufacturer_id: 1,
        item_measure: 'кг',
        operation: 'приход',
      },
      selectStore: 'Выбрать склад',
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
    ...mapGetters(['PRODUCTS', 'CATEGORY', 'STORAGES']),
  },
  methods: {
    addProductProduced() {
      if (this.selectStore == 'Выбрать склад') {
        alert('Выберете склад!');
      } else if (this.form.date == null){
        alert('Введите дату!');
      } else {
        this.form.product_id = this.product.id;
        this.form.storage_id = this.selectStore;
        this.form.author_id = this.$cookies.get('id')
        let content = this.form
        axios
          ({
          method: 'POST',
          url: `${config.url}/productions`,
          data: content,
				  headers: { 
					    "authorization":  this.$cookies.get('authorization')
          }
			  })
          .then((res) => {
            alert('Продукт добавлен');
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
        }
    },
    isEmpty() {
      if (this.product.inStock == 0) {
        return false;
      }
      return true;
    },
  },
};
</script>
