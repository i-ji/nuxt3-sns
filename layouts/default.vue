<template>
  <v-app-bar color="primary" class="text-center" @click="topScroll">
    <template v-slot:prepend>
      <v-avatar color="info">
        <v-icon icon="mdi-account-circle"></v-icon>
      </v-avatar>
    </template>
    <v-app-bar-title>Application</v-app-bar-title>
  </v-app-bar>
  <v-main>
    <slot />
  </v-main>
  <v-footer color="primary" app>
    <v-col class="text-center">(c) BBLED</v-col>
  </v-footer>
</template>

<script setup lang="ts">
const topScroll = () => {
  window.scrollTo({
    top: 0,
    behavior: "smooth",
  });
};
</script>
