<template>
  <div class="admin-panel">
    <div class="catalog-element-wrap text-centered">
      <label class="text-reader">
        <input type="file" ref="files" id="files" @change="handleImage" accept="image/*"/>
        <input type="button" class="btn" value="Загрузить изображение" @click="addFiles()"/>
      </label>
      <div class="catalog-element-text">
        <label for="name">
          <input
              type="text"
              id="name"
              v-model="product.name"
              class="input"
              placeholder="Название"
              required
          />
        </label>
        <textarea
            placeholder="Описание"
            type="text"
            class="input"
            v-model="product.description"
            required
        ></textarea>
      </div>
      <p>Категория: {{ selectCategory.name }}</p>
      <select class="input" v-model="selectCategory.id">
        <option
            v-for="category in CATEGORY"
            :key="category.id"
            :value="category.id"
            :selected="selectCategory.name"
        >
          {{ category.name }}
        </option>
      </select>
      <!-- <div  v-for="category in CATEGORY"
          :key="category.id">
      <input type="radio" v-model="selectCategory.id"
          :value="category"/>
          <label :checked="selectCategory.name">{{category.name}}</label>
        </div> -->
      <div class="text-centered">
        <button class="btn" @click="ApplyProductChanges(product.id)">
          Применить изменения
        </button>
        <button
            class="btn"
            @click="isVisibleProduct(product.id)"
            v-if="product.active"
        >
          Скрыть
        </button>
        <button
            class="btn"
            @click="isVisibleProduct(product.id)"
            v-if="!product.active"
        >
          Показать
        </button>
      </div>
      <button class="btn" @click="priceChange = !priceChange">
        Настройка цен
      </button>
      <div class="border-no-absolutle margin-10-0" v-if="priceChange">
        <div v-for="price in PRICES" :key="price.id">
          <div v-if="price.product_id == product.id">
            <div class="cart-element-wrap">
              <input
                  type="number"
                  id="description"
                  class="input small-index"
                  placeholder="price.item_price"
                  v-model="price.item_price"
                  v-if="price.visible"
                  required
              />
              <p class="paragraph" v-if="price.visible">&nbsp;₽ | 1 &nbsp;</p>
              <select
                  v-model="price.item_measure"
                  class="input"
                  v-if="price.visible"
              >
                <option>кг</option>
                <option>л</option>
                <option>шт</option>
              </select>
              <button
                  class="btn"
                  @click="chagePriceChanges(price)"
                  v-if="price.visible"
              >
                Применить
              </button>
            </div>
          </div>
          <div v-if="price.product_id == product.id" class="cart-element-wrap">
            <input
                type="checkbox"
                true-value="1"
                false-value="0"
                v-model="price.active"
                @click="ApplyPriceChanges(price.active, price, price.id)"
                v-if="!price.visible"
            />
            <p class="paragraph" v-if="!price.visible">
              {{ price.item_price }}&nbsp;₽ | 1 &nbsp; {{ price.item_measure }}
            </p>
            <button
                class="btn"
                @click="price.visible = !price.visible"
                v-if="!price.visible"
            >
              Изменить
            </button>
          </div>
        </div>
        <div class="centered pickpoint-map-container">
          <label for="description">
            <input
                type="number"
                id="description"
                class="input small-index"
                placeholder="Цена"
                v-model="item_price"
                required
            />
          </label>
          <p class="paragraph margin-0-10">| 1</p>
          <select v-model="item_measure" class="input">
            <option>кг</option>
            <option>л</option>
            <option>шт</option>
          </select>
          <button
              class="btn"
              @click="addPriceProduct(item_measure, item_price)"
          >
            Добавить
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {mapGetters} from 'vuex';
import axios from 'axios';
import config from '@/config.js'

export default {
  name: 'AdminPanel',
  components: {},

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
  data() {
    return {
      avatar: '',
      check: [],
      item_price: '',
      item_measure: 'кг',
      priceChange: false,
      isActivePrice: true,
      selectCategory: {name: 'Все категории', id: ''},
    };
  },
  computed: {
    ...mapGetters(['PRICES', 'CATEGORY']),
  },
  mounted() {
    this.form = this.product;
    this.isActivePrice = this.PRICES.active;
    this.nameSelect();
  },
  methods: {
    addFiles() {
      this.$refs.files.click()
    },
    nameSelect() {
      axios({
        method: 'GET',
        url: `${config.url}/categories/${this.product.category_id}`,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((res) => {
            return (this.selectCategory.name = res.data.name);
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    chagePriceChanges(price) {
      axios({
        method: 'PATCH',
        url: `${config.url}/prices/${price.id}`,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((res) => {
            alert('Цена успешно изменена');
            price.visible = !price.visible;
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    addPriceProduct(item_measure, item_price) {
      let formAddPrice = {};
      formAddPrice.item_measure = item_measure;
      formAddPrice.item_price = item_price;
      formAddPrice.product_id = this.product.id;
      formAddPrice.active = 1;
      formAddPrice.author_id = 1;
      axios({
        method: 'POST',
        url: `${config.url}/prices/`,
        data: formAddPrice,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((res) => {
            location.reload(res);
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    handleImage(e) {
      const selectedImage = e.target.files[0]; // get first file
      this.createBase64Image(selectedImage);
      let type = selectedImage.type.split('/')[1];
      this.product.ext = type;
    },
    createBase64Image(fileObject) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.image = e.target.result;
        const {image} = this;
        let base64 = image.split(',')[1];
        this.product.image = base64;
      };
      reader.readAsDataURL(fileObject);
    },
    isVisibleProduct(index) {
      let proof = confirm('Изменить видимость товара?');
      if (proof == true) {
        this.product.active = !this.product.active;
        axios({
          method: 'PATCH',
          url: `${config.url}/products/${index}`,
          data: this.product,
          headers: {
            authorization: this.$cookies.get('authorization'),
          },
        })
            .then((res) => {
              location.reload(res);
            })
            .catch((error) => {
              console.log(error);
              alert('Ошибка в работе приложения. Обратитесь к администратору.');
            });
      }
      return;
    },
    ApplyProductChanges(index) {
      this.product.category_id = this.selectCategory.id;
      let product = this.product;
      axios({
        method: 'PATCH',
        url: `${config.url}/products/${index}`,
        data: product,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((res) => {
            location.reload(res);
            this.product.category_id = this.selectCategory.id;
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    ApplyPriceChanges(isActive, price, index) {
      price.active = Number(!isActive);
      axios({
        method: 'PATCH',
        url: `${config.url}/prices/${index}`,
        data: price,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((res) => {
            alert('Изменения применены');
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => this.$emit('load', e.target.result);
      reader.readAsText(file);
    },
  },
};
</script>
