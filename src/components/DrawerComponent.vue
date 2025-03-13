<template>
  <div>
    <q-item to="/home" clickable v-close-popup>
      <q-item-section>
        <q-item-label class="text-weight-bolder">Inicio</q-item-label>
      </q-item-section>
    </q-item>
    <q-expansion-item
      v-for="category in categorizedItems"
      :key="category.title"
      :label="category.title"
      expand-separator
      :content-inset-level="0.5"
      class="text-weight-bolder"
    >
      <q-list style="padding-bottom: 12px">
        <q-item
          v-for="item in category.items"
          :key="item.label"
          :to="item.route"
          clickable
          v-close-popup
          class="text-weight-bolder"
        >
          <q-item-section>
            <q-item-label>{{ item.label }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-expansion-item>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, computed } from 'vue';
import { api } from 'src/boot/axios';

// Datos del usuario
const user = ref({ role: 'invitado', isAdmin: false, isViewerOnly: false });

interface NavigationItem {
  label: string;
  route: string;
}

interface Category {
  title: string;
  items: NavigationItem[];
}

// Categorías de navegación
const categories: Category[] = [
  {
    title: 'Avales',
    items: [
      { label: 'Avales de Tutorías', route: '/lista_avales_tuto' },
      { label: 'Avales de Publicación', route: '/lista_avales_public' },
      { label: 'Avales de Bibliografías', route: '/lista_avales_biblio' },
    ],
  },
  {
    title: 'Reportes',
    items: [
      { label: 'Avales por Departamento', route: '/Report_depart' },
      { label: 'Avales por Año', route: '/Report_fecha' },
      { label: 'Avales por Autor', route: '/Report_Autor' },
    ],
  },
];

// Función para obtener datos del usuario
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

    if (response.status === 200) {
      user.value.role = response.data.role;
      user.value.isAdmin = response.data.role === 'admin';
      user.value.isViewerOnly = response.data.role === 'especialista';
      console.log('Datos del usuario obtenidos correctamente.');
    } else {
      console.error(
        `Error al obtener los datos del usuario: Estado ${response.status}`
      );
    }
  } catch (error) {
    console.error('Error al obtener los datos del usuario:', error);
  }
};

// Items categorizados para la navegación
const categorizedItems = computed(() => categories);

onMounted(fetchUserData);
</script>
