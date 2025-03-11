<template>
  <q-page class="row items-center justify-evenly">
    <div
      class="column items-center justify-around"
      style="margin-top: 20px; margin-bottom: 20px"
    >
      <div class="my-card q-pa-md items-start">
        <div class="row items-center justify-between">
          <div class="row items-center">
            <div>
              <q-avatar size="120px">
                <img src="src/assets/logo.png" />
              </q-avatar>
            </div>
            <div>
              <q-avatar size="250px">
                <img src="src/assets/logodgic.png" />
              </q-avatar>
            </div>
          </div>
        </div>
        <div class="row justify-center items-center text-bold text-h4">
          Aval de Bibliografía
        </div>
        <div class="text-semibold datos-recurso text-h6">
          MsC. Adrián Eduardo Cancino Gutiérrez, Director de Información
          Científico-Técnica de la Universidad de Camagüey "Ignacio Agramonte
          Loynaz"
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Hago constar que: {{ response.nombre }} {{ response.apellidos }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Realizó la revisión bibliográfica correspondiente a:
          {{ response.rev_bilio }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          El resultado de la búsqueda es la siguiente:
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Total de asientos bibliográficos(en letras):
          {{ response.total_asient }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Tipo de búsqueda informativa utilizada:
        </div>
        <div class="row justify-around items-center">
          <div class="column items-start">
            <div class="col text-semibold text-h6">
              CD-ROM/DVD:
              <q-checkbox
                v-model="isCdromDvdEnabled"
                :disable="response.cd_rom"
              />
            </div>

            <div class="col text-semibold text-h6">
              Base de datos locales:
              <q-checkbox
                v-model="isBdLocalEnabled"
                :disable="response.bd_local"
              />
            </div>
            <div class="col text-semibold text-h6">
              Base de datos en Internet:
              <q-checkbox
                v-model="isBdInterEnabled"
                :disable="response.bd_internet"
              />
            </div>
            <div class="col text-semibold text-h6">
              Curso de Posgrado:
              <q-checkbox
                v-model="isCursoPosEnabled"
                :disable="response.curso_pos_bus"
              />
            </div>
          </div>
          <div class="column">
            <div class="col text-semibold text-h6">
              Búsqueda en Internet:
              <q-checkbox
                v-model="isBusqIntEnabled"
                :disable="response.busqueda_internet"
              />
            </div>
            <div class="col text-semibold text-h6">
              Bibliografía personal:
              <q-checkbox
                v-model="isBiblioPerEnabled"
                :disable="response.biblio_personal"
              />
            </div>
            <div class="col text-semibold text-h6">
              Otros:
              <q-checkbox v-model="isOtrosEnabled" :disable="response.otros" />
            </div>
            <div class="col text-semibold text-h6">
              No existe bibliografía:
              <q-checkbox
                v-model="isNoBiblioEnabled"
                :disable="response.no_biblio"
              />
            </div>
          </div>
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Nivel de actualización: {{ response.niv_act }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Y para que así lo pueda acreditar, se firma la presente con la fecha:
          {{ response.fecha }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Este aval tiene una duración de un año a partir de la fecha en que se
          expide.
        </div>
        <div class="row">
          <div class="text-semibold datos-recurso text-h6">
            Tomo: {{ response.tomo }}
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Folio: {{ response.folio }}
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Pág: {{ response.pag }}
          </div>
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Revisado Por(nombre y apellidos): {{ response.nombre }}
          {{ response.apellidos }}
        </div>
        <div class="text-semibold datos-recurso text-h6">Firma:___________</div>
        <div class="text-semibold datos-recurso text-h6">
          Dirección de Información Científico-Técnica de la Universidad de
          Camagüey "Ignacio Agramonte Loynaz". Telefono:(53)32262332. E-mail:
          dict@reduc.edu.cu. http://infocien.reduc.edu.cu
        </div>
      </div>
      <div>
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
          icon="print"
          size="sm"
          color="primary"
          rounded
          label="Exportar a PDF"
          @click="exportToPDF"
          class="btn-export mt-4 text-weight-bolder"
          style="margin-top: 20px; margin-bottom: 20px"
        />
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { useRoute } from 'vue-router';
import { api } from 'src/boot/axios';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

interface ResponseItem {
  nombre: string;
  apellidos: string;
  departamento: string;
  rev_bilio: string;
  niv_act: string;
  total_asient: string;
  bd_local?: boolean;
  cd_rom?: boolean;
  bd_internet?: boolean;
  curso_pos_bus: boolean;
  busqueda_internet?: boolean;
  biblio_personal?: boolean;
  otros?: boolean;
  no_biblio?: boolean;
  tomo: string;
  pag: string;
  folio: string;
  fecha: string;
}
const router = useRouter();
const route = useRoute();
const id = route.params.id;
const goBack = () => {
  router.back();
};
const $q = useQuasar();
const response = ref<ResponseItem>({
  nombre: '',
  apellidos: '',
  departamento: '',
  rev_bilio: '',
  niv_act: '',
  total_asient: '',
  bd_local: false || true,
  cd_rom: false || true,
  bd_internet: false || true,
  curso_pos_bus: false || true,
  busqueda_internet: false || true,
  biblio_personal: false || true,
  otros: false || true,
  no_biblio: false || true,
  tomo: '',
  pag: '',
  folio: '',
  fecha: '',
});
const isBdLocalEnabled = computed(() => response.value.bd_local);
const isCdromDvdEnabled = computed(() => response.value.cd_rom);
const isBdInterEnabled = computed(() => response.value.bd_internet);
const isCursoPosEnabled = computed(() => response.value.curso_pos_bus);
const isBusqIntEnabled = computed(() => response.value.busqueda_internet);
const isBiblioPerEnabled = computed(() => response.value.biblio_personal);
const isOtrosEnabled = computed(() => response.value.otros);
const isNoBiblioEnabled = computed(() => response.value.no_biblio);
const fetchData = async () => {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    const result = await api.get<ResponseItem>(
      `/api/avales_biblio/${id}/`,
      config
    );
    response.value = result.data;
  } catch (error) {
    console.error('Error al obtener los detalles:', error);
  }
};

onMounted(fetchData);

const exportToPDF = () => {
  const pdf = new jsPDF();
  const element = document.querySelector('.my-card'); // Considera usar un componente Vue para referenciar
  if (!element) {
    console.error('No se encontró el elemento.my-card');
    return;
  }
  html2canvas(element).then((canvas) => {
    const imgData = canvas.toDataURL('image/png');
    const imgProps = pdf.getImageProperties(imgData);
    const pdfHeight = pdf.internal.pageSize.getHeight(); // Altura del PDF
    const pdfWidth = (pdfHeight * imgProps.width) / imgProps.height;
    pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
    pdf.save(
      `Aval Bibliográfico ${response.value.nombre} ${response.value.apellidos} de ${response.value.rev_bilio} ${response.value.fecha}.pdf`
    );
  });
  $q.notify({
    type: 'positive',
    message: '¡Aval Exportado Correctamente!',
    position: 'bottom-right',
  });
};
</script>

<style scoped>
.my-card {
  width: 100%;
  max-width: 800px;
  background-color: #f9f9f9;
  border-radius: 8px;
}
.datos-recurso {
  word-break: break-word;
  overflow-wrap: break-word;
  max-width: 90%;
  margin: auto;
  margin-top: 20px;
}
</style>
