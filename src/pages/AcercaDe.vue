<template>
  <q-page class="q-pa-lg">
    <div class="row q-col-gutter-xl items-center justify-center">
      <!-- Sección Hero -->
      <div class="col-12 col-md-6 text-center">
        <q-img
          src="/src/assets/logo-bg.gif"
          alt="Logo SIGAV"
          class="rounded-borders shadow-5 fondo-transparente"
          style="max-width: 500px"
        />
      </div>

      <div class="col-12 col-md-6">
        <div class="text-h3 text-weight-bold text-primary q-mb-sm">
          Bienvenido al Futuro de la Excelencia Académica
        </div>
        <p class="text-h6 text-grey-7 q-mb-md">
          Plataforma oficial de la Universidad de Camagüey para la gestión de
          avales científicos
        </p>
        <q-card class="q-pa-md bg-primary text-white">
          <p class="text-body1">
            "Desde 2023, hemos facilitado la aprobación de más de 1,200
            proyectos de investigación, conectando a 350 profesores y 15
            departamentos académicos en una sola plataforma"
          </p>
        </q-card>
      </div>

      <div class="row q-col-gutter-lg justify-center">
        <div
          class="col-12 col-md-4"
          v-for="(step, index) in pasos"
          :key="index"
        >
          <q-card class="full-height">
            <q-card-section class="text-center">
              <div class="text-h2 text-primary q-mb-md">0{{ index + 1 }}</div>
              <q-icon
                :name="step.icon"
                size="lg"
                color="primary"
                class="q-mb-md"
              />
              <div class="text-h6 text-weight-bold q-mb-sm">
                {{ step.titulo }}
              </div>
              <div class="text-caption text-grey-7">{{ step.descripcion }}</div>
            </q-card-section>
          </q-card>
        </div>
      </div>

      <!-- Sección Beneficios -->
      <!-- Sección Beneficios Mejorada -->
      <div class="col-12 q-pt-xl">
        <div class="text-center q-mb-xl">
          <div class="text-h3 text-weight-bold text-primary q-mb-sm">
            Ventajas Cuantificables
          </div>
          <div class="text-h6 text-grey-7">
            Resultados medibles que impulsan la excelencia académica
          </div>
        </div>

        <div class="row q-col-gutter-lg justify-center">
          <q-card
            v-for="(beneficio, index) in beneficiosMejorados"
            :key="index"
            class="col-12 col-md-5"
            :class="beneficio.color"
          >
            <q-card-section class="q-pa-lg">
              <div class="row items-center">
                <div class="col-3 text-center">
                  <q-icon
                    :name="beneficio.icon"
                    size="xl"
                    color="white"
                    class="q-mb-sm"
                  />
                  <div class="text-h5 text-white text-weight-bold">
                    {{ beneficio.metrica }}
                  </div>
                  <div class="text-caption text-white">
                    {{ beneficio.unidad }}
                  </div>
                </div>

                <q-separator vertical color="white-20" class="q-mx-lg" />

                <div class="col">
                  <div class="text-h6 text-white text-weight-bold q-mb-xs">
                    {{ beneficio.titulo }}
                  </div>
                  <div class="text-caption text-white-80 q-mb-sm">
                    {{ beneficio.descripcion }}
                  </div>
                  <q-linear-progress
                    v-if="beneficio.progreso"
                    :value="beneficio.progreso"
                    size="8px"
                    color="white"
                    track-color="white-20"
                    class="rounded-borders"
                  />
                </div>
              </div>
            </q-card-section>

            <q-tooltip v-if="beneficio.detalle" :delay="500">
              <div class="text-caption">{{ beneficio.detalle }}</div>
            </q-tooltip>
          </q-card>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup>
const pasos = [
  {
    titulo: 'Registro Digital',
    descripcion:
      'Carga tu documento en formato electrónico con validación automática',
    icon: 'cloud_upload',
  },
  {
    titulo: 'Revisión Colaborativa',
    descripcion: 'Sistema de comentarios en tiempo real con los evaluadores',
    icon: 'group',
  },
  {
    titulo: 'Aprobación Instantánea',
    descripcion: 'Recibe tu aval digital con firma electrónica certificada',
    icon: 'verified',
  },
];
const beneficiosMejorados = [
  {
    titulo: 'Eficiencia Operativa',
    descripcion: 'Reducción promedio en procesos de validación',
    metrica: '78%',
    unidad: 'Menos tiempo',
    icon: 'timer',
    color: 'bg-gradient-primary',
    progreso: 0.78,
    detalle: 'Basado en 1,240 procesos analizados en 2023',
  },
  {
    titulo: 'Colaboración Interdepartamental',
    descripcion: 'Aumento en revisiones colaborativas',
    metrica: '3.5x',
    unidad: 'Más interacciones',
    icon: 'group_work',
    color: 'bg-gradient-teal',
    progreso: 0.65,
    detalle: 'Comparado con sistema anterior (2020-2022)',
  },
  {
    titulo: 'Satisfacción de Usuarios',
    descripcion: 'Calificación promedio de la plataforma',
    metrica: '4.8',
    unidad: '/5.0 estrellas',
    icon: 'star',
    color: 'bg-gradient-orange',
    detalle: 'Encuesta a 358 usuarios activos',
  },
  {
    titulo: 'Reducción de Errores',
    descripcion: 'Documentación incompleta o errónea',
    metrica: '92%',
    unidad: 'Menos incidencias',
    icon: 'bug_report',
    color: 'bg-gradient-purple',
    progreso: 0.92,
  },
];
</script>

<style scoped>
.bg-gradient-primary {
  background: linear-gradient(135deg, #1976d2, #2196f3);
}

.bg-gradient-teal {
  background: linear-gradient(135deg, #009688, #4db6ac);
}

.bg-gradient-orange {
  background: linear-gradient(135deg, #ff9800, #ffb74d);
}

.bg-gradient-purple {
  background: linear-gradient(135deg, #9c27b0, #ba68c8);
}

.q-card {
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 16px;
  overflow: hidden;
}

.q-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.2);
}

.q-linear-progress {
  border-radius: 4px;
}

.rounded-borders {
  border-radius: 20px;
}
.text-h4 {
  font-size: 2.5rem;
  line-height: 1.2;
}

.text-body1 {
  font-size: 1.1rem;
  line-height: 1.6;
}
</style>
