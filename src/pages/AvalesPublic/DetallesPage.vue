<template>
  <div class="q-pa-lg">
    <q-table
      bordered
      dense
      virtual-scroll
      title="Avales de Publicación"
      title-class="text-bold "
      :rows="rows"
      :columns="columns"
      row-key="name"
      :filter="search"
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
            to="/crear_aval_public"
            ><q-tooltip> Registrar Nuevo Aval</q-tooltip></q-btn
          >
          <q-btn
            icon="expand_less"
            label="Menos Datos"
            color="primary"
            size="sm"
            align="left"
            dense
            to="/lista_avales_public"
            ><q-tooltip> Mostrar menos Datos </q-tooltip></q-btn
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
            autogrow
            v-model="editForm.titulo_recurso"
            label="Titulo del Recurso"
          />
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
            autogrow
            v-model="editForm.lugar_pub"
            label="Lugar de Publicacion"
          />

          <q-input
            v-model="editForm.tipo_recurso"
            label="Tipo de Recurso"
            class="form-item"
            @click="showTipoRecursoDialog = true"
          />
          <q-dialog v-model="showTipoRecursoDialog" persistent>
            <selector-tipo-recurso
              v-model="editForm.tipo_recurso"
              :resource-types="['Artículo', 'Libro', 'Capítulo', 'Epígrafe']"
              :open-dialog-automatically="showTipoRecursoDialog"
              @update:modelValue="editForm.tipo_recurso = $event"
              @dialogClosed="hideTipoRecursoDialog"
            />
          </q-dialog>
          <q-input
            v-model="editForm.tipo_publicacion"
            label="Tipo de Publicación"
            class="form-item"
            @click="showTipoPublicDialog = true"
          />
          <q-dialog v-model="showTipoPublicDialog" persistent>
            <selector-tipo-public
              v-model="editForm.tipo_publicacion"
              :public-types="[
                'Revista Impresa',
                'Revista Digital',
                'Libro Impreso',
                'Libro Digital',
              ]"
              :open-dialog-automatically="showTipoPublicDialog"
              @update:modelValue="editForm.tipo_publicacion = $event"
              @dialogClosed="hideTipoPublicDialog"
            />
          </q-dialog>

          <q-input
            v-if="editForm.tipo_publicacion === 'Revista Impresa'"
            v-model="editForm.issn"
            label="ISSN"
            class="form-item"
          />
          <q-input
            v-if="editForm.tipo_publicacion === 'Revista Digital'"
            v-model="editForm.e_issn"
            label="E-ISSN"
            class="form-item"
          />
          <q-input
            v-if="
              editForm.tipo_publicacion === 'Libro Impreso' ||
              editForm.tipo_publicacion === 'Libro Digital'
            "
            v-model="editForm.isbn"
            label="ISBN"
            class="form-item"
          />
          <q-input
            v-model="editForm.grupo"
            label="Grupo"
            class="form-item"
            @click="showGrupoDialog = true"
          />
          <q-dialog v-model="showGrupoDialog" persistent>
            <selector-grupo
              v-model="editForm.grupo"
              :public-types="[
                'Grupo 1',
                ' Grupo 2',
                ' Grupo 3',
                ' Grupo 4',
                ' No Precisado',
              ]"
              :open-dialog-automatically="showGrupoDialog"
              @update:modelValue="editForm.grupo = $event"
              @dialogClosed="hideGrupoDialog"
            />
          </q-dialog>
          <q-input autogrow v-model="editForm.url" label="URL" />
          <q-checkbox v-model="editForm.cdrom_dvd" label="CDROM/DVD" />
          <q-checkbox v-model="editForm.base_de_datos" label="Base de Datos" />
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
import SelectorDepartamento from 'src/components/SelectorDepartamento.vue';
import SelectorTipoRecurso from 'src/components/SelectorTipoRecurso.vue';
import SelectorTipoPublic from 'src/components/SelectorTipoPublic.vue';
import SelectorGrupo from 'src/components/SelectorGrupo.vue';
import { api } from 'src/boot/axios';
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
const showTipoRecursoDialog = ref(false);
const hideTipoRecursoDialog = () => {
  showTipoRecursoDialog.value = false;
};

