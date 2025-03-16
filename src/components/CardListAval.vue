<template>
  <div class="row items-baseline q-col-gutter-md">
    <div class="col-2">
      <div class="column q-gutter-y-md">
        <q-card
          v-for="(aval, tipo) in tiposDeAval"
          :key="tipo"
          class="column justify-around"
        >
          <q-card-section class="text-bold text-white text-color">
            {{ aval.titulo }}:
          </q-card-section>
          <q-card-section>
            <div>
              <div class="row justify-center text-h4 text-bold">
                <div>
                  {{ datos?.avales_por_tipo[tipo] || '-' }}
                </div>
              </div>
            </div>
          </q-card-section>
        </q-card>
        <q-card>
          <q-card-section class="text-bold text-white text-color">
            Total de Avales:
          </q-card-section>
          <q-card-section>
            <div>
              <div class="row justify-center text-h4 text-bold">
                <div>
                  {{ datos?.total_avales || '-' }}
                </div>
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>
    <div class="col-10">
      <q-card class="full-height">
        <q-card-section class="text-h6 text-primary">
          Últimos 5 Avales Registrados
        </q-card-section>
        <q-separator />
        <q-card-section class="q-pt-none">
          <div class="row q-col-gutter-sm">
            <div v-for="aval in ultimosAvales" :key="aval.id" class="col-12">
              <q-card @click="showRow(aval)" class="rounded-card">
                <q-card-section class="q-pa-sm">
                  <div class="row items-center q-col-gutter-sm">
                    <div class="text-bold text-primary">
                      {{ aval.tipo_aval }}
                    </div>
                    <q-separator vertical inset />
                    <div class="text-subtitle2">
                      {{ aval.fecha ? formatDate(aval.fecha) : 'Sin fecha' }}
                    </div>
                  </div>
                </q-card-section>
                <q-separator />
                <q-card-section class="q-pa-sm">
                  <div class="text-semibold text-grey-7">
                    Autor: {{ aval.nombre }} {{ aval.apellidos }}
                  </div>
                  <div class="text-semibold text-grey-7">
                    Título:
                    {{ aval.titulo_recurso }}
                  </div>
                </q-card-section>
              </q-card>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { api } from 'src/boot/axios';
import { useQuasar } from 'quasar';
import { useRoute, useRouter } from 'vue-router';

const router = useRouter();

// Interfaces
interface Datos {
  total_avales: number;
  avales_por_tipo: Record<string, number>;
}

interface TipoAval {
  titulo: string;
}

interface Aval {
  id: number;
  tipo_aval: string;
  fecha: string | null;
  [key: string]: any;
}

// Estado para los datos de avales por tipo
const tiposDeAval: Record<string, TipoAval> = {
  Profesor: { titulo: 'Aval de Publicación' },
  avales_tuto: { titulo: 'Aval de Tutoría' },
  avales_biblio: { titulo: 'Aval de Bibliografía' },
};

const datos = ref<Datos | null>(null);
const loading = ref(false);

// Estado para los últimos avales
const ultimosAvales = ref<Aval[]>([]);

const $q = useQuasar();

// Funciones para cargar datos
async function cargarDatos() {
  try {
    loading.value = true;
    const authToken = localStorage.getItem('authToken');
    if (!authToken) throw new Error('Token de autenticación no encontrado');

    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };

    const response = await api.get(`/api/reporte-departamento//`, config);
    if (!response.data) throw new Error('No se recibieron datos válidos');

    datos.value = {
      ...response.data,
      avales_por_tipo: response.data.avales_por_tipo || {},
    };
  } catch (error) {
    console.error('Error al cargar los datos:', error);
    $q.notify({
      type: 'negative',
      message: 'Error al cargar los datos del servidor',
    });
  } finally {
    loading.value = false;
  }
}

async function cargarUltimosAvales() {
  try {
    const authToken = localStorage.getItem('authToken');
    if (!authToken) throw new Error('Token de autenticación no encontrado');

    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };

    const response = await api.get('/api/ultimos-avales/', config);
    if (!response.data) throw new Error('No se recibieron datos válidos');

    ultimosAvales.value = response.data;
    console.log(ultimosAvales);
  } catch (error) {
    console.error('Error al cargar los últimos avales:', error);
    $q.notify({
      type: 'negative',
      message: 'Error al cargar los últimos avales del servidor',
    });
  }
}

// Funciones de utilidad

function formatDate(fecha: string): string {
  return new Date(fecha).toLocaleDateString('es-ES', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
}

const showRow = (aval: any) => {
  let routeName;
  if (aval.tipo_aval === 'Aval de Publicación') {
    routeName = 'show';
  } else if (aval.tipo_aval === 'Aval de Tutoría') {
    routeName = 'ShowTuto';
  } else if (aval.tipo_aval === 'Aval de Bibliografía') {
    routeName = 'ShowBiblio';
  }
  router.push({
    name: routeName,
    params: {
      id: aval.id,
    },
  });
};

// Ciclo de vida
onMounted(() => {
  cargarDatos();
  cargarUltimosAvales();
});

// Exponer funciones
defineExpose({ cargarDatos, cargarUltimosAvales });
</script>

<style scoped>
.rounded-card {
  border-radius: 15px;
}
</style>
