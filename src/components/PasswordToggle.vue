<template>
  <q-input
    dense
    :type="showPassword ? 'text' : 'password'"
    :model-value="contraseña"
    @update:model-value="emitValue"
    label="Contraseña"
    :rules="rules"
    @input="handleInput"
  >
    <template v-slot:append>
      <q-icon
        :name="showPassword ? 'visibility_off' : 'visibility'"
        class="cursor-pointer"
        @click="togglePasswordVisibility"
      />
    </template>
  </q-input>
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue';

const props = defineProps<{
  value?: string; // Valor inicial opcional
  rules?: Array<(val: string) => boolean>; // Reglas de validación opcionales
}>();

const emit = defineEmits(['update:value']); // Emite un evento personalizado para v-model

let showPassword = ref(false); // Asume que modelValue es booleano
let contraseña = ref('');

watchEffect(() => {
  contraseña.value = props.value ?? '';
});

function togglePasswordVisibility() {
  showPassword.value = !showPassword.value;
}

function handleInput(value: string) {
  // Actualizar el valor solo si las reglas son válidas
  if (props.rules?.every((rule) => rule(value))) {
    // Directamente actualizamos el valor del modelo sin emitir un evento aquí
    contraseña.value = value;
  }
}

function emitValue(newValue: string) {
  contraseña.value = newValue;
  showPassword.value = newValue === ''; // Ocultar si el valor es vacío
  emit('update:value', showPassword.value);
}
</script>
