<template>
  <body class="q-pa-md">
    <q-page-section>
      <div class="full-height">
        <CardListAval />
      </div>
    </q-page-section>

    <q-page-section>
      <div class=""></div>
    </q-page-section>
  </body>
</template>

<script setup>
import ChartComponent from 'src/components/ChartComponent.vue';
import CardListAval from 'src/components/CardListAval.vue';

// You can use the imported component here
</script>
<style scoped>
.div3 {
  margin: 0 auto;
}
.div4 {
  margin: 0 auto;
  max-height: 400px;
}
.div2 {
  width: 500px;
  height: 250px;
}
</style>
