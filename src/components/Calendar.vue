<template>
  <div>
    <v-select v-model="productID" :items="products" dense outlined hide-details item-text="name" item-value="id"
        class="ma-2" label="Select Product" style="width: 50%;" @change="getEvents"></v-select>
    <v-sheet tile height="4vh" color="grey lighten-3" class="d-flex align-center">
      <v-btn icon @click="navigateToPreviousMonth()">
        <v-icon>mdi-chevron-left</v-icon>
      </v-btn>
      <v-btn icon @click="navigateToNextMonth()">
        <v-icon>mdi-chevron-right</v-icon>
      </v-btn>
      <v-toolbar-title>{{ title }}</v-toolbar-title>
    </v-sheet>
    <v-sheet height="94vh">
      <v-calendar ref="calendar" v-model="currentDate" type="month" :events="events" :event-color="getEventColor" locale="en-US"
        :day-format="(timestamp) => new Date(timestamp.date).getDate()"
        :month-format="(timestamp) => new Date(timestamp.date).getMonth() + 1 + ' /'" @change="getEvents"
        ></v-calendar>
    </v-sheet>
  </div>
  
</template>

<script>
import axios from 'axios'
import moment from 'moment';

export default {
  data: () => ({
    events: [],
    currentDate: new Date(),
    month: new Date().getMonth()+1,
    year:new Date().getFullYear(),
    productID: 0,
    products: []
  }),
  computed: {
    title() {
      return moment(this.currentDate).format('yyyy M');
    },
  },
  mounted() {
    this.listProduct()
  },
  methods: {
    navigateToPreviousMonth() {
      this.$refs.calendar.prev(); 
      this.month = this.currentDate.getMonth() + 1;
      this.year = this.currentDate.getFullYear()
    },
    navigateToNextMonth() {
      this.$refs.calendar.next();
      this.month = this.currentDate.getMonth() + 1;
      this.year = this.currentDate.getFullYear()
    },
    getEvents() {
      const config = { headers: { 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Headers': 'Origin, X-Requested-With, Content-Type, Accept' } };
      var url = '/product/' + this.productID + '?month=' + this.month + '&year=' + this.year
      axios.get(url, config)
        .then(response => {
          var stockProducts = response.data.stock_item_data;

          var events = stockProducts.map(item => ({
            name: (item.available_stock).toString(),
            start: moment(item.date).toDate(),
           end: moment(item.date).toDate(),
            color: item.available_stock > 0 ? 'green' : 'red',
            timed: false
          }));
          this.events = events;
        })
        .catch(error => {
          console.error('Error fetching data:', error);
        });
    },
    listProduct() {
      const config = { headers: { 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Headers': 'Origin, X-Requested-With, Content-Type, Accept' } };
      var url = '/list/product'
      axios.get(url, config)
        .then(response => {
          this.products = response.data;
          this.productID = this.products[0].id;
          this.getEvents()
        })
        .catch(error => {
          console.error('Error Get Product:', error);
        });
    },
    getEventColor(event) {
      return event.color;
    },
  },
};
</script>
