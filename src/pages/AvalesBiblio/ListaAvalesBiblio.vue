<template>
  <div class="q-pa-lg">
    <q-table
      bordered
      dense
      virtual-scroll
      title="Avales de Bibliografía"
      title-class="text-bold  "
      :rows="rows"
      :columns="columns"
      row-key="id"
      :filter="search"
      :sort="true"
      sortBy="fecha"
      sortDesc="true"
      no-data-label="No hay datos disponibles."
      no-results-label="No se encontraron resultados para tu búsqueda."
      :loading="isLoading"
      rows-per-page-options="7"
    >
      <template v-slot:loading>
        <q-inner-loading showing color="primary" />
      </template>
      <template v-slot:top-right>
        <div class="row q-gutter-md">
          <q-btn
            v-if="user.isAdmin"
            label="Nuevo Aval"
            icon="add"
            color="primary"
            size="sm"
            align="left"
            class="text-bold"
            dense
            to="/crear_aval_biblio"
            ><q-tooltip> Registrar Nuevo Aval</q-tooltip></q-btn
          >
          <q-btn
            icon="expand_more"
            label="Más Datos"
            color="primary"
            size="sm"
            align="left"
            dense
            to="/detallesbiblio"
            ><q-tooltip> Mostrar más Datos </q-tooltip></q-btn
          >
          <q-input dense outlined v-model="search" placeholder="Buscar" />
        </div>
      </template>

      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td v-for="col in props.cols" :key="col.name" :props="props">
            {{ col.value }}
          </q-td>

          <q-td auto-width class="q-gutter-sm">
            <q-btn
              color="primary"
              icon="visibility"
              size="sm"
              flat
              dense
              @click="showRow(props.row)"
            />
            <q-btn
              v-if="user.isViewerOnly || user.isAdmin"
              color="positive"
              icon="edit"
              size="sm"
              flat
              dense
              @click="editRow(props.row)"
            />
            <q-btn
              v-if="user.isViewerOnly || user.isAdmin"
              color="negative"
              icon="delete"
              size="sm"
              class="q-ml-sm"
              flat
              dense
              @click="eliminar(props.row)"
            />
          </q-td>
        </q-tr>
      </template>
    </q-table>
    <q-dialog v-model="editDialogOpen" backdrop-filter="blur(4px)">
      <q-card style="width: 500px; height: 500px">
        <q-card-section>
          <div class="text-h6">Editar Aval</div>
        </q-card-section>
        <q-separator inset />
        <q-card-section>
          <q-input autogrow v-model="editForm.nombre" label="Nombre" />
          <q-input autogrow v-model="editForm.apellidos" label="Apellidos" />

          <q-input
            v-model="editForm.departamento"
            label="Departamento"
            class="form-item"
            @click="showSelectorDepartamento = true"
          />
          <q-dialog v-model="showSelectorDepartamento" persistent>
            <SelectorDepartamento
              v-model="editForm.departamento"
              :open-first-dialog-automatically="true"
              @close-first-dialog="closeFirstDialogAndUpdateModel"
            />
          </q-dialog>
          <q-input
            v-model="editForm.total_asient"
            label="Total de Asientos Bibliográficos"
            class="form-item"
          />
          <q-input
            v-model="editForm.rev_bilio"
            label="Tipo de Revisión Bibliográfica"
            class="form-item"
            @click="showRevBiblioDialog = true"
          />
          <q-dialog v-model="showRevBiblioDialog" persistent>
            <selector-rev-biblio
              v-model="editForm.rev_bilio"
              :public-types="[
                'Maestría',
                'Doctorado',
                'Trabajo de Diplma',
                'Proyecto',
                'Curso de Posgrado',
                'Projecto de Curso',
                'Disciplina',
                'Asignatura',
                'Evaluación Ind. Prof',
              ]"
              :open-dialog-automatically="showRevBiblioDialog"
              @update:modelValue="editForm.rev_bilio = $event"
              @dialogClosed="hideRevBiblioDialog"
            />
          </q-dialog>
          <q-input
            v-model="editForm.niv_act"
            label="Nivel de Actualización"
            class="form-item"
            @click="showNivActDialog = true"
          />
          <q-dialog v-model="showNivActDialog" persistent>
            <selector-niv-act
              v-model="editForm.niv_act"
              :public-types="['Alto', 'Medio', 'Bajo']"
              :open-dialog-automatically="showNivActDialog"
              @update:modelValue="editForm.niv_act = $event"
              @dialogClosed="hideNivActDialog"
            />
          </q-dialog>

          <q-checkbox
            v-model="editForm.bd_local"
            label="Base de Datos Locales"
          />
          <q-checkbox v-model="editForm.cd_rom" label="CD ROM / DVD" />
          <q-checkbox
            v-model="editForm.bd_internet"
            label="Base de Datos en Internet"
          />
          <q-checkbox
            v-model="editForm.curso_pos_bus"
            label="Curso de Posgrado"
          />

          <q-checkbox
            v-model="editForm.busqueda_internet"
            label="Busqueda en Internet"
          />
          <q-checkbox
            v-model="editForm.biblio_personal"
            label="Bibliografía Personal"
          />
          <q-checkbox v-model="editForm.otros" label="Otros" />
          <q-checkbox
            v-model="editForm.no_biblio"
            label="No Existe Bibliografía"
          />

          <q-input v-model="editForm.pag" label="Página" />
          <q-input v-model="editForm.tomo" label="Tomo" />
          <q-input v-model="editForm.folio" label="Folio" />
        </q-card-section>

        <q-card-actions align="right">
          <q-btn rounded label="Cancelar" v-close-popup />
          <q-btn rounded color="primary" label="Guardar" @click="saveEdit" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive, watch, toRefs } from 'vue';
