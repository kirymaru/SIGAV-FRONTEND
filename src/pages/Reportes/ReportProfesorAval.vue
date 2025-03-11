<template>
  <div class="q-pa-lg">
    <q-table
      bordered
      dense
      :title="'Avales Registrados de: ' + nombre + ' ' + apellidos"
      title-class="text-bold"
      :rows="rows"
      :columns="columnas"
      row-key="id"
      :filter="search"
      class="q-mt-md"
      no-data-label="No hay datos disponibles."
      no-results-label="No se encontraron resultados para tu búsqueda."
      :loading="isLoading"
      loading-label="Cargando..."
      rows-per-page-options="8"
    >
      <template v-slot:loading>
        <q-inner-loading showing color="primary" />
      </template>
      <template v-slot:top-right>
        <div class="row q-gutter-md">
          <q-btn
            icon="arrow_back"
            label="Volver"
            color="primary"
            size="sm"
            align="left"
            class="text-bold"
            dense
            @click="goBack"
          />
          <q-input dense outlined v-model="search" placeholder="Buscar" /></div
      ></template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td v-for="col in props.cols" :key="col.name" :props="props">
            {{ col.value }}
          </q-td>

          <q-td auto-width>
            <q-btn
              color="primary"
              icon="visibility"
              size="sm"
              flat
              dense
              @click="showRow(props.row)"
            />
          </q-td>
        </q-tr>
      </template>
    </q-table>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { api } from 'src/boot/axios';
import { useRoute, useRouter } from 'vue-router';

const isLoading = ref(true);
const search = ref('');
const route = useRoute();
const router = useRouter();
const id = route.params.id;
const nombre = route.params.nombre;
const goBack = () => {
  router.back();
};
const apellidos = route.params.apellidos;
const rows = ref<RowType[]>([]);
type RowType = {
  id: number;
  titulo_recurso: string;
  tipo_aval: string;
  rev_bilio: string;
};

const columnas = [
  {
    name: 'tipo_aval',
    label: 'Tipo de Aval',
    align: 'left',
    field: 'tipo_aval',
    required: true,
    filter: true,
    sortable: true,
  },
  {
    name: 'titulo_recurso',
    label: 'Titulo del Recurso',
    field: 'titulo_recurso',
    align: 'left',
    sortable: true,
    filter: true,
    classes: 'texto-truncado',
  },
  {
    name: 'rev_bilio',
    label: 'Tipo de Revisión Bibliográfica',
    align: 'left',
    field: 'rev_bilio',
    required: true,
    filter: true,
    sortable: true,
  },
];
const showRow = (row: any) => {
  let routeName;
  if (row.tipo_aval === 'Aval de Publicación') {
    routeName = 'show';
  } else if (row.tipo_aval === 'Aval de Tutoría') {
    routeName = 'ShowTuto';
  } else if (row.tipo_aval === 'Aval de Bibliografía') {
    routeName = 'ShowBiblio';
  }
  router.push({
    name: routeName,
    params: {
      id: row.id,
    },
  });
};

const fetchData = async () => {
  try {
    const authToken = localStorage.getItem('authToken');
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    const result = await api.get(`/api/avales-profesor/${id}/`, config);
    rows.value = result.data;
    console.log(rows.value);
  } catch (error) {
    console.error('Error al obtener los detalles:', error);
  }
  isLoading.value = false;
};

onMounted(fetchData);
</script>
