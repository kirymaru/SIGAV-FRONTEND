<template>
  <div>
    <q-card class="q-pa-md" style="width: 800px">
      <canvas ref="chartCanvas"></canvas>
    </q-card>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import Chart from 'chart.js/auto';
import { api } from 'src/boot/axios';
import { useQuasar } from 'quasar';

const $q = useQuasar();

export default {
  setup() {
    const chartCanvas = ref(null);
    let chart = null;

    // Función para obtener los datos del API
    const obtenerDatos = async () => {
      try {
        const authToken = localStorage.getItem('authToken');
        const config = {
          headers: {
            Authorization: `Token ${authToken}`,
            'Content-Type': 'application/json',
          },
        };

        const response = await api.get(
          '/api/reporte-total-avaless-por-fecha/',
          config
        );
        return response.data;
      } catch (error) {
        console.error('Error al obtener los datos:', error);
        return null;
      }
    };

    // Función para actualizar el gráfico
    const actualizarGrafico = (datos) => {
      if (chart) {
        chart.destroy();
      }

      const labels = Object.keys(datos);
      const totales = Object.values(datos).map((item) => item.total);
      const departamentos = Object.values(datos).map(
        (item) => item.departamentos
      );

      chart = new Chart(chartCanvas.value.getContext('2d'), {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Total de Avales',
              data: totales,
              backgroundColor: '#1976D2',
              borderColor: '#1976D2',
              borderWidth: 1,
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: 'top' },
            tooltip: {
              callbacks: {
                label: function (context) {
                  const fecha = context.dataset.label;
                  const datosFecha = datos[fecha];
                  let tooltip = `Total: ${context.raw}`;

                  Object.entries(datosFecha.departamentos).forEach(
                    ([dept, count]) => {
                      tooltip += `\n${dept}: ${count}`;
                    }
                  );

                  return tooltip;
                },
              },
            },
          },
        },
      });
    };

    // Obtener datos y crear el gráfico inicial
    onMounted(async () => {
      const datos = await obtenerDatos();
      if (datos) {
        actualizarGrafico(datos);
      }
    });

    return { chartCanvas };
  },
};
</script>