import { useRouter } from 'vue-router';
import { api } from 'src/boot/axios';
import SelectorDepartamento from 'src/components/SelectorDepartamento.vue';
import SelectorRevBiblio from 'src/components/SelectorRevBiblio.vue';
import SelectorNivAct from 'src/components/SelectorNivAct.vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();
const user = ref({ role: 'invitado', isAdmin: false, isViewerOnly: false });
const search = ref('');
const rows = ref<RowType[]>([]);
const selectedRow = ref<RowType | null>(null);
const router = useRouter();
const showSelectorDepartamento = ref(false);
const closeFirstDialogAndUpdateModel = () => {
  showSelectorDepartamento.value = false;
};
const showNivActDialog = ref(false);
const hideNivActDialog = () => {
  showNivActDialog.value = false;
};
const showRevBiblioDialog = ref(false);
const hideRevBiblioDialog = () => {
  showRevBiblioDialog.value = false;
};
const isLoading = ref(true);
type RowType = {
  id: number;
  nombre: string;
  apellidos: string;
  departamento: string;
  total_asient: string;
  rev_bilio: string;
  bd_local: boolean;
  cd_rom?: boolean;
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
};
const columns = [
  {
    name: 'nombre',
    required: true,
    label: 'Nombre ',
    align: 'left',
    field: 'nombre',
    filter: true,
    sortable: true,
  },
  {
    name: 'apellidos',
    required: true,
    label: ' Apellidos',
    field: 'apellidos',
    align: 'left',
    filter: true,
    sortable: true,
  },

  {
    name: 'departamento',
    label: 'Departamento de Trabajo',
    field: 'departamento',
    align: 'left',
    sortable: true,
    filter: true,
  },

  {
    name: 'rev_bilio',
    label: 'Tipo de Revisión',
    field: 'rev_bilio',
    align: 'left',
    sortable: true,
    filter: true,
  },
  {
    name: 'niv_act',
    label: 'Nivel de Actualizacion',
    field: 'niv_act',
    align: 'left',
    sortable: true,
    filter: true,
  },
  {
    name: 'fecha',
    label: 'Fecha de Publicación',
    field: 'fecha',
    align: 'left',
    sortable: true,
    filter: true,
  },
];

