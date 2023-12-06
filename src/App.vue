<script setup lang="ts">
import PageHeader from "./components/PageHeader.vue";
import axiosClient from "./utils/axios";
import { onMounted, ref, watch } from "vue";
import { Country } from "./models/country.model";
import CountryList from "./components/CountryList.vue";

const fetchCountries = async () => {
  try {
    const { data } = await axiosClient.get("/all");
    countries.value = data;
    totalItems.value = countries.value.length;
  } catch (error) {
    console.log(error);
  }
};

onMounted(() => {
  fetchCountries();
});

const countries = ref<Country[]>([]);
const search = ref("");
const filteredCountries = ref<Country[]>([]);
const page = ref(1);
const itemsPerPage = ref(12);
const paginetedCountries = ref<Country[]>([]);
const totalItems = ref(0);

const filterCountries = () => {
  filteredCountries.value = countries.value.filter((country) =>
    country.name.common.toLowerCase().includes(search.value.toLowerCase())
  );
};

const sliceCountries = (courrentCountries: Country[]) => {
  const start = (page.value - 1) * itemsPerPage.value;
  const end = page.value * itemsPerPage.value;
  paginetedCountries.value = courrentCountries.slice(start, end);
};

watch([countries, page, filteredCountries], () => {
  sliceCountries(
    filteredCountries.value.length <= 0 && search.value == ""
      ? countries.value
      : filteredCountries.value
  );
});

const changePage = (newPage: number) => {
  page.value = newPage;
};
</script>

<template>
  <PageHeader />

  <div class="container max-w-screen-lg mx-auto px-6">
    <div class="mb-8">
      <input
        @input="filterCountries"
        type="text"
        v-model="search"
        class="border border-gray-300 rounded w-full p1 px-4"
        placeholder="Search by country name"
      />
    </div>
    <div class="mb-8 flex justify-center space-x-6">
      <button
        :class="{ 'opacity-50': page <= 1 }"
        :disabled="page <= 1"
        @click="changePage(page - 1)"
        class="border border-gray-300 rounded px-2 py-0.5 hover:bg-gray-200"
      >
        Previous
      </button>
      <button
        :class="{ 'opacity-50': page >= totalItems / itemsPerPage }"
        :disabled="page >= totalItems / itemsPerPage"
        @click="changePage(page + 1)"
        class="border border-gray-300 rounded px-2 py-0.5 hover:bg-gray-200"
      >
        Next
      </button>
    </div>
    <CountryList :countries="paginetedCountries" />
  </div>
</template>
