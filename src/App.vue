<script setup lang="ts">
import { RouterLink, RouterView } from "vue-router";
//import HelloWorld from "./components/HelloWorld.vue";
import {
  NLayout,
  NLayoutHeader,
  NLayoutContent,
  //NLayoutFooter,
  NMenu,
  type MenuOption,
} from "naive-ui";
import { h } from "vue";

const menuOptions: MenuOption[] = [
  {
    label: () =>
      h(RouterLink, { to: { name: "home" } }, { default: () => "Home" }),
    key: "home",
  },
  {
    label: () =>
      h(RouterLink, { to: { name: "profile" } }, { default: () => "Profile" }),
    key: "profile",
  },
];
</script>

<template>
  <!-- <img
      alt="Vue logo"
      class="logo"
      src="@/assets/logo.svg"
      width="125"
      height="125"
    /> -->

  <NLayout>
    <!-- <HelloWorld msg="You did it!" /> -->
    <NLayoutHeader>
      <NMenu :options="menuOptions" mode="horizontal" />
    </NLayoutHeader>
    <NLayoutContent>
      <RouterView />
    </NLayoutContent>
    <!-- <NLayoutFooter>Footer</NLayoutFooter> -->
  </NLayout>
</template>
