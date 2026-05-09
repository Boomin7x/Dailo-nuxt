<template>
  <div class="border-b w-full border-gray-200 p-4">
    <div class="container mx-auto flex items-center justify-between">
      <h3 class="text-xl font-bold">Dailo.</h3>
      <div>
        <ul class="flex space-x-4">
          <li v-for="item in navitems" :key="item.name">
            <NuxtLink :to="item.link">{{ item.name }}</NuxtLink>
          </li>
        </ul>
      </div>
      <div>
        <NuxtLink :to="dashboardLink">Go to dashboard</NuxtLink>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { ROUTES } from "~/utils/routes";
const navitems = ref([
  { name: "Home", link: "/" },
  { name: "About", link: "/about" },
  { name: "Contact", link: "/contact" },
]);
const dashboardLink = ref(ROUTES.DASHBOARD.ROOT);
</script>
