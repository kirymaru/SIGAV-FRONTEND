<template>
  <q-page>
    <div
      class="row justify-around"
      style="margin-top: 30px; margin-bottom: 30px"
    >
      <div>
        <p class="text-bold text-body1">Aval de Bibliografía</p>
        <p class="text-bold text-body2" style="color: grey">
          Introduce los datos del aval
        </p>
      </div>
      <q-form @submit="onSubmit" id="form">
        <div class="q-gutter-md row justify-center items-center">
          <div class="q-gutter-xl q-gutter-y-md row justify-around">
            <div
              class="column q-gutter-md-y-sm suggested-authors-list-container"
            >
              <div><p class="text-bold text-body2">Nombre</p></div>
              <q-input
                style="width: 200px"
                autogrow
                outlined
                dense
                v-model="form.nombre"
                label="Ej: Pedro"
                class="form-item"
                :rules="nombreRules"
              />
              <ul
                style="max-height: 150px"
                v-if="autoresSugeridos.length > 0"
                class="suggested-authors-list"
              >
                <li
                  v-for="autor in autoresSugeridos"
                  :key="autor.id"
                  @click="selectAuthor(autor)"
                >
                  {{ autor.nombre }}
                  {{ autor.apellidos }}
                </li>
              </ul>
            </div>
            <div class="column q-gutter-md-y-sm">
              <div><p class="text-bold text-body2">Apellidos</p></div>
              <q-input
                style="width: 200px"
                autogrow
                outlined
                dense
                v-model="form.apellidos"
                label="Ej: Rodriguez"
                class="form-item"
                :rules="apellidosRules"
              />
            </div>
            <div class="column q-gutter-md-y-sm">
              <div>
                <p class="text-bold text-body2">Departamento de Trabajo</p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.departamento"
                label="Selecciona un Departamento"
                class="form-item"
                @click="showSelectorDepartamento = true"
                :rules="departamentoRules"
              />
              <q-dialog v-model="showSelectorDepartamento" persistent>
                <SelectorDepartamento
                  v-model="form.departamento"
                  :departamento-rules="departamentoRules"
                  :open-first-dialog-automatically="true"
                  @close-first-dialog="closeFirstDialogAndUpdateModel"
                />
              </q-dialog>
            </div>
          </div>
          <div class="q-gutter-xl row justify-center items-center">
            <div>
              <div>
                <p class="text-bold text-body2">
                  Total de Asientos Bibliográficos
                </p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.total_asient"
                label="Introduce el Total de Asientos "
                class="form-item"
                :rules="[
                  (val) =>
                    (val && val.trim().length > 0) ||
                    'Total de Asientos es requerido',
                  (val) => /^\d+$/.test(val) || 'Solo se permiten números',
                ]"
              />
            </div>
            <div>
              <div>
                <p class="text-bold text-body2">
                  Tipo de Revisión Bibliográfica
                </p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.rev_bilio"
                label="Selecciona el Tipo de Revisión  "
                class="form-item"
                @click="showRevBiblioDialog = true"
                :rules="revRules"
              />
              <q-dialog v-model="showRevBiblioDialog" persistent>
                <selector-rev-biblio
                  v-model="form.rev_bilio"
                  :public-types="[
                    'Maestría',
                    'Doctorado',
                    'Trabajo de Diploma',
                    'Proyecto',
                    'Curso de Posgrado',
                    'Proyecto de Curso',
                    'Disciplina',
                    'Asignatura',
                    'Evaluación Ind. Prof',
                  ]"
                  :open-dialog-automatically="showRevBiblioDialog"
                  @update:modelValue="form.rev_bilio = $event"
                  @dialogClosed="hideRevBiblioDialog"
                />
              </q-dialog>
            </div>
            <div>
              <div>
                <p class="text-bold text-body2">Nivel de Actualización</p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.niv_act"
                label="Selecciona el Nivel de Actualización"
                class="form-item"
                @click="showNivActDialog = true"
                :rules="revRules"
              />
              <q-dialog v-model="showNivActDialog" persistent>
                <selector-niv-act
                  v-model="form.niv_act"
                  :public-types="['Alto', 'Medio', 'Bajo']"
                  :open-dialog-automatically="showNivActDialog"
                  @update:modelValue="form.niv_act = $event"
                  @dialogClosed="hideNivActDialog"
                />
              </q-dialog>
            </div>
          </div>

          <div class="q-gutter-xl row justify-center items-center">
            <div>
              <div>
                <p class="text-bold text-body2" style="width: 200px">Tomo</p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.tomo"
                label="Introduce el Tomo"
                :rules="[
                  (val) =>
                    (val && val.trim().length > 0) || 'Tomo es requerido',
                  (val) => /^\d+$/.test(val) || 'Solo se permiten números',
                ]"
              />
            </div>
            <div>
              <div>
                <p class="text-bold text-body2" style="width: 200px">Folio</p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.folio"
                label="Introduce el Folio"
                class="form-item"
                :rules="[
                  (val) =>
                    (val && val.trim().length > 0) || 'Folio es requerido',
                  (val) => /^\d+$/.test(val) || 'Solo se permiten números',
                ]"
              />
            </div>
            <div>
              <div>
                <p class="text-bold text-body2" style="width: 200px">Página</p>
              </div>

              <q-input
                style="width: 200px"
                outlined
                dense
                v-model="form.pag"
                label="Introduce la Página"
                class="form-item"
                :rules="[
                  (val) =>
                    (val && val.trim().length > 0) || 'Página es requerida',
                  (val) => /^\d+$/.test(val) || 'Solo se permiten números',
                ]"
              />
            </div>
            <div>
              <div>
                <p class="text-bold text-body2" style="width: 200px">Fecha</p>
              </div>
              <q-input
                style="width: 200px"
                outlined
                dense
                readonly
                v-model="form.fecha"
                label="Selecciona una Fecha "
                :rules="fechaRules"
              >
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                    >
                      <q-date v-model="form.fecha" mask="YYYY-MM-DD">
                        <div class="row items-center justify-end">
                          <q-btn v-close-popup label="Cerrar" color="primary" />
                        </div>
                      </q-date>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>
          </div>
        </div>
        <div class="column items-center">
          <h6>Tipo de Búsqueda Informativa</h6>
          <div class="row q-gutter-xl">
            <div class="column">
              <q-checkbox
                v-model="form.bd_local"
                label="Base de Datos Locales"
              />
              <q-checkbox v-model="form.cd_rom" label="CD ROM / DVD" />
              <q-checkbox
                v-model="form.bd_internet"
                label="Base de Datos en Internet"
              />
              <q-checkbox
                v-model="form.curso_pos_bus"
                label="Curso de Posgrado"
              />
            </div>
            <div class="column">
              <q-checkbox
                v-model="form.busqueda_internet"
                label="Búsqueda en Internet"
              />
              <q-checkbox
                v-model="form.biblio_personal"
                label="Bibliografía Personal"
              />
              <q-checkbox v-model="form.otros" label="Otros" />
              <q-checkbox
                v-model="form.no_biblio"
                label="No Existe Bibliografía"
              />
            </div>
          </div>
        </div>
        <q-separator inset class="container" />
        <div class="row justify-end items-center">
          <q-btn
            icon="arrow_back"
            rounded
            size="sm"
            label="Volver"
            class="form-item text-weight-bolder"
            color="primary"
            style="margin-top: 20px; margin-bottom: 20px; margin-right: 10px"
            @click="goBack"
          />
          <q-btn
            icon="save"
            rounded
            size="sm"
            label="Guardar"
            type="submit"
            class="form-item text-weight-bolder"
            color="primary"
            style="margin-top: 20px; margin-bottom: 20px"
          />
        </div>
      </q-form>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue';
