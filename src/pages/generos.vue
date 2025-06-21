<template>
  <v-container>
    <v-row justify="space-between" align="center" class="mb-4">
      <v-col>
        <h2>Gestión de Géneros</h2>
      </v-col>
      <v-col cols="auto" class="d-flex align-center">
        <v-btn icon @click="fetchGeneros" class="mr-2" :title="'Refrescar'">
          <v-icon>mdi-refresh</v-icon>
        </v-btn>
        <v-btn color="primary" @click="openDialog()">
          <v-icon left>mdi-plus</v-icon>Nuevo Género
        </v-btn>
      </v-col>
    </v-row>

    <v-row class="mb-4">
      <v-col cols="12" md="6">
        <v-text-field v-model="filters.titulo" label="Filtrar por título" clearable prepend-inner-icon="mdi-tag"
          @keyup.enter="fetchGeneros" />
      </v-col>
    </v-row>
    <v-data-table-server
      v-model:items-per-page="pagination.limit" 
      :headers="headers"
      :items="generos"
      :items-length="pagination.total" 
      :loading="loading" 
      loading-text="Cargando géneros..."
      class="elevation-1"
      item-key="id"
      @update:options="onPageChange"
      >

      <template #item.createdAt="{ item }">
        {{ formatDate(item.createdAt) }}
      </template>
      <template #item.updatedAt="{ item }">
        {{ formatDate(item.updatedAt) }}
      </template>
      <template #item.actions="{ item }">
        <v-icon small class="mr-2" color="primary" @click="openDialog(item)">
          mdi-pencil
        </v-icon>
        <v-icon small color="red" @click="deleteGenero(item.id)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table-server>

    <v-dialog v-model="dialog" max-width="400">
      <v-card>
        <v-card-title>
          <span class="title-h5">{{ form.id ? 'Editar Género' : 'Nuevo Género' }}</span>
        </v-card-title>
        <v-card-title>
          <v-form ref="formRef" v-model="isValid">
            <v-text-field v-model="form.titulo" label="Título" :rules="[v => !!v || 'Requerido']" required />
          </v-form>
        </v-card-title>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="closeDialog()">Cancelar</v-btn>
          <v-btn :disabled="!isValid" color="primary" @click="saveGenero()">
            Guardar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script setup>
import { ref, reactive, onMounted, watch } from 'vue'

const API_BASE = 'http://localhost:3333/api'

const generos = ref([])
const loading = ref(false)
const dialog = ref(false)
const isValid = ref(false)
const formRef = ref()

const filters = reactive({
  titulo: '',
})

const pagination = reactive({
  page: 1,
  limit: 5,
  total: 0,
})

const form = reactive({
  id: undefined,
  titulo: '',
})

const headers = [
  { title: 'ID', value: 'id' },
  { title: 'Título', value: 'titulo', sortable: true },
  { title: 'Creado', value: 'createdAt', sortable: false },
  { title: 'Actualizado', value: 'updatedAt', sortable: false },
  { title: 'Acciones', value: 'actions', sortable: false },
]

function formatDate(dateStr) {
  if (!dateStr) return ''
  return dateStr.split('T')[0]
}

const fetchGeneros = async () => {
  loading.value = true
  try {
    let url = `${API_BASE}/generos?page=${pagination.page}&limit=${pagination.limit}`
    if (filters.titulo) url += `&titulo=${encodeURIComponent(filters.titulo)}`
    const res = await fetch(url)
    const data = await res.json()
    const items = Array.isArray(data.data) ? data.data : data
    generos.value = items
    pagination.total = data.meta ? data.meta.total : items.length
  } catch (e) {
    alert('Error al cargar los géneros')
  }
  loading.value = false
}

function onPageChange({page, itemsPerPage}) {
  pagination.page = page
  pagination.limit = itemsPerPage
  fetchGeneros()
}

function openDialog(item) {
  if (item) Object.assign(form, { ...item })
  else Object.assign(form, { id: undefined, titulo: '' })
  dialog.value = true
}

function closeDialog() {
  dialog.value = false
  formRef.value?.resetValidation()
}

async function saveGenero() {
  if (!formRef.value) return
  const body = { titulo: form.titulo }
  const url = form.id ? `${API_BASE}/generos/${form.id}` : `${API_BASE}/generos`
  const method = form.id ? 'PUT' : 'POST'
  const response = await fetch(url, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  })
  if (!response.ok) {
    const err = await response.json()
    alert(err.message || 'Error al guardar género')
    return
  }
  await fetchGeneros()
  closeDialog()
}

async function deleteGenero(id) {
  if (!confirm('¿Seguro que deseas eliminar este género?')) return
  const response = await fetch(`${API_BASE}/generos/${id}`, { method: 'DELETE' })
  if (!response.ok) return alert('Error al eliminar género')
  await fetchGeneros()
}

watch(() => filters.titulo, () => {
  pagination.page = 1
  fetchGeneros()
})

onMounted(fetchGeneros)
</script>