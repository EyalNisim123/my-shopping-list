<template>
  <v-app>
    <v-main>
      <MyShopping/>
    </v-main>
  </v-app>
</template>

<script>

import MyShopping from './components/MyShopping';

export default {
  name: 'App',

  components: {
    MyShopping,
  }
};

</script>
