<template>
  <div class="order-element">
    <div class="order-title">
      <p class="title-2">Дата заказа: {{ date }}</p>
      <p class="title-2">Номер заказа: {{ order.id }}</p>
      <p class="title-2">Статус: {{order.status}}</p>
      <p class="cancellation bold centered-horizontally" v-if="order.status == 'отменен пользователем'">
        Заказ отменен пользователем!
      </p>
      <p class="cancellation bold centered-horizontally" v-if="order.status == 'отменен'">
        Заказ отменен сыроваром! Причина: {{ order.comment }}
      </p>
      <div class="border-line"></div>
    </div>
    <div
        v-for="(content, index) in CONTENTS"
        :key="content.id"
    >
      <ProductOrderElement
          ref="ProductOrderElement"
          :content="content"
          v-if="content.order_id === order.id"
          :index="index" :order="order"
      />
    </div>
    <div v-for="pickpoint in DELIVERY_POINTS" :key="pickpoint.id">
      <p class="title-3" v-if="pickpoint.id === order.pickpoint_id">
        <span class="bold">Адрес доставки:</span>
        {{ pickpoint.name }}
      </p>
    </div>
    <div v-for="user in USERS" :key="user.id">
      <div v-if="user.id === order.user_id">
        <p class="title-3"><span class="bold">Ф.И.О.</span> {{ user.fio }}</p>
        <p class="title-3">
          <span class="bold">E-mail:</span> {{ user.email }}
        </p>
        <p class="title-3">
          <span class="bold">Телефон:</span> {{ user.phone }}
        </p>
      </div>
    </div>
    <div class="button-right">
      <button @click="orderSentToThePoint" type="button" class="btn centered" v-if="order.status === 'в обработке'">
        Отправить заказ на точку
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-bar-right"
             viewBox="0 0 16 16">
          <path fill-rule="evenodd"
                d="M6 8a.5.5 0 0 0 .5.5h5.793l-2.147 2.146a.5.5 0 0 0 .708.708l3-3a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L12.293 7.5H6.5A.5.5 0 0 0 6 8zm-2.5 7a.5.5 0 0 1-.5-.5v-13a.5.5 0 0 1 1 0v13a.5.5 0 0 1-.5.5z"/>
        </svg>
      </button>
      <button @click="orderCheckToThePoint" type="button" class="btn centered" v-if="order.status === 'в обработке'">
        Подготовить заказ для отправки
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-bar-right"
             viewBox="0 0 16 16">
          <path fill-rule="evenodd"
                d="M6 8a.5.5 0 0 0 .5.5h5.793l-2.147 2.146a.5.5 0 0 0 .708.708l3-3a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L12.293 7.5H6.5A.5.5 0 0 0 6 8zm-2.5 7a.5.5 0 0 1-.5-.5v-13a.5.5 0 0 1 1 0v13a.5.5 0 0 1-.5.5z"/>
        </svg>
      </button>
      <button @click="orderStopToThePoint" v-if="order.status === 'в обработке'" type="button"
              class="cancellation centered-horizontally btn-text">
        Отменить заказ
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-bar-right"
             viewBox="0 0 16 16">
          <path fill-rule="evenodd"
                d="M6 8a.5.5 0 0 0 .5.5h5.793l-2.147 2.146a.5.5 0 0 0 .708.708l3-3a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L12.293 7.5H6.5A.5.5 0 0 0 6 8zm-2.5 7a.5.5 0 0 1-.5-.5v-13a.5.5 0 0 1 1 0v13a.5.5 0 0 1-.5.5z"/>
        </svg>
      </button>
    </div>
<!--    <p v-if="order.status === 'отправлен на точку'" class="input bold centered-horizontally">-->
<!--      Заказ успешно отправлен на точку!-->
<!--    </p>-->
<!--    <p v-if="order.status === 'прибыл в магазин'" class="btn bold centered-horizontally">-->
<!--      Заказ прибыл на точку!-->
<!--    </p>-->
  </div>
</template>
<script>
import axios from 'axios';
import {mapGetters, mapActions} from 'vuex';
import ProductOrderElement from './ProductOrderElement.vue';
import config from '@/config.js';

