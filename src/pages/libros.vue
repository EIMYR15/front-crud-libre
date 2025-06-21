<template>
  <v-container>
    <v-row justify="space-between" align="center" class="mb-4">
      <v-col>
        <h2>Gestión de Libros</h2>
      </v-col>
      <v-col cols="auto" class="d-flex align-center">
        <v-btn icon @click="fetchLibros" class="mr-2" :title="'Refrescar'">
          <v-icon>mdi-refresh</v-icon>
        </v-btn>
        <v-btn color="primary" @click="openDialog()">
          <v-icon left>mdi-plus</v-icon>Nuevo Libro
        </v-btn>
      </v-col>
    </v-row>

    <v-tabs v-model="tab">
      <v-tab>Activos</v-tab>
      <v-tab>Inactivos</v-tab>
    </v-tabs>

    <v-row class="mb-4">
      <v-col cols="12" md="4">
        <v-title-field
          v-model="filters.titulo"
          label="Filtrar por título"
          clearable
          prepend-inner-icon="mdi-book"
          @keyup.enter="fetchLibros"
        />
      </v-col>
      <v-col cols="12" md="4">
        <v-title-field
          v-model="filters.autor"
          label="Filtrar por autor"
          clearable
          prepend-inner-icon="mdi-account"
          @keyup.enter="fetchLibros"
        />
      </v-col>
      <v-col cols="12" md="4">
        <v-title-field
          v-model="filters.genero"
          label="Filtrar por género"
          clearable
          prepend-inner-icon="mdi-tag"
          @keyup.enter="fetchLibros"
        />
      </v-col>
    </v-row>

    <v-data-table
      :headers="headers"
      :items="tab === 0 ? libros : librosInactivos"
      :items-per-page="pagination.limit"
      :page="pagination.page"
      :server-items-length="pagination.total"
      :loading="loading"
      loading-title="Cargando libros..."
      class="elevation-1"
      item-key="id"
      dense
      @update:page="onPageChange"
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
      <template #item.actions="{ item }">
        <v-icon small class="mr-2" color="primary" v-if="tab === 0" @click="openDialog(item)">
          mdi-pencil
        </v-icon>
        <v-icon small v-if="tab === 0" color="red" @click="deleteLibro(item.id)">
          mdi-delete
        </v-icon>
        <v-icon small v-if="tab === 1" color="green" @click="restoreLibro(item.id)">
          mdi-restore
        </v-icon>
        <v-icon small v-if="tab === 1" color="red" @click="forceDeleteLibro(item.id)">
          mdi-delete-forever
        </v-icon>
      </template>
    </v-data-table>

    <v-pagination
      v-model="pagination.page"
      :length="Math.ceil(pagination.total / pagination.limit)"
      class="my-4"
      @input="fetchLibros"
    />

    <v-dialog v-model="dialog" max-width="500">
      <v-card>
        <v-card-title>
          <span class="title-h5">{{ form.id ? 'Editar Libro' : 'Nuevo Libro' }}</span>
        </v-card-title>
        <v-card-title>
          <v-form ref="formRef" v-model="isValid">
            <v-title-field
              v-model="form.titulo"
              label="Título"
              :rules="[v => !!v || 'Requerido']"
              required
            />
            <v-title-field
              v-model="form.autor"
              label="Autor"
              :rules="[v => !!v || 'Requerido']"
              required
            />
            <v-title-field
              v-model="form.anioPublicacion"
              label="Año de publicación"
              type="number"
              :rules="[v => !!v || 'Requerido']"
              required
            />
            <v-title-field
              v-model="form.genero"
              label="Género"
              :rules="[v => !!v || 'Requerido']"
              required
            />
          </v-form>
        </v-card-title>
        <v-card-actions>
          <v-spacer />
          <v-btn title @click="closeDialog()">Cancelar</v-btn>
          <v-btn :disabled="!isValid" color="primary" @click="saveLibro()">
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

const libros = ref([])
const librosInactivos = ref([])
const loading = ref(false)
const dialog = ref(false)
const isValid = ref(false)
const formRef = ref()
const tab = ref(0)

