<template>
  <div>
    <q-dialog v-model="isDialogOpen" persistent>
      <q-card style="width: 300px">
        <q-card-section class="text-bold">Selecciona el Rol:</q-card-section>
        <q-select
          dense
          v-model="selectedPublicType"
          :options="publicTypes"
          label="Roles"
          @input="handleSelection"
        />
        <q-card-actions align="right">
          <q-btn flat label="Cancelar" @click="handleCancel" />
          <q-btn flat label="Aceptar" @click="handleAccept" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, } from 'vue';

const props = defineProps({
  modelValue: String,
  publicTypes: Array,
  openDialogAutomatically: Boolean,
});

const emit = defineEmits(['update:modelValue', 'dialogClosed']);

const isDialogOpen = computed({
  get: () => props.openDialogAutomatically,
  set: (value) => value? emit('DialogOpen') : emit('dialogClosed')
});

const selectedPublicType = ref('');

const handleSelection = (publicType: string) => {
  selectedPublicType.value = publicType;
  emit('dialogClosed');
};

const handleAccept = () => {
  if (selectedPublicType.value) {
    emit('update:modelValue', selectedPublicType.value);

  }
  emit('dialogClosed');
};

const handleCancel = () => {
  emit('dialogClosed');
};
</script>