export default {
  name: 'OrderElement',
  data() {
    return {
      newOrder: {},
      orderRun: false,
      comment: '',
      products: {},
      usersInfo: {},
      content: {},
    };
  },
  props: {
    order: {
      type: Object,
      default() {
        return {};
      },
    },
    index: {
      type: Number,
    },
  },
  components: {
    ProductOrderElement,
  },
  computed: {
    ...mapGetters(['CONTENTS', 'USERS', 'DELIVERY_POINTS']),
    date() {
      let date = this.order.order_date.split('T')[0].split('-');
      return date[2] + '-' + date[1] + '-' + date[0];
    },
  },
  mounted() {
    this.GET_DELIVERY_POINTS_FROM_API();
  },
  methods: {
    ...mapActions(['GET_DELIVERY_POINTS_FROM_API']),
    getUserInfo() {
      axios({
        method: 'GET',
        url: `${config.url}/users/info/`,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((order) => {
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
    currentDate(date) {
      let dd = date.getDate();
      if (dd < 10) dd = '0' + dd;
      let mm = date.getMonth() + 1;
      if (mm < 10) mm = '0' + mm;
      let yyyy = date.getFullYear();
      if (yyyy < 10) yyyy = '0' + yyyy;
      let hour = date.getHours()
      if (hour < 10) hour = '0' + hour
      let minutes = date.getMinutes()
      if (minutes < 10) minutes = '0' + minutes
      let sec = date.getSeconds()
      if (sec < 10) sec = '0' + sec
      return yyyy + '-' + mm + '-' + dd + 'T' + hour + ':' + minutes + ':' + sec;
    },
    orderCheckToThePoint() {
      let contentsLength = Object.keys(this.CONTENTS).length;
      for (let i = 0; i < contentsLength; i++) {
        this.$refs.ProductOrderElement[i].productCheck();
      }
    },
    orderSentToThePoint() {
      let date = new Date();
      this.orderRun = true;
      this.order.date = this.currentDate(date);
      this.order.delivery_date = this.currentDate(date);
      let order = this.order;
      let contents = this.CONTENTS
      let staffed = []
      let notStaffed = []
      let cancelled = []
      for (let value of Object.values(contents)) {
        if ((value.order_id === this.order.id)) {
          if (value.status === 'подготовлен к отправке') {
            staffed.push(value)
          } else if (value.status === 'отменен') {
            cancelled.push(value)
          } else {
            notStaffed.push(value)
            // console.log('sdcvs');
          }
        }
      }
      if (notStaffed.length === 0) {
        this.order.status = 'отправлен на точку'
        axios({
          method: 'PATCH',
          url: `${config.url}/orders/${this.order.id}`,
          data: order,
          headers: {
            authorization: this.$cookies.get('authorization'),
          },
        })
            .then((order) => {
              console.log(order);
            })
            .catch((error) => {
              this.order.status = 'в обработке'
              console.log(error);
              alert('Ошибка в работе приложения. Обратитесь к администратору.');
            });
      }
    },
    orderStopToThePoint() {
      let comment = prompt('Введите причину отмены.');
      if (comment == '') {
        alert('Вы не ввели причину отмены. Пожалуйста, повторите попытку.');
      } else if (comment == null) {
        return
      } else {
        let contents = this.CONTENTS
        for (let value of Object.values(contents)) {
          if ((value.order_id == this.order.id)) {
            let date = new Date();
            value.date = this.currentDate(date);
            value.status = 'отменен';
            value.comment = comment;
            let content = value;
            content.operation = 3
            console.log(content);
            axios({
              method: 'PATCH',
              url: `${config.url}/contents/${value.id}`,
              data: content,
              headers: {
                authorization: this.$cookies.get('authorization'),
              },
            })
                .then((order) => {
                  console.log(order);
                  alert('Товар убран из заказа.');
                })
                .catch((error) => {
                  console.log(error);
                  alert('Ошибка в работе приложения. Обратитесь к администратору.');
                });
          } else {
          }
        }
      }
      let date = new Date();
      this.order.status = 'отменен';
      this.order.comment = comment;
      this.order.date = this.currentDate(date);
      this.order.delivery_date = this.currentDate(date);
      axios({
        method: 'PATCH',
        url: `${config.url}/orders/${this.order.id}`,
        data: this.order,
        headers: {
          authorization: this.$cookies.get('authorization'),
        },
      })
          .then((order) => {
            // alert('Заказ успешно отменен.');
          })
          .catch((error) => {
            console.log(error);
            alert('Ошибка в работе приложения. Обратитесь к администратору.');
          });
    },
  },
};
</script>
