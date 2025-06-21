<template>
  <v-container class="fill-height d-flex justify-center align-center">
    <v-card class="pa-6 rounded-xl elevation-12" max-width="500">
      <v-card-title class="text-h5 font-weight-bold text-center">
        📚 Bienvenido a la Biblioteca
      </v-card-title>

      <v-card-text class="text-center">
        Explora nuestros libros y géneros disponibles.
      </v-card-text>

      <v-card-actions class="d-flex flex-column gap-4">
        <v-btn
          color="primary"
          variant="flat"
          class="w-100"
          @click="router.push('/libros')"
        >
          Ver Libros
        </v-btn>

        <v-btn
          color="secondary"
          variant="flat"
          class="w-100"
          @click="router.push('/generos')"
        >
          Ver Géneros
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
const router = useRouter()
</script>
