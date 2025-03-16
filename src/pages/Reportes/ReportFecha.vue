<template>
  <div class="q-pa-lg column items-center q-gutter-y-xl">
    <!-- Mejora en el contenedor principal -->
    <q-expansion-item
      v-for="(resultado, index) in resultados"
      :key="index"
      class="shadow-3 overflow-hidden"
      style="max-width: 800px; width: 100%"
      header-class="bg-primary text-white"
      expand-icon-class="text-white"
      :duration="300"
    >
      <!-- Header rediseñado -->
      <template v-slot:header>
        <q-item-section class="q-pa-md">
          <div class="row items-center justify-between q-col-gutter-x-lg">
            <div class="col">
              <div class="text-h6 text-weight-bold">{{ resultado.fecha }}</div>
              <div class="text-caption">Fecha de reporte</div>
            </div>

            <div class="col-auto">
              <q-badge class="text-h6 q-px-md q-py-sm">
                {{ resultado.total }} Avales
              </q-badge>
            </div>
          </div>
        </q-item-section>
      </template>

      <!-- Contenido mejorado -->
      <q-card class="q-pa-md bg-color">
        <q-list bordered separator class="rounded-borders">
          <q-item
            v-for="(count, departamento) in resultado.departamentos"
            :key="departamento"
            class="q-my-xs rounded-borders"
          >
            <q-item-section avatar>
              <q-icon name="location_city" color="primary" />
            </q-item-section>

            <q-item-section>
              <q-item-label class="text-weight-medium">{{
                departamento
              }}</q-item-label>
            </q-item-section>

            <q-item-section side>
              <q-badge color="primary" class="q-px-sm">
                {{ count }}
              </q-badge>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card>
    </q-expansion-item>

    <!-- Estado vacío -->
    <q-card v-if="resultados.length === 0" class="q-pa-xl text-center">
      <div class="text-h6 text-grey-6">No hay resultados disponibles</div>
    </q-card>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { api } from 'src/boot/axios';
import { useQuasar } from 'quasar';

const resultados = ref([]);

const $q = useQuasar();

// Agregar skeleton loading
const loading = ref(true);

onMounted(async () => {
  try {
    const authToken = localStorage.getItem('authToken');
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    $q.loading.show();
    const response = await api.get(
      '/api/reporte-total-avaless-por-fecha/',
      config
    );
    resultados.value = response.data;
    $q.loading.hide();
  } catch (error) {
    $q.loading.hide();
    console.error('Error al cargar los datos:', error);
  } finally {
    loading.value = false;
  }
});
</script>
<style scoped>
/* Animación suave para la expansión */
.q-expansion-item {
  transition: all 0.3s ease;
  border-radius: 12px;
}

/* Mejor aspecto del gradiente */
.bg-gradient-primary {
  background: linear-gradient(145deg, var(--q-primary), #1a237e);
}

/* Efecto hover para los ítems */
.q-item:hover {
  background-color: rgba(0, 0, 0, 0.03);
  transform: translateX(5px);
  transition: all 0.2s ease;
}
</style>
