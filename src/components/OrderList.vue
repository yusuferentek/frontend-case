<template>
  <div class="w-full p-6 bg-gray-50">
    <!-- Filtreleme Alanı -->
    <div class="flex flex-wrap items-center space-x-4 mb-6">

      <!-- Ship Address Filter -->
      <div class="flex-1 min-w-[150px]">
        <label for="ship-address" class="block text-sm font-medium text-gray-700">Ship Address</label>
        <input type="text" id="ship-address" v-model="filters.shipAddress" @input="onFilterChange"
          placeholder="Adres Giriniz"
          class="mt-1 w-full border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500" />
      </div>

      <!-- Order Date Picker -->
      <div class="flex-1 min-w-[150px]">
        <label for="order-date" class="block text-sm font-medium text-gray-700">Order Date</label>
        <input type="date" id="order-date" v-model="filters.orderDate" @change="onFilterChange"
          class="mt-1 w-full border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500" />
      </div>

      <!-- SortValue Dropdown -->
      <div class="flex-1 min-w-[150px]">
        <label for="sort-value" class="block text-sm font-medium text-gray-700">Sort By</label>
        <select id="sort-value" v-model="filters.sortValue"
          class="mt-1 w-full border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500">
          <option disabled value="">Sıralamak için bir sütun seçiniz</option>
          <option value="orderDate">Order Date</option>
          <option value="freight">Freight</option>
          <option value="shipCity">Ship City</option>
          <option value="shipCountry">Ship Country</option>
        </select>
      </div>

      <!-- SortType Dropdown -->
      <div class="flex-1 min-w-[150px]">
        <label for="sort-type" class="block text-sm font-medium text-gray-700">Sort Type</label>
        <select id="sort-type" v-model="filters.sortType"
          class="mt-1 w-full border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500">
          <option disabled value="">Sıralama Tipini Seçiniz</option>
          <option value="asc">ASC</option>
          <option value="desc">DESC</option>
        </select>
      </div>
    </div>

    <!-- Freight Min -->
    <div class="flex flex-wrap items-center space-x-4 mb-6">
      <div class="flex-1 min-w-[150px]">
        <label for="freight-min" class="block text-sm font-medium text-gray-700">Freight Min</label>
        <input type="number" id="freight-min" v-model="filters.freightMin"
          class="mt-1 border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500"
          placeholder="Min value" />
      </div>
    <!-- Freight Min -->
    <div class="flex-1 min-w-[150px]">
        <label for="freight-max" class="block text-sm font-medium text-gray-700">Freight Max</label>
        <input type="number" id="freight-max" v-model="filters.freightMax"
          class="mt-1 border-gray-300 rounded-lg shadow-sm p-3 focus:ring-indigo-500 focus:border-indigo-500"
          placeholder="Max value" />
      </div>

      <div class="flex-1 min-w-[150px]">
        <button @click="toggleCountryDropdown" class="bg-white text-[#415465] py-2 px-4 rounded-lg border border-gray-300 font-medium cursor-pointer">
          Filtre Ekle - Ship Country
        </button>
        <div v-if="showCountryDropdown" class="absolute bg-white border border-gray-300 shadow-md rounded-lg p-3 w-64">
          <input type="text" placeholder="Ara..." v-model="countrySearch" class="w-full p-2 border border-gray-300 rounded-md mb-2" />
          <div class="max-h-60 overflow-y-auto">
            <div v-for="country in filteredCountries" :key="country">
              <label class="flex items-center">
                <input type="checkbox" v-model="filters.shipCountry" :value="country" class="mr-2" />
                {{ country }}
              </label>
            </div>
          </div>
        </div>
      </div>

      <div class="flex-1 min-w-[150px]">
        <button @click="toggleCityDropdown" class="bg-white text-[#415465] py-2 px-4 rounded-lg border border-gray-300 font-medium cursor-pointer">
          Filtre Ekle - Ship City
        </button>
        <div v-if="showCityDropdown" class="absolute bg-white border border-gray-300 shadow-md rounded-lg p-3 w-64">
          <input type="text" placeholder="Ara..." v-model="citySearch" class="w-full p-2 border border-gray-300 rounded-md mb-2" />
          <div class="max-h-60 overflow-y-auto">
            <div v-for="city in filteredCities" :key="city">
              <label class="flex items-center">
                <input type="checkbox" v-model="filters.shipCity" :value="city" class="mr-2" />
                {{ city }}
              </label>
            </div>
          </div>
        </div>
      </div>

    </div>

    <!-- Table -->
    <div v-if="loading">Yükleniyor...</div>
    <div v-else-if="error">{{ error }}</div>
    <div v-else>
      <table class="min-w-full table-auto border-collapse border border-gray-300 rounded-lg">
        <thead class="bg-gray-200">
          <tr>
            <th class="border border-gray-300 px-4 py-2">Order Date</th>
            <th class="border border-gray-300 px-4 py-2">Ship City</th>
            <th class="border border-gray-300 px-4 py-2">Ship Country</th>
            <th class="border border-gray-300 px-4 py-2">Freight</th>
            <th class="border border-gray-300 px-4 py-2">Ship Address</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="orders.data.length === 0">
            <td colspan="5" class="border border-gray-300 px-4 py-2 text-center">Veri Bulunamadı</td>
          </tr>
          <tr v-for="order in orders.data" :key="order.id" class="hover:bg-gray-100">
            <td class="border border-gray-300 px-4 py-2">{{ order.orderDate }}</td>
            <td class="border border-gray-300 px-4 py-2">{{ order.shipCity }}</td>
            <td class="border border-gray-300 px-4 py-2">{{ order.shipCountry }}</td>
            <td class="border border-gray-300 px-4 py-2">{{ order.freight }}</td>
            <td class="border border-gray-300 px-4 py-2">{{ order.shipAddress }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'OrderList',
  data() {
    return {
      orders: [],
      loading: true,
      error: null,
      shipCities: [],
      shipCountries: [],
      filters: {
        shipCountry: [],
        shipCity: [],
        freightMin: null,
        freightMax: null,
        sortValue: '',
        sortType: '',
        orderDate: '',
        shipAddress: ''
      },
      showCountryDropdown: false,
      showCityDropdown: false,
      citySearch: "",
      countrySearch: "",
    };
  },
  computed: {
    filteredCountries() {
      return this.shipCountries.filter(country =>
        country.toLowerCase().includes(this.countrySearch.toLowerCase())
      );
    },
    filteredCities() {
      return this.shipCities.filter(city =>
        city.toLowerCase().includes(this.citySearch.toLowerCase())
      );
    }
  },
  mounted() {
    this.fetchFilters();
    this.fetchOrders();
  },
  watch: {
    filters: {
      handler() {
        this.fetchOrders();
      },
      deep: true
    }
  },
  methods: {
    async fetchFilters() {
      try {
        const response = await axios.get('https://case.frontend.unalmustafa.online/Order/Filters');
        this.shipCities = response.data.shipCity;
        this.shipCountries = response.data.shipCountry;
        console.log(response.data);
      } catch (err) {
        console.error('Error fetching filters:', err);
      }
    },
    async fetchOrders() {
      this.loading = true;
      try {
        const params = {
          shipCountry: this.filters.shipCountry.join(','),
          shipCity: this.filters.shipCity.join(','),
          freightMin: this.filters.freightMin,
          freightMax: this.filters.freightMax,
          sortValue: this.filters.sortValue,
          sortType: this.filters.sortType,
          orderDate: this.filters.orderDate,
          shipAddress: this.filters.shipAddress
        };
        const response = await axios.get('https://case.frontend.unalmustafa.online/Order/List', { params });
        this.orders = response.data;
        console.log(this.orders);
        console.log('Request:', {
          params
        });
      } catch (err) {
        this.error = 'Failed to load orders';
      } finally {
        this.loading = false;
      }
    },
    toggleCountryDropdown() {
      this.showCountryDropdown = !this.showCountryDropdown;
    },
    toggleCityDropdown() {
      this.showCityDropdown = !this.showCityDropdown;
    },
    onFilterChange() {
      this.fetchOrders();
    }
  }
};
</script>

<style scoped>
table {
  width: 100%;
}
</style>