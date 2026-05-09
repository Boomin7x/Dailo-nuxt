<script setup></script>
<template>
  <section class="min-h-screen overflow-y-auto overflow-x-hidden flex flex-col">
    <LayoutHeader />
    <main class="flex-1 container mx-auto p-4">
      <slot />
    </main>
    <LayoutFooter />
  </section>
</template>
<style scoped></style>
