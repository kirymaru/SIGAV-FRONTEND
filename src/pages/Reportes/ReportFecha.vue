<template>
  <div
    style="padding: 30px"
    autogrow
    class="column justify-around items-center q-gutter-md"
  >
    <q-expansion-item
      v-for="(resultados, index) in resultados"
      :key="`${index}`"
      class="shadow-1 overflow-hidden"
      style="border-radius: 30px; width: 800px"
      icon="explore"
      label="Departamentos:"
      header-class="bg-secondary text-black"
      expand-icon-class="text-white "
    >
      <template v-slot:header>
        <q-item-section>
          <div class="row justify-around">
            <div class="text-bold text-h6">{{ resultados.fecha }}</div>
            <div class="text-bold text-h6">
              Total de Avales: {{ resultados.total }}
            </div>
          </div>
        </q-item-section>
      </template>

      <ul>
        <li
          v-for="(count, departamento) in resultados.departamentos"
          :key="departamento"
        >
          {{ departamento }}: {{ count }}
        </li>
      </ul>
    </q-expansion-item>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { api } from 'src/boot/axios';
import { useQuasar } from 'quasar';

const resultados = ref([]);

const $q = useQuasar();

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
  }
});
</script>