const filters = reactive({
  titulo: '',
  autor: '',
  genero: '',
})

const pagination = reactive({
  page: 1,
  limit: 10,
  total: 0,
})

const form = reactive({
  id: undefined,
  titulo: '',
  autor: '',
  anioPublicacion: '',
  genero: '',
  activo: true,
})

const headers = [
  { title: 'ID', value: 'id' },
  { title: 'Título', value: 'titulo', sortable: true },
  { title: 'Autor', value: 'autor', sortable: true },
  { title: 'Año', value: 'anioPublicacion', sortable: true },
  { title: 'Género', value: 'genero', sortable: true },
  { title: 'Activo', value: 'activo', sortable: false },
  { title: 'Creado', value: 'createdAt', sortable: false },
  { title: 'Actualizado', value: 'updatedAt', sortable: false },
  { title: 'Acciones', value: 'actions', sortable: false },
]

function formatDate(dateStr) {
  if (!dateStr) return ''
  return dateStr.split('T')[0]
}

const fetchLibros = async () => {
  loading.value = true
  try {
    let url = `${API_BASE}/libros?page=${pagination.page}&limit=${pagination.limit}`
    if (filters.titulo) url += `&titulo=${encodeURIComponent(filters.titulo)}`
    if (filters.autor) url += `&autor=${encodeURIComponent(filters.autor)}`
    if (filters.genero) url += `&genero=${encodeURIComponent(filters.genero)}`
    const res = await fetch(url)
    const data = await res.json()
    // Adonis paginate: { data: [...], meta: {...} }
    const items = Array.isArray(data.data) ? data.data : data
    libros.value = items.filter(l => l.activo && !l.deletedAt)
    librosInactivos.value = items.filter(l => !l.activo || l.deletedAt)
    pagination.total = data.meta ? data.meta.total : items.length
  } catch (e) {
    alert('Error al cargar los libros')
  }
  loading.value = false
}

function onPageChange(page) {
  pagination.page = page
  fetchLibros()
}

function openDialog(item) {
  if (item) Object.assign(form, { ...item })
  else Object.assign(form, {
    id: undefined,
    titulo: '',
    autor: '',
    anioPublicacion: '',
    genero: '',
    activo: true,
  })
  dialog.value = true
}

function closeDialog() {
  dialog.value = false
  formRef.value?.resetValidation()
}

async function saveLibro() {
  if (!formRef.value) return
  const body = {
    titulo: form.titulo,
    autor: form.autor,
    anioPublicacion: form.anioPublicacion,
    genero: form.genero,
  }
  const url = form.id ? `${API_BASE}/libros/${form.id}` : `${API_BASE}/libros`
  const method = form.id ? 'PUT' : 'POST'
  const response = await fetch(url, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  })
  if (!response.ok) {
    const err = await response.json()
    alert(err.message || 'Error al guardar libro')
    return
  }
  await fetchLibros()
  closeDialog()
}

async function deleteLibro(id) {
  if (!confirm('¿Seguro que deseas inactivar este libro?')) return
  const response = await fetch(`${API_BASE}/libros/${id}`, { method: 'DELETE' })
  if (!response.ok) return alert('Error al inactivar libro')
  await fetchLibros()
}

async function restoreLibro(id) {
  if (!confirm('¿Restaurar este libro?')) return
  const response = await fetch(`${API_BASE}/libros/${id}/restore`, { method: 'POST' })
  if (!response.ok) return alert('Error al restaurar libro')
  await fetchLibros()
}

async function forceDeleteLibro(id) {
  if (!confirm('¿Eliminar definitivamente este libro?')) return
  const response = await fetch(`${API_BASE}/libros/${id}/force`, { method: 'DELETE' })
  if (!response.ok) return alert('Error al eliminar definitivamente')
  await fetchLibros()
}

watch([() => filters.titulo, () => filters.autor, () => filters.genero], () => {
  pagination.page = 1
  fetchLibros()
})

watch(tab, () => {
  pagination.page = 1
  fetchLibros()
})

onMounted(fetchLibros)
</script>