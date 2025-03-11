<template>
  <div class="q-pa-lg">
    <q-table
      bordered
      dense
      virtual-scroll
      title="Lista de Autores"
      title-class="text-bold "
      :rows="Autor"
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
        <q-input dense outlined v-model="search" placeholder="Buscar" />
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
      <q-card style="width: 500px; height: 400px">
        <q-card-section>
          <div class="text-h6">Editar Aautor</div>
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

        </q-card-section>

        <q-card-actions align="right" >
          <q-btn rounded label="Cancelar" v-close-popup />
          <q-btn rounded color="primary" label="Guardar" @click="saveEdit" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive, watch, toRefs } from 'vue';
import { api } from 'src/boot/axios';
import { useRouter } from 'vue-router';
import { useQuasar } from 'quasar';
import SelectorDepartamento from 'src/components/SelectorDepartamento.vue';

const user = ref({ role: 'invitado', isAdmin: false, isViewerOnly: false });
const $q = useQuasar();
const router = useRouter();
const isLoading = ref(true);
const selectedRow = ref<AutorType | null>(null);
const search = ref('');
const showSelectorDepartamento = ref(false);
const closeFirstDialogAndUpdateModel = () => {
  showSelectorDepartamento.value = false;
};


type AutorType = {
  id: number;
  nombre: string;
  apellidos: string;
  departamento: string;
};
const Autor = ref<AutorType[]>([]);

const columnas = ref([
  {
    name: 'nombre',
    required: true,
    label: 'Nombre',
    align: 'left',
    field: 'nombre',
    filter: true,
    sortable: true,
  },
  {
    name: 'apellidos',
    required: true,
    label: 'Apellidos',
    align: 'left',
    field: 'apellidos',
    filter: true,
    sortable: true,
  },
  {
    name: 'departamento',
    required: true,
    label: 'Departamento',
    align: 'left',
    field: 'departamento',
    filter: true,
    sortable: true,
  },
]);

const editDialogOpen = ref(false);
interface Form {
  nombre: string;
  apellidos: string;
  departamento: string;

}

const editForm = reactive<Form>({
  nombre: '',
  apellidos: '',
  departamento: '',

});
//boton editar
const editRow = (row: AutorType) => {
  selectedRow.value = row;
  editForm.nombre = row.nombre;
  editForm.apellidos = row.apellidos;
  editForm.departamento = row.departamento;
  editDialogOpen.value = true;
};

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
// Propiedad para recibir los datos de los autores

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
    const response = await api.get('/api/autores/', config);
    Autor.value = response.data.results;
    console.log('Formulario enviado con éxito:', response.data.results);
  } catch (error) {
    console.error('Error al obtener los datos de los profesores:', error);
  }
  isLoading.value = false;
  fetchUserData();
});

///boton mostrar
const showRow = (row: any) => {
  router.push({
    name: 'AvalesProfesor',
    params: {
      id: row.id,
      nombre: row.nombre,
      apellidos: row.apellidos,
    },
  });
  console.log(row);
};


//boton editar
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
      `/api/autores/${selectedRow.value.id}/`,
      editForm,
      config
    );

    const index = Autor.value.findIndex(
      (row) => row.id === selectedRow.value.id
    );
    if (index !== -1) {
      Object.assign(Autor.value[index], editForm);
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
    message: '¡Autor Actualizado Correctamente!',
    position: 'bottom-right',
  });
};

//boton eliminar
async function eliminar(row: { id: null }) {
  try {
    const authToken = localStorage.getItem('authToken');
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    await $q
      .dialog({

        title: 'Eliminar Autor',
        message: '¿Estás seguro de eliminar?',
        cancel: true,
        persistent: true,
      })
      .onOk(() => {
        api
          .delete(`/api/autores/${row.id}/`, config)
          .then(() => {
            console.log('Recurso eliminado con éxito');
           Autor.value =Autor.value.filter((item) => item.id !== row.id);
            $q.notify({
              type: 'positive',
              message: '¡Autor Eliminado Correctamente!',
              position: 'bottom-right',
            });
          })
          .catch((error) => {
            console.error('Error al eliminar el recurso:', error);
            $q.notify({
              type: 'negative',
              message: 'Hubo un error al eliminar el Autor.',
              position: 'bottom-right',
            });
          });
      });
  } catch (error) {
    console.error('Error al mostrar el diálogo:', error);
  }
}
</script>
