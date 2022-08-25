<template>
  <v-app>
    <v-main>
      <CalculatorApp/>
    </v-main>
  </v-app>
</template>

<script>
import CalculatorApp from './components/CalculatorApp.vue';

export default {
  name: 'App',

  components: {
    CalculatorApp
},

  data: () => ({
    //
  }),
};
</script>
