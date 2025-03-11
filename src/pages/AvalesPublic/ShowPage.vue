<template>
  <q-page class="row justify-evenly">
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
          Aval de Publicación
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
          Del Departamento: {{ response.departamento }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Ha publicado el: {{ response.tipo_recurso }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          Denominado: {{ response.titulo_recurso }}
        </div>
        <div class="text-semibold datos-recurso text-h6">
          En: {{ response.lugar_pub }}
        </div>

        <div class="text-semibold datos-recurso text-h6">
          Siendo esta publicación: {{ response.tipo_publicacion }}
        </div>

        <div class="text-semibold datos-recurso text-h6">
          Sitio Web(indicar URL): {{ response.url }}
        </div>
        <div class="row">
          <div class="text-semibold datos-recurso text-h6">
            CD-ROM/DVD:
            <q-checkbox
              v-model="isCdromDvdEnabled"
              :disable="response.cdrom_dvd"
            />
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Base(s) de Datos:
            <q-checkbox
              v-model="isBDEnabled"
              :disable="response.base_de_datos"
            />
          </div>
        </div>
        <div class="row">
          <div class="text-semibold datos-recurso text-h6" v-if="response.isbn">
            ISBN: {{ response.isbn }}
          </div>
          <div class="text-semibold datos-recurso text-h6" v-if="response.issn">
            ISSN: {{ response.issn }}
          </div>
          <div
            class="text-semibold datos-recurso text-h6"
            v-if="response.e_issn"
          >
            E-ISSN: {{ response.e_issn }}
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Grupo: {{ response.grupo }}
          </div>
        </div>

        <div class="text-semibold datos-recurso text-h6">
          Para que así conste, firmo la presente en la fecha:
          {{ response.fecha }}
        </div>
        <div class="row items-center justify-between">
          <div class="text-semibold datos-recurso text-h6">
            MsC. Adrián Eduardo Cancino Gutiérrez / Director
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Firma:______________
          </div>
        </div>
        <div class="row">
          <div class="text-semibold datos-recurso text-h6">
            Tomo: {{ response.tomo }}
          </div>
          <div class="text-semibold datos-recurso text-h6">
            Folio: {{ response.folio }}
          </div>
        </div>

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
  apellidos: string;
  departamento: string;
  fecha: string;
  folio: string;
  nombre: string;
  titulo_recurso: string;
  tomo: string;
  lugar_pub: string;
  grupo: string;
  tipo_recurso: string;
  tipo_publicacion: string;
  url: string;
  cdrom_dvd: boolean;
  base_de_datos: boolean;
  issn: string;
  e_issn: string;
  isbn: string;
}
const router = useRouter();
const route = useRoute();
const id = route.params.id;
const goBack = () => {
  router.back();
};
const response = ref<ResponseItem>({
  apellidos: '',
  departamento: '',
  fecha: '',
  folio: '',
  nombre: '',
  titulo_recurso: '',
  tomo: '',
  lugar_pub: '',
  grupo: '',
  tipo_recurso: '',
  tipo_publicacion: '',
  url: '',
  cdrom_dvd: false,
  base_de_datos: false,
  issn: '',
  e_issn: '',
  isbn: '',
});
const isCdromDvdEnabled = computed(() => response.value.cdrom_dvd);
const isBDEnabled = computed(() => response.value.base_de_datos);
const $q = useQuasar();
const fetchData = async () => {
  try {
    const authToken = localStorage.getItem('authToken'); // Asume que tienes un authToken almacenado
    const config = {
      headers: {
        Authorization: `Token ${authToken}`,
        'Content-Type': 'application/json',
      },
    };
    $q.loading.show();
    const result = await api.get<ResponseItem>(
      `/api/profesores/${id}/`,
      config
    );
    response.value = result.data;
    $q.loading.hide();
  } catch (error) {
    $q.loading.hide();
    console.error('Error al obtener los detalles:', error);
  }
};

onMounted(fetchData);

const exportToPDF = () => {
  $q.loading.show();
  const pdf = new jsPDF();
  const element = document.querySelector('.my-card');
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
      `Aval de Publicación Autor:${response.value.nombre} ${response.value.apellidos} Titulo:${response.value.titulo_recurso} ${response.value.fecha}.pdf`
    );
  });
  $q.loading.hide();
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
