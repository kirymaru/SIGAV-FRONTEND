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
          Aval de Tutoría
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
          Tuteló : {{ response.titulo_recurso }}
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
import { ref, onMounted } from 'vue';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { useRoute } from 'vue-router';
import { api } from 'src/boot/axios';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

interface ResponseItem {
  apellidos?: string;
  departamento?: string;
  fecha?: string;
  folio?: string;
  id?: number;
  nombre?: string;
  titulo_recurso?: string;
  tomo?: string;
}
const router = useRouter();
const route = useRoute();
const id = route.params.id;
const goBack = () => {
  router.back();
};
const $q = useQuasar();
const response = ref<ResponseItem>({});

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
      `/api/avales_tuto/${id}/`,
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
  const element = document.querySelector('.my-card'); // Considera reemplazar esto con un componente Vue personalizado
  if (!element) {
    console.error('No se encontró el elemento.my-card');
    return;
  }
  html2canvas(element, { scrollY: -window.scrollY }).then((canvas) => {
    // Añade scrollY para manejar scroll en la página
    const imgData = canvas.toDataURL('image/png');
    const imgProps = pdf.getImageProperties(imgData);
    const pdfWidth = pdf.internal.pageSize.getWidth(); // Obtiene el ancho completo de la página PDF
    const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width; // Calcula la altura proporcionalmente
    pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight); // Añade la imagen al PDF con las dimensiones calculadas
    pdf.save(
      `Aval de Tutoría Autor:${response.value.nombre} ${response.value.apellidos} ${response.value.fecha}.pdf`
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