import { api } from 'src/boot/axios';
import { useRouter } from 'vue-router';
import SelectorDepartamento from 'src/components/SelectorDepartamento.vue';
import SelectorRevBiblio from 'src/components/SelectorRevBiblio.vue';
import SelectorNivAct from 'src/components/SelectorNivAct.vue';
import { useQuasar } from 'quasar';

interface Form {
  nombre: string;
  apellidos: string;
  departamento: string;
  total_asient: string;
  rev_bilio: string;
  bd_local: boolean;
  cd_rom: boolean;
  bd_internet: boolean;
  curso_pos_bus: boolean;
  busqueda_internet: boolean;
  biblio_personal: boolean;
  otros: boolean;
  no_biblio: boolean;
  tomo: string;
  pag: string;
  folio: string;
  fecha: string;
  niv_act: string;
}

// Definición de tipos para reglas de validación
type Rule = (value: string) => boolean | string;

// Inicialización del estado reactivo
const form = reactive<Form>({
  nombre: '',
  apellidos: '',
  departamento: '',
  rev_bilio: '',
  niv_act: '',
  total_asient: '',
  bd_local: false,
  cd_rom: false,
  bd_internet: false,
  curso_pos_bus: false,
  busqueda_internet: false,
  biblio_personal: false,
  otros: false,
  no_biblio: false,
  tomo: '',
  pag: '',
  folio: '',
  fecha: '',
});

