<template>
  <v-app v-on:keydown.up="alert('up')">
    <v-navigation-drawer v-model="drawer" app clipped>
      <v-list>
        <v-subheader>AUTOSTORE</v-subheader>
        <v-list-item link to="/">
          <v-list-item-action>
            <v-icon>mdi-gamepad-variant-outline</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Playing</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item link to="/howto">
          <v-list-item-action>
            <v-icon>mdi-help</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>How to play</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item link to="/about">
          <v-list-item-action>
            <v-icon>mdi-information-variant</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>About</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar app clipped-left flat dense color="indigo">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title>AutoStore Game</v-toolbar-title>
    </v-app-bar>
    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    drawer: null
  }),
  created() {
    this.$vuetify.theme.dark = true
  }
}
</script>
