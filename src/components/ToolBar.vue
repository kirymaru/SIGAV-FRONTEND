<template>
  <q-header elevated class="bg-gradient-primary text-white shadow-4">
    <q-toolbar>
      <!-- Botón menú -->
      <q-btn
        flat
        dense
        round
        icon="menu"
        aria-label="Menú"
        class="q-mr-sm fondo-transparente text-white"
        @click="toggleLeftDrawer"
      />

      <!-- Logo y título -->
      <div class="row items-center">
        <q-avatar size="50px" class="q-mr-sm">
          <img src="src/assets/logotoolbar.png" alt="Logo SIGAV" />
        </q-avatar>
        <q-toolbar-title class="text-h5 text-weight-bold">
          SIGAV
          <div class="text-caption text-weight-light">
            Gestión de Avales Académicos
          </div>
        </q-toolbar-title>
      </div>

      <q-space />

      <!-- Menú usuario -->
      <q-btn
        round
        flat
        dense
        class="q-mr-sm fondo-transparente text-white"
        :icon="$q.screen.lt.md ? 'account_circle' : ''"
      >
        <div v-if="$q.screen.gt.sm" class="row items-center">
          <q-avatar size="40px" class="q-mr-sm">
            <q-icon name="account_circle" size="40px" />
          </q-avatar>
          <div class="column text-left">
            <div class="text-weight-bold">{{ abbreviatedUsername }}</div>
            <div class="text-caption">
              {{ formattedRole }}
            </div>
          </div>
        </div>

        <q-menu
          transition-show="jump-down"
          transition-hide="jump-up"
          class="shadow-5"
        >
          <div class="row no-wrap q-pa-lg" style="min-width: 300px">
            <!-- Sección izquierda -->
            <div class="column items-center q-pr-lg">
              <q-avatar size="80px" class="q-mb-sm">
                <q-icon name="account_circle" size="80px" />
              </q-avatar>
              <q-badge color="primary" class="text-uppercase">
                {{ formattedRole }}
              </q-badge>
            </div>

            <q-separator vertical inset class="q-mx-lg" />

            <!-- Sección derecha -->
            <div class="column">
              <q-list class="rounded-borders" dense>
                <q-item
                  v-if="user.isAdmin"
                  clickable
                  v-ripple
                  class="text-primary"
                  to="/Usuarios"
                >
                  <q-item-section avatar>
                    <q-icon name="people" />
                  </q-item-section>
                  <q-item-section>Administrar Usuarios</q-item-section>
                </q-item>

                <q-item clickable v-ripple to="/acerca_de">
                  <q-item-section avatar>
                    <q-icon name="info" />
                  </q-item-section>
                  <q-item-section>Acerca de SIGAV</q-item-section>
                </q-item>

                <q-separator spaced />

                <q-item
                  clickable
                  v-ripple
                  class="text-negative"
                  @click="logout"
                >
                  <q-item-section avatar>
                    <q-icon name="logout" />
                  </q-item-section>
                  <q-item-section>Cerrar Sesión</q-item-section>
                </q-item>
              </q-list>
            </div>
          </div>
        </q-menu>
      </q-btn>
    </q-toolbar>
  </q-header>

  <!-- Drawer -->
  <q-drawer
    v-model="leftDrawerOpen"
    :width="280"
    side="left"
    bordered
    overlay
    elevated
    class="bg-grey-2"
    :breakpoint="700"
  >
    <drawer-component />

    <q-separator class="q-mt-auto" />
    <q-footer class="q-py-sm text-center text-caption text-grey-7">
      <div>Versión 1.0</div>
      <div class="text-caption">© 2024 SIGAV</div>
    </q-footer>
  </q-drawer>
</template>
<script setup lang="ts">
import DrawerComponent from 'src/components/DrawerComponent.vue';
import { ref, onMounted, computed } from 'vue';
import { api } from 'src/boot/axios';
import { useRouter } from 'vue-router';

const user = ref({ role: 'invitado', isAdmin: false, isViewerOnly: false });
const router = useRouter();
const leftDrawerOpen = ref(false);
interface UserData {
  email: string;
  role: string;
}

const usuario = ref<UserData>({
  email: '',
  role: '',
});

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
    usuario.value = response.data;
    // Verificar si la petición fue exitosa
    if (response.status === 200) {
      user.value.role = response.data.role;
      user.value.isAdmin = response.data.role === 'admin';
      user.value.isViewerOnly = response.data.role === 'invitado';
    } else {
      console.error(
        `Error al obtener los datos del usuario: Estado ${response.status}`
      );
    }
  } catch (error) {
    console.error('Error al obtener los datos del usuario:', error);
  }
};

const logout = () => {
  localStorage.removeItem('authToken');
  router.push('/');
};

const toggleLeftDrawer = () => {
  leftDrawerOpen.value = !leftDrawerOpen.value;
};

const abbreviatedUsername = computed(() => {
  if (!usuario.value.email) return 'Usuario';

  // Tomar la parte antes del @ si es email
  const username = usuario.value.email.split('@')[0];

  // Acortar a 15 caracteres máximo
  return username.length > 10 ? `${username.substring(0, 9)}...` : username;
});

const formattedRole = computed(() => {
  switch (usuario.value.role.toLowerCase()) {
    case 'admin':
      return 'Admin';
    case 'especialista':
      return 'Especialista';
    case 'invitado':
      return 'Invitado';
    default:
      return 'Usuario';
  }
});

onMounted(fetchUserData);
</script>

<style scoped>
.bg-gradient-primary {
  background: linear-gradient(145deg, #2c3e50, #3498db);
}

.q-header {
  transition: all 0.3s ease;
}

.q-menu {
  border-radius: 12px;
}

.q-item:hover {
  background: rgba(0, 0, 0, 0.05);
  transform: translateX(5px);
  transition: all 0.2s ease;
}

.q-drawer {
  box-shadow: 4px 0 15px rgba(0, 0, 0, 0.1);
}

.q-footer {
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}
</style>