const showTipoPublicDialog = ref(false);
const hideTipoPublicDialog = () => {
  showTipoPublicDialog.value = false;
};
const showGrupoDialog = ref(false);
const hideGrupoDialog = () => {
  showGrupoDialog.value = false;
};
const isLoading = ref(true);
type RowType = {
  id: number;
  nombre: string;
  apellidos: string;
  titulo_recurso: string;
  departamento: string;
  lugar_pub: string;
  tomo: string;
  folio: string;
  tipo_publicacion: string;
  grupo: string;
  issn: string;
  e_issn: string;
  isbn: string;
  cdrom_dvd: boolean;
  base_de_datos: boolean;
  url: string;
  tipo_recurso: string;
  fecha: string;
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
    name: 'titulo_recurso',
    label: 'Titulo del Recurso',
    field: 'titulo_recurso',
    align: 'left',
    sortable: true,
    filter: true,
    classes: 'texto-truncado',
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
    name: 'lugar_pub',
    label: 'Lugar de Publicación',
    field: 'lugar_pub',
    align: 'left',
    sortable: true,
    filter: true,
  },
  {
    name: 'tipo_recurso',
    label: 'Tipo de Publicación',
    field: 'tipo_recurso',
    align: 'left',
    sortable: true,
    filter: true,
    classes: 'texto-truncado',
  },
  {
    name: 'tipo_publicacion',
    label: 'Tipo de Recurso',
    field: 'tipo_publicacion',
    align: 'left',
    sortable: true,
    filter: true,
    classes: 'texto-truncado',
  },

  {
    name: 'grupo',
    label: 'Grupo',
    field: 'grupo',
    align: 'left',
    sortable: true,
    filter: true,
    classes: 'texto-truncado',
  },
  {
    name: 'tomo',
    label: 'Tomo',
    align: 'left',
    field: 'tomo',
    sortable: true,
    filter: true,
  },
  {
    name: 'folio',
    label: 'Folio',
    field: 'folio',
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
    const response = await api.get('/api/profesores/', config);
    console.log('Formulario enviado con éxito:', response.data.results);
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
  titulo_recurso: string;
  departamento: string;
  lugar_pub: string;
  grupo: string;
  tomo: string;
  folio: string;
  tipo_publicacion: string;
  issn: string;
  e_issn: string;
  isbn: string;
  cdrom_dvd: boolean;
  base_de_datos: boolean;
  url: string;
  tipo_recurso: string;
  fecha: string;
}
const editForm = reactive<Form>({
  nombre: '',
  apellidos: '',
  titulo_recurso: '',
  departamento: '',
  lugar_pub: '',
  grupo: '',
  tomo: '',
  folio: '',
  tipo_publicacion: '',
  issn: '',
  e_issn: '',
  isbn: '',
  cdrom_dvd: false,
  base_de_datos: false,
  url: '',
  tipo_recurso: '',
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
function formatWithInfiniteSeparators(value) {
  // Convertir el valor a una cadena y eliminar caracteres no numéricos
  let cleaned = ('' + value).replace(/\D/g, '');

  // Dividir la cadena en grupos de tres dígitos
  let groups = [];
  for (let i = 0; i < cleaned.length; i += 4) {
    groups.push(cleaned.substr(i, 4));
  }

  // Concatenar los grupos con guiones
  let result = groups.join('-');

  return result;
}
watch(
  () => editForm.isbn,
  (newValue) => {
    editForm.isbn = formatWithInfiniteSeparators(newValue);
  },
  { immediate: true }
);
watch(
  () => editForm.issn,
  (newValue) => {
    editForm.issn = formatWithInfiniteSeparators(newValue);
  },
  { immediate: true }
);

watch(
  () => editForm.e_issn,
  (newValue) => {
    editForm.e_issn = formatWithInfiniteSeparators(newValue);
  },
  { immediate: true }
);

//boton editar
const editRow = (row: RowType) => {
  selectedRow.value = row;
  editForm.nombre = row.nombre;
  editForm.apellidos = row.apellidos;
  editForm.titulo_recurso = row.titulo_recurso;
  editForm.departamento = row.departamento;
  editForm.lugar_pub = row.lugar_pub;
  editForm.tomo = row.tomo;
  editForm.grupo = row.grupo;
  editForm.folio = row.folio;
  editForm.tipo_publicacion = row.tipo_publicacion || '';
  editForm.issn = row.issn || '';
  editForm.e_issn = row.e_issn || '';
  editForm.isbn = row.isbn || '';
  editForm.cdrom_dvd = row.cdrom_dvd || false;
  editForm.base_de_datos = row.base_de_datos || false;
  editForm.url = row.url || '';
  editForm.tipo_recurso = row.tipo_recurso || '';
  editForm.fecha = row.fecha;

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
      `/api/profesores/${selectedRow.value.id}/`,
      { ...editForm },
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
  console.log('Mostrando detalles del recurso:', row);
  router.push({ name: 'show', params: { id: row.id } });
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
        $q.loading.show();
        api
          .delete(`/api/profesores/${row.id}/`, config)
          .then(() => {
            console.log('Recurso eliminado con éxito');
            rows.value = rows.value.filter((item) => item.id !== row.id);
            $q.loading.hide();
            $q.notify({
              type: 'positive', // Cambiado a positive para indicar éxito
              message: '¡Aval Eliminado Correctamente!',
              position: 'bottom-right',
            });
          })
          .catch((error) => {
            $q.loading.hide();
            console.error('Error al eliminar el recurso:', error);
            $q.notify({
              type: 'negative',
              message: 'Hubo un error al eliminar el Aval.',
              position: 'bottom-right',
            });
          });
      });
  } catch (error) {
    $q.loading.hide();
    console.error('Error al mostrar el diálogo:', error);
  }
}
</script>
