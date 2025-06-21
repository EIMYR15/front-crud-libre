<template>
  <v-container>
    <v-card class="mx-auto my-8" max-width="1200">
      <v-card-title>
        <span class="title-h5">Lista de Libros</span>
      </v-card-title>
      <v-card-title>
        <v-data-table
          :headers="headers"
          :items="libros"
          :loading="loading"
          loading-title="Cargando libros..."
          class="elevation-1"
          item-key="id"
          dense
        >
          <template #item.activo="{ item }">
            <v-chip :color="item.activo ? 'green' : 'red'" dark>
              {{ item.activo ? 'Sí' : 'No' }}
            </v-chip>
          </template>
          <template #item.createdAt="{ item }">
            {{ formatDate(item.createdAt) }}
          </template>
          <template #item.updatedAt="{ item }">
            {{ formatDate(item.updatedAt) }}
          </template>
        </v-data-table>
        <v-alert v-if="error" type="error" class="mt-4">
          {{ error }}
        </v-alert>
      </v-card-title>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const libros = ref([])
const loading = ref(true)
const error = ref('')

const headers = [
  { title: 'ID', value: 'id' },
  { title: 'Título', value: 'titulo' },
  { title: 'Autor', value: 'autor' },
  { title: 'Año', value: 'anioPublicacion' },
  { title: 'Género', value: 'genero' },
  { title: 'Activo', value: 'activo' },
  { title: 'Creado', value: 'createdAt' },
  { title: 'Actualizado', value: 'updatedAt' },
]
/*const headers = [
  { title: 'ID', value: 'id' },
  { title: 'Nombre Completo', value: 'nombreCompleto', sortable: true },
  { title: 'Especialidad', value: 'especialidad', sortable: true },
  { title: 'Días y horarios', value: 'diasHorarios' },
  { title: 'Acciones', value: 'actions', sortable: false },
]*/

function formatDate(dateStr) {
  if (!dateStr) return ''
  // Si viene en formato ISO, solo la fecha
  return dateStr.split('T')[0]
}

const fetchLibros = async () => {
  loading.value = true
  error.value = ''
  try {
    const res = await fetch('http://localhost:3333/api/libros')
    const data = await res.json()
    // Si usas paginación en el backend, los datos están en data.data
    libros.value = Array.isArray(data.data) ? data.data : data
  } catch (e) {
    error.value = 'Error al cargar los libros'
  }
  loading.value = false
}

onMounted(fetchLibros)
</script>