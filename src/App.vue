<template>
  <v-app :style="{background: $vuetify.theme.themes[theme].background}">
    <v-main>
      <Configurator/>
    </v-main>
  </v-app>
</template>

<script>
import Configurator from './components/Configurator';

export default {
  name: 'App',
computed:{
    theme(){
      return (this.$vuetify.theme.dark) ? 'dark' : 'light'
    }
  },
  components: {
   Configurator,
  },

  data: () => ({
    //
  }),
};
</script>
