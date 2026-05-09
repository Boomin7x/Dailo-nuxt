<template>
  <footer class="border-t w-full border-gray-200 p-4 text-center">
    <div class="container mx-auto">
      <p>&copy; {{ currentYear }} Dailo. All rights reserved.</p>
    </div>
  </footer>
</template>
<script setup>
import { ref } from "vue";
import moment from "moment";
const currentYear = ref(moment().year());
</script>
