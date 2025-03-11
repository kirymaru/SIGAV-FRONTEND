<template>
  <div class="q-pa-lg">
    <q-card class="column q-gutter-y-md rounded-card">
      <div class="column text-subtitle1 text-bold">
        <q-card-section>
          <div class="text-h6 text-bold">
            Cantidad de Avales por Departamento y Tipo de Aval :
          </div>
          <q-separator />
        </q-card-section>
        <q-card-section>
          <div>
            <p class="text-bold text-body2">Departamento de Trabajo</p>
          </div>
          <q-input
            style="max-width: 250px"
            autogrow
            outlined
            dense
            v-model="departamentoSeleccionado"
            label="Seleccione un Departamento"
            @click="showSelectorDepartamento = true"
            @update:model-value="mostrarAvales"
          />
          <q-dialog v-model="showSelectorDepartamento" persistent>
            <SelectorDepartamento
              v-model="departamentoSeleccionado"
              :open-first-dialog-automatically="true"
              @close-first-dialog="closeFirstDialogAndUpdateModel"
              @update:model-value="mostrarAvales"
            />
          </q-dialog>
        </q-card-section>
      </div>
      <q-separator />
      <q-card-section>
        <div class="row justify-around text-body1 text-bold">
          <div>
            Aval de Publicación: {{ datos?.avales_por_tipo['Profesor'] }}
          </div>
          <div>
            Aval de Tutoría: {{ datos?.avales_por_tipo['avales_tuto'] }}
          </div>
          <div>
            Aval de Bibliografía: {{ datos?.avales_por_tipo['avales_biblio'] }}
          </div>
          <div>Total: {{ datos?.total_avales }}</div>
        </div>
      </q-card-section>
    </q-card>
    <q-card class="rounded-card">
      <q-card-section>
        <div class="text-h6 text-bold">
          Cantidad de Avales por Departamento y Facultad:
        </div>
        <q-separator />
      </q-card-section>
      <q-card-section class="flex-container">
        <div
          class="flex-item"
          v-for="(departamentos, facultad) in avalesPorFacultad"
          :key="facultad"
        >
          <strong>{{ facultad }}:</strong>
          <ul class="text-bold">
            <li
              v-for="departamento in Object.keys(departamentos)"
              :key="departamento"
            >
              {{ departamento }}: {{ departamentos[departamento] || 0 }}
            </li>
          </ul>
        </div>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, reactive, watch } from 'vue';
import { api } from 'src/boot/axios';
import SelectorDepartamento from 'src/components/SelectorDepartamento.vue';
import { useQuasar } from 'quasar';

interface avalesPorFacultad {
  [facultad: string]: {
    [departamento: string]: number;
  };
}

let departamentoSeleccionado = ref('');

const showSelectorDepartamento = ref(false);
const closeFirstDialogAndUpdateModel = () => {
  showSelectorDepartamento.value = false;
};
const $q = useQuasar();

interface Datos {
  total_avales: number;
  avales_por_tipo: Record<string, number>;
}
const datos = ref<Datos | null>(null);
interface AvaPorDepartamento {
  [departamento: string]: number;
}

const avalesPorDepartamento: AvaPorDepartamento = reactive({});
const avalesPorFacultad: avalesPorFacultad = reactive({});
const departamentosPorFacultad = reactive({
  'Ciencias Sociales': [
    'Derecho',
    'Estudios Socioculturales',
    'Psicología - Sociología',
  ],
  'Ciencias Aplicadas': [
    'Alimentos',
    'Educación Biología',
    'Educación Geografía',
    'Ingeniería Química',
    'Ingeniería Industrial',
    'Química',
    'CEECE',
    'CEGEA',
  ],
  'Ciencias Agropecuarias': [
    'Agronomía',
    'Educación Agropecuaria',
    'Morfofisiología',
    'Medicina Veterinaria',
    'CEDEPA',
  ],
  'Ciencias Económicas': [
    'Contabilidad',
    'Economía',
    'Educación Economía',
    'Turismo',
    'CEMTUR',
    'CEDET',
  ],
  Electromecánica: [
    'Educación Electromecánica',
    'Ingeniería Eléctrica',
    'Ingeniería Mecánica',
    'CEEFREP',
  ],
  Construcciones: [
    'Arquitectura',
    'Educación Construcción',
    'Ingeniería Civil',
    'CECODEC',
  ],
  'Lengua y Comunicación': [
    'Español',
    'Lenguas Extranjeras',
    'Periodismo y Comunicación Social',
    'Centro de Idiomas',
  ],
  'Informática y Ciencias Exactas': [
    'Educación Laboral e Informática',
    'Ciencias de la Información',
    'Ingeniería Informática',
    'Física',
    'Matemática',
  ],
  'Ciencias Pedagógicas': [
    'Educación Artística',
    'Educación Especial',
    'Educación Pedagogía - Psicología',
    'Formación Pedagógica General',
    'Educación Preescolar',
    'Educación Primaria',
  ],
  'Cultura Física': [
    'Cultura Física',
    'Ciencias Aplicadas al Deporte',
    'Didáctica del Deporte',
    'Educación Física y Recreación',
    'CEAFIDE',
  ],
  CUM: [
    'Céspedes',
    'Esmeralda',
    'Florida',
    'Guáimaro',
    'Jimaguayú',
    'Minas',
    'Najasa',
    'Nuevitas',
    'Santa Cruz del Sur',
    'Sibanicú',
    'Sierra de Cubitas',
    'Vertientes',
  ],
});

///////////consultas////////
async function cargarDatos() {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    const response = await api.get(
      `/api/reporte-departamento/${departamentoSeleccionado.value}/`,
      config
    );

    datos.value = response.data;
  } catch (error) {
    console.error('Error al cargar los datos del endpoint:', error);
  }
}

onMounted(async () => {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    $q.loading.show();
    const response = await api.get(
      '/api/reporte-total-avaless-por-departamento/',
      config
    );
    const data = response.data;

    Object.keys(data).forEach((departamento) => {
      const facultad = obtenerFacultadPorDepartamento(departamento);

      avalesPorDepartamento[departamento] = data[departamento];

      if (!avalesPorFacultad[facultad]) {
        avalesPorFacultad[facultad] = {};
      }
      avalesPorFacultad[facultad][departamento] = data[departamento];
    });

    Object.keys(avalesPorFacultad).forEach((facultad) => {
      let sumaTotal = 0;
      Object.values(avalesPorFacultad[facultad]).forEach((cantidad) => {
        sumaTotal += cantidad;
      });

      if (!avalesPorFacultad[facultad].total) {
        avalesPorFacultad[facultad].total = 0;
      }
      avalesPorFacultad[facultad].total = sumaTotal;
    });
    cargarDatos();
    $q.loading.hide();
  } catch (error) {
    $q.loading.hide();
    console.error('Error al cargar los datos del endpoint:', error);
  }
});

/////////////////////// Funciónes
function mostrarAvales() {
  if (departamentoSeleccionado.value) {
    cargarDatos();
  }
}

function obtenerFacultadPorDepartamento(departamento: string): string | '' {
  for (const [facultad, departamentos] of Object.entries(
    departamentosPorFacultad
  )) {
    if (departamentos.includes(departamento)) {
      return facultad;
    }
  }

  return '';
}

///watchers////////////////////////////////////////////////
watch(departamentoSeleccionado, async (nuevoValor, antiguoValor) => {
  if (nuevoValor !== antiguoValor) {
    await mostrarAvales();
  }
});
</script>
<style scoped></style>