const showNivActDialog = ref(false);
const hideNivActDialog = () => {
  showNivActDialog.value = false;
};
const router = useRouter();
const showRevBiblioDialog = ref(false);
const hideRevBiblioDialog = () => {
  showRevBiblioDialog.value = false;
};
let isSearching = true;
const $q = useQuasar();
const showSelectorDepartamento = ref(false);
const closeFirstDialogAndUpdateModel = () => {
  showSelectorDepartamento.value = false;
};
const goBack = () => {
  router.back();
};
const autoresSugeridos = ref([]);
const mostrarAutoresSugeridos = ref(false);

//metodos
function capitalizeWords(text: string): string {
  return text
    .split(/\s+/)
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}

//watchers
watch(
  () => form.nombre,
  (newValue) => {
    form.nombre = capitalizeWords(newValue);
  },
  { deep: true }
);

watch(
  () => form.apellidos,
  (newValue) => {
    form.apellidos = capitalizeWords(newValue);
  },
  { deep: true }
);
watch(
  () => form.nombre,
  async (newVal, oldVal) => {
    if (isSearching === true && newVal.length >= 3) {
      await buscarAutores();
    } else {
      autoresSugeridos.value = [];
      mostrarAutoresSugeridos.value = false;
    }
  },
  { immediate: true }
);
watch(
  () => form.apellidos,
  async (newVal, oldVal) => {
    if (isSearching === true && form.nombre.length >= 3) {
      await buscarAutores();
    } else {
      autoresSugeridos.value = [];
      mostrarAutoresSugeridos.value = false;
    }
  },
  { immediate: true }
);
// Reglas de validación
const nombreRules: Rule[] = [
  (v) => !!v || 'El Nombre es requerido',
  (v) => v.length <= 50 || 'El nombre excede el límite de 100 caracteres',
];
const revRules: Rule[] = [(v) => !!v || 'El dato es requerido'];
const apellidosRules: Rule[] = [
  (v) => !!v || 'Los Apellidos son requeridos',
  (v) => v.length <= 50 || 'Los apellidos exceden el límite de 100 caracteres',
];

const departamentoRules: Array<(value: string) => boolean | string> = [
  (v) => !!v || 'El Departamento de Trabajo es requerido',
];
const fechaRules: Rule[] = [(v) => !!v || 'La Fecha es requerida'];

//peticioens
function onSubmit() {
  const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
  const config = {
    headers: {
      Authorization: `Token ${authToken}`,
      'Content-Type': 'application/json',
    },
  };
  if (!form.nombre || !form.apellidos) {
    errorMessage.value = 'Por favor, completa todos los campos requeridos.';
    return;
  }
  $q.loading.show();
  api
    .post('/api/avales_biblio/', form, config)
    .then((response) => {
      router.push({ name: 'ListaAvalesBiblio' });
      $q.loading.hide();
    })
    .catch((error) => {
      if (error.response && error.response.status === 400) {
        $q.loading.hide();
        $q.notify({
          type: 'negative',
          message: 'Hubo un error al enviar el formulario.',
          position: 'bottom-right',
        });
      } else {
        $q.loading.hide();
        $q.notify({
          type: 'negative',
          message:
            'Hubo un error al enviar el formulario. Por favor, inténtalo de nuevo.',
          position: 'bottom-right',
        });
      }
      console.error('Error al enviar el formulario:', error);
    });
  $q.notify({
    type: 'positive',
    message: '¡Aval Registrado con Éxito !',
    position: 'bottom-right',
  });
}
async function buscarAutores() {
  // Concatena nombre y apellidos con un espacio entre ellos
  const terminoNombre = form.nombre;
  const terminoApellidos = form.apellidos;

  // Verifica si la longitud del término de búsqueda es mayor o igual a 3
  if (terminoNombre.length >= 3 || terminoApellidos.length >= 3) {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
      params: { nombre: terminoNombre, apellidos: terminoApellidos }, // Usa el término de búsqueda concatenado y ajustado aquí
    };
    try {
      const response = await api.get('/api/autores/buscar', config); // Usando axios.get con config
      autoresSugeridos.value = response.data;
      mostrarAutoresSugeridos.value = true;
    } catch (error) {
      console.error('Error buscando autores:', error);
    }
  } else {
    autoresSugeridos.value = [];
    mostrarAutoresSugeridos.value = false;
  }
}

function selectAuthor(author) {
  if (author) {
    form.nombre = author.nombre;
    form.apellidos = author.apellidos;
    form.departamento = author.departamento;
    autoresSugeridos.value = [];
    mostrarAutoresSugeridos.value = false;
    isSearching = false; // Detiene la búsqueda
  }
}
</script>
