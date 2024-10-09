<template>
  <div class="max-w-4x2 mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Order List</h1>
    <div class="mb-4 space-y-2">
      <div class="space-x-2">
        <input v-model="freightMin" placeholder="Min Yük" type="number" class="border border-gray-300 p-2 rounded">
        <input v-model="freightMax" placeholder="Max Yük" type="number" class="border border-gray-300 p-2 rounded">
        <select v-model="selectedCountry" @change="fetchCountries" class="border border-gray-300  min-w-[150px] p-2 rounded">
          <option v-for="country in shipCountries" :key="country" :value="country">{{ country }}</option>
        </select>

        <select v-model="selectedCity" class="border border-gray-300 p-2 rounded  min-w-[150px]">
          <option v-for="city in shipCities" :key="city" :value="city">{{ city }}</option>
        </select>
        <input v-model="shipAddress" placeholder="Kargo Adresi" class="border border-gray-300 p-2 rounded">
        <input v-model="orderDate" type="date" class="border border-gray-300 p-2 rounded">
        <button @click="applyFilters" class="bg-blue-500 text-white p-2 rounded">Filtreyi Uygula</button>
        <button @click="sortData('Freight', 'ASC')" class="bg-green-500 text-white p-2 rounded">Yükleri A-Z Sırala</button>
        <button @click="sortData('Freight', 'DESC')" class="bg-red-500 text-white p-2 rounded">Yükleri Z-A Sırala</button>
      </div>     
    </div>

    <table class="min-w-full bg-white border border-gray-300">
      <thead>
        <tr class="bg-gray-100">
          <th class="border border-gray-300 px-4 py-2">Sipariş Tarihi</th>
          <th class="border border-gray-300 px-4 py-2">Kargo Tarihi</th>
          <th class="border border-gray-300 px-4 py-2">Yük</th> 
          <th class="border border-gray-300 px-4 py-2">Ship Address</th>     
          <th class="border border-gray-300 px-4 py-2">Kargo Ülkesi</th>
          <th class="border border-gray-300 px-4 py-2">Kargo Şehri</th> 
        </tr>
      </thead>
      <tbody>
        <tr v-for="order in orders" :key="order.OrderID">
          <td class="border border-gray-300 px-4 py-2">{{ order.OrderDate }}</td>
          <td class="border border-gray-300 px-4 py-2">{{ order.ShipDate }}</td>
          <td class="border border-gray-300 px-4 py-2">{{ order.Freight }}</td>
          <td class="border border-gray-300 px-4 py-2">{{ order.ShipAddress }}</td>
          <td class="border border-gray-300 px-4 py-2">{{ order.ShipCountry }}</td>
          <td class="border border-gray-300 px-4 py-2">{{ order.ShipCity }}</td>
        </tr>
      </tbody>
    </table>

    <div class="flex justify-between mt-4">
      <button @click="previousPage" class="bg-gray-300 p-2 rounded">Önceki</button>
      <button @click="nextPage" class="bg-gray-300 p-2 rounded">Sonraki</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
    name:'OrderList',
    data() {
    return {
      orders: [],
      shipCountries: [],
      shipCities: [], 
      selectedCountry: '',
      selectedCity: '',
      freightMin: null,
      freightMax: null,
      shipAddress: '',
      orderDate: '',
      page: 1,
      pageSize: 10,
      totalOrders: 0
    };
  },
  methods: {
    fetchCountries() {
      axios.get('https://case.frontend.unalmustafa.online/order/filters')
        .then(response => {
          this.shipCountries = response.data.shipCountry; 
          this.shipCities = response.data.shipCity; 
        })
        .catch(error => {
          console.error('Error fetching filters:', error);
        });
    },
    fetchData() {
      const params = {
        page: this.page,
        pageSize: this.pageSize,
        FreightMin: this.freightMin,
        FreightMax: this.freightMax,
        ShipAddress: this.shipAddress,
        ShipCity: this.selectedCity,
        ShipCountry: this.selectedCountry,
        OrderDate: this.orderDate,
        SortValue: 'Freight', 
        SortType: 'ASC'
      };

      axios.get('https://case.frontend.unalmustafa.online/order/list', { params })
        .then(response => {
          this.orders = response.data.data; 
          this.totalOrders = response.data.totalEntry;
        })
        .catch(error => {
          console.error('Verilerin getirilmesi anında hata:', error);
        });
    },
    applyFilters() {
      this.page = 1; 
      this.fetchData();
    },
    previousPage() {
      if (this.page > 1) {
        this.page--;
        this.fetchData();
      }
    },
    nextPage() {
      if (this.page * this.pageSize < this.totalOrders) {
        this.page++;
        this.fetchData();
      }
    }
  },
  mounted() {
    this.fetchCountries();
    this.fetchData(); 
  }
}
</script>

<style scoped>
table {
  width: 100%;
}
</style>