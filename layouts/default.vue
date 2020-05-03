<template>
  <v-app light>
    <v-navigation-drawer
      v-model="drawer"
      color="primary"
      :mini-variant="miniVariant"
      clipped
      fixed
      app
      dark
    >
      <v-list>
        <v-list-item v-for="(item, i) in items" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar color="primary" clipped-left fixed app dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-breadcrumbs items="breadcrumbs" />
    <v-content>
      <v-container>
        <nuxt />
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      drawer: false,
      fixed: false,
      items: [
        {
          icon: "mdi-chart-bell-curve",
          title: "SFCDQ3 Rand Simulator",
          to: "/"
        }
      ],
      miniVariant: false,
      title: "SFCDQ3 Rand Simulator"
    };
  }
};
</script>
