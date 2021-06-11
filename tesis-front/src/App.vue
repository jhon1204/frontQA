<template>
  <v-app>
    <v-main>
      <NavBar/>
      <Map/>
    </v-main>
  </v-app>
</template>

<script>
import NavBar from './components/NavBar.vue';
import Map from './components/Map.vue';

export default {
  name: 'App',

  components: {
    Map,
    NavBar
  },

  data: () => ({
    //
  }),
};
</script>