const fetchUserData = async () => {
  try {
    const authToken = localStorage.getItem('authToken');
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };

    const response = await api.get('/api/users', config);

    // Verificar si la petición fue exitosa
    if (response.status === 200) {
      user.value.role = response.data.role;
      user.value.isAdmin = response.data.role === 'admin';
      user.value.isViewerOnly = response.data.role === 'especialista';
      console.log('Datos del usuario obtenidos correctamente.');
    } else {
      console.error(
        `Error al obtener los datos del usuario: Estado ${response.status}`
      );
    }
  } catch (error) {
    console.error('Error al obtener los datos del usuario:', error);
  }
};
onMounted(async () => {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    const response = await api.get('/api/avales_biblio/', config);
    console.log('Formulario enviado con éxito:');
    rows.value = response.data.results;
  } catch (error) {
    console.error('Error al obtener los datos de los profesores:', error);
  }
  isLoading.value = false;
  fetchUserData();
});
const editDialogOpen = ref(false);
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

const editForm = reactive<Form>({
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

const { nombre, apellidos } = toRefs(editForm);
//metodos
function capitalizeWords(text: string): string {
  return text
    .split(/\s+/)
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}

//watchers
watch(
  nombre,
  (newValue) => {
    editForm.nombre = capitalizeWords(newValue);
  },
  { deep: true }
);

watch(
  apellidos,
  (newValue) => {
    editForm.apellidos = capitalizeWords(newValue);
  },
  { deep: true }
);

//boton editar
const editRow = (row: RowType) => {
  selectedRow.value = row;
  editForm.nombre = row.nombre;
  editForm.apellidos = row.apellidos;
  editForm.niv_act = row.niv_act;
  editForm.rev_bilio = row.rev_bilio;
  editForm.total_asient = row.total_asient;
  editForm.departamento = row.departamento;
  editForm.tomo = row.tomo;
  editForm.folio = row.folio;
  editForm.fecha = row.fecha;
  editForm.bd_local = row.bd_local;
  editForm.bd_internet = row.bd_internet;
  editForm.cd_rom = row.cd_rom || false;
  editForm.curso_pos_bus = row.curso_pos_bus;
  editForm.busqueda_internet = row.busqueda_internet;
  editForm.biblio_personal = row.biblio_personal;
  editForm.pag = row.pag;
  editForm.otros = row.otros;
  editForm.no_biblio = row.no_biblio;
  editDialogOpen.value = true;
};

const saveEdit = async () => {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    $q.loading.show();
    await api.put(
      `/api/avales_biblio/${selectedRow.value.id}/`,
      editForm,
      config
    );

    const index = rows.value.findIndex(
      (row) => row.id === selectedRow.value.id
    );
    if (index !== -1) {
      Object.assign(rows.value[index], editForm);
    }
    $q.loading.hide();
    console.log('Recurso actualizado con éxito');

    editDialogOpen.value = false;
  } catch (error) {
    $q.loading.hide();
    console.error('Error al actualizar el recurso:', error);
  }
  $q.notify({
    type: 'positive',
    message: '¡Aval Actualizado Correctamente!',
    position: 'bottom-right',
  });
};

//boton mostrar
const showRow = (row: null) => {
  console.log('Mostrando detalles del recurso:');
  router.push({ name: 'ShowBiblio', params: { id: row.id } });
};
// boton eliminar
async function eliminar(row: { id: null }) {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    await $q
      .dialog({
        title: 'Eliminar Aval',
        message: '¿Estás seguro de eliminar?',
        cancel: true,
        persistent: true,
      })
      .onOk(() => {
        api
          .delete(`/api/avales_biblio/${row.id}/`, config)
          .then(() => {
            console.log('Recurso eliminado con éxito');
            rows.value = rows.value.filter((item) => item.id !== row.id);
            $q.notify({
              type: 'positive', // Cambiado a positive para indicar éxito
              message: '¡Aval Eliminado Correctamente!',
              position: 'bottom-right',
            });
          })
          .catch((error) => {
            console.error('Error al eliminar el recurso:', error);
            $q.notify({
              type: 'negative',
              message: 'Hubo un error al eliminar el Aval.',
              position: 'bottom-right',
            });
          });
      });
  } catch (error) {
    console.error('Error al mostrar el diálogo:', error);
  }
}
</script>
