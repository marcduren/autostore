<template>
  <div class="d-flex justify-center align-center">
    <v-card style="max-width:500px;">
      <v-card-title>
        <img alt="Vue logo" src="../assets/logo.png" width="250" />
      </v-card-title>
      <v-card-text>
        <h2>Your Informatique Company</h2>
        <table class="subheading">
          <tr>
            <th align="right">Vuejs</th>
            <td>{{ vueVersion }}</td>
          </tr>
          <tr>
            <th align="right">Vuetify</th>
            <td>{{ vuetifyVersion }}</td>
          </tr>
          <tr>
            <th align="right">Developer</th>
            <td>Marc DUREN</td>
          </tr>
          <tr>
            <th>Open source</th>
            <td><a href="https://github.com/marcduren/autostore">Github</a></td>
          </tr>
        </table>
      </v-card-text>
    </v-card>
  </div>
</template>
<script>
import Vuetify from 'vuetify/lib'
import Vue from 'vue'

export default {
  name: 'About',
  data() {
    return {
      vuetifyVersion: Vuetify.version,
      vueVersion: Vue.version
    }
  }
}
</script>
