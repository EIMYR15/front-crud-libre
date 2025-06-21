<template>
  <v-app>
    <!-- Barra de navegación superior -->
    <v-app-bar color="indigo-darken-3" dark flat>
      <v-toolbar-title>📚 Biblioteca</v-toolbar-title>
      <v-spacer />

      <v-btn
        text
        to="/libros"
        :class="{ 'text-white text-decoration-underline': $route.path.startsWith('/libros') }"
      >
        Libros
      </v-btn>

      <v-btn
        text
        to="/generos"
        :class="{ 'text-white text-decoration-underline': $route.path.startsWith('/generos') }"
      >
        Géneros
      </v-btn>
    </v-app-bar>

    <!-- Contenido principal -->
    <v-main>
      <router-view />
    </v-main>

    <!-- Pie de página -->
    <AppFooter />
  </v-app>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router'
const $route = useRoute()
</script>