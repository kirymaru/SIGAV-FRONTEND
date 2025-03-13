<template>
  <div>
    <q-dialog v-model="firstDialog" persistent>
      <q-card style="width: 380px">
        <q-card-section class="text-bold q-gutter-sm"
          >Selecciona la Facultad:
          <div>
            <q-select
              class="text-bold"
              dense
              style="width: 350px"
              v-model="selectedFaculty"
              :options="faculty"
              label="Facultades"
              @input="selectFaculty"
            />
          </div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" @click="handleCloseAndEmit" />
          <q-btn
            color="primary"
            flat
            label="Aceptar"
            @click="() => selectFaculty(selectedFaculty)"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="secondDialog" persistent>
      <q-card style="width: 380px">
        <q-card-section class="text-bold q-gutter-sm"
          >Selecciona un Departamento:
          <div>
            <q-select
              dense
              v-model="selectedDepartment"
              :options="selectedDepartmentOptions"
              label="Departamentos"
              @input="selectDepartment"
            /></div
        ></q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" @click="closeSecondDialog" />
          <q-btn
            color="primary"
            flat
            label="Aceptar"
            @click="() => selectDepartment(selectedDepartment)"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits, defineProps } from 'vue';

const props = defineProps({
  modelValue: String,
  departamentoRules: Array,
  openFirstDialogAutomatically: Boolean, // Nuevo prop para controlar la apertura automática del primer diálogo
});

interface DepartmentsByFaculty {
  [key: string]: string[];
}
// Definición de datos reactivos
const firstDialog = ref(false);
const secondDialog = ref(false);
const selectedFaculty = ref('');
const selectedDepartment = ref('');
const selectedDepartmentOptions = ref(['']);
const faculty = [
  'Ciencias Sociales',
  'Ciencias Aplicadas',
  'Ciencias Agropecuarias',
  'Ciencias Económicas',
  'Electromecánica',
  'Construcciones',
  'Lengua y Comunicación',
  'Informática y Ciencias Exactas',
  'Ciencias Pedagógicas',
  'Cultura Física',
  'CUM',
];
const departmentsByFaculty: DepartmentsByFaculty = {
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
};
const emit = defineEmits(['update:modelValue', 'closeFirstDialog']);

// Métodos
const openFirstDialog = () => {
  firstDialog.value = true;
};
if (props.openFirstDialogAutomatically) {
  openFirstDialog();
}
const closeFirstDialog = () => {
  firstDialog.value = false;
};
const handleCloseAndEmit = () => {
  firstDialog.value = false;
  emit('closeFirstDialog');
};

const selectFaculty = (faculty: string) => {
  if (!faculty.trim()) {
    alert('Por favor selecciona una facultad.');
    return;
  }
  closeFirstDialog();
  selectedFaculty.value = faculty;
  openSecondDialogWithDepartments(faculty);
};
const openSecondDialogWithDepartments = (faculty: string) => {
  if (departmentsByFaculty[faculty]) {
    secondDialog.value = true;
    selectedDepartment.value = '';
    selectedDepartmentOptions.value = departmentsByFaculty[faculty];
  }
};
const closeSecondDialog = () => {
  secondDialog.value = false;
  emit('closeFirstDialog');
};
const selectDepartment = (department: string) => {
  if (!department.trim()) {
    alert('Por favor selecciona un departamento.');
    return;
  }
  closeSecondDialog();
  emit('update:modelValue', department);
  emit('closeFirstDialog');
};

// Propiedades computadas y otros hooks opcionales van aquí
</script>
