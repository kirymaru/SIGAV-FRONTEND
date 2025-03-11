<template>
  <q-page class="flex flex-center bg-color">
    <q-form @submit="onSubmit" id="form-login">
      <div class="row">
        <div>
          <img
            src="/src/assets/logotoolbar.png"
            alt="Descripción de la imagen"
            class="centered-image"
            width="200px"
          />
        </div>
        <div class="column q-gutter-md justify-center items-center">
          <q-input
            dense
            v-model="email"
            label="Correo Electrónico"
            lazy-rules
            style="width: 200px"
          />

          <PasswordToggle v-model="password" style="width: 200px" />

          <div>
            <q-btn
              size="sm"
              rounded
              label="Iniciar sesión"
              type="submit"
              color="primary"
            />
          </div>
        </div>
      </div>
    </q-form>
  </q-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { api } from 'src/boot/axios';
import { useRouter } from 'vue-router';
import { useQuasar } from 'quasar';
import PasswordToggle from 'src/components/PasswordToggle.vue';

const email = ref('');
const password = ref('');
const router = useRouter();

const errorMessage = ref('');

const $q = useQuasar();
const onSubmit = async () => {
  if (!email.value || !password.value) {
    handleLoginError('Por favor, complete todos los campos.');
    return;
  }
  $q.loading.show();
  try {
    const response = await api.post('/api/token/', {
      email: email.value,
      password: password.value,
    });

    const token = response.data.token;
    localStorage.setItem('authToken', token);
    $q.loading.hide();
    navigateHome();
  } catch (error) {
    handleLoginError('Correo electrónico o Contraseña incorrectos.');
    console.error('Error al iniciar sesión:', error);
    $q.loading.hide();
  } finally {
  }
};

const handleLoginError = (message: string) => {
  errorMessage.value = message;

  $q.notify({
    type: 'negative',
    message: 'Correo electrónico o Contraseña incorrectos.',
    position: 'top-right',
    color: 'red',
    textColor: 'white',
    icon: 'error_outline',
  });
};

const navigateHome = () => {
  router.push('/home');
  $q.notify({
    type: 'positive',
    message: '¡Inicio de sesión exitoso!',
    position: 'bottom-right',
  });
};
</script>

<style scoped>
.my-card {
  width: 100%;
  max-width: 400px;
}
</style>
