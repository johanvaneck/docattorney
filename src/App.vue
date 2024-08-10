<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'

import Menubar from 'primevue/menubar';

import { ref } from "vue";

const items = ref([
  {
    label: 'Home',
    icon: 'pi pi-home',
    route: '/',
  },
  {
    label: 'About',
    icon: 'pi pi-home',
    route: '/about',
  },
]);

</script>

<template>
  <div class="card">
    <Menubar :model="items">
      <template #item="{ item, props, hasSubmenu }">
        <RouterLink v-if="item.route" v-slot="{ href, navigate }" :to="item.route" custom>
          <a v-ripple :href="href" v-bind="props.action" @click="navigate">
            <span :class="item.icon" />
            <span class="ml-2">{{ item.label }}</span>
          </a>
        </RouterLink>
        <a v-else v-ripple :href="item.url" :target="item.target" v-bind="props.action">
          <span :class="item.icon" />
          <span class="ml-2">{{ item.label }}</span>
          <span v-if="hasSubmenu" class="pi pi-fw pi-angle-down ml-2" />
        </a>
      </template>
    </Menubar>
  </div>

  <div id="space" class="h-5"></div>

  <main class="w-screen h-screen px-10">
    <RouterView />
  </main>
</template>

<style scoped></style>
