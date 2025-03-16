<template>
  <div class="q-pa-lg">
    <q-card class="rounded-card shadow-5">
      <q-card-section class="bg-gradient-primary text-white">
        <div class="text-h5 text-weight-bold q-pb-md">
          📊 Distribución de Avales por Facultad y Departamento
        </div>
        <q-separator color="white" />
      </q-card-section>

      <q-card-section>
        <div class="row q-col-gutter-lg">
          <div
            v-for="(departamentos, facultad) in avalesPorFacultad"
            :key="facultad"
            class="col-12 col-sm-6 col-md-4 col-lg-3"
          >
            <q-card class="facultad-card">
              <q-card-section class="bg-blue-1 text-primary">
                <div class="row items-center">
                  <q-icon name="school" size="sm" class="q-mr-sm" />
                  <div class="text-h6 text-weight-bold">{{ facultad }}</div>
                  <q-space />
                  <q-badge rounded color="primary">
                    Total: {{ departamentos.total || 0 }}
                  </q-badge>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-pt-none">
                <q-list>
                  <q-item
                    v-for="depto in Object.keys(departamentos).filter(
                      (k) => k !== 'total'
                    )"
                    :key="depto"
                    class="q-py-xs"
                  >
                    <q-item-section>
                      <div class="row items-center">
                        <q-icon
                          name="apartment"
                          size="xs"
                          color="grey-6"
                          class="q-mr-sm"
                        />
                        <div class="text-caption text-weight-medium">
                          {{ depto }}
                        </div>
                      </div>
                    </q-item-section>

                    <q-item-section side>
                      <q-badge color="green" class="text-weight-bold">
                        {{ departamentos[depto] }}
                      </q-badge>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <div
          v-if="!Object.keys(avalesPorFacultad).length"
          class="text-center q-pa-xl"
        >
          <q-icon name="warning" size="xl" color="grey-5" />
          <div class="text-h6 text-grey-5 q-mt-md">
            No hay datos disponibles
          </div>
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
<style scoped>
.bg-gradient-primary {
  background: linear-gradient(145deg, #2c387e, #2196f3);
}

.facultad-card {
  transition: transform 0.3s ease;
  height: 100%;
}

.facultad-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.q-item:hover {
  background-color: rgba(0, 0, 0, 0.03);
}
</style>
