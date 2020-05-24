<template>
  <v-app light>
    <v-navigation-drawer v-model="drawer" :mini-variant="miniVariant" clipped fixed app light>
      <v-list nav dense>
        <v-subheader>Menu</v-subheader>
        <template v-for="item0 in lists">
          <v-list-item v-if="!item0.lists" :key="item0.title" :to="item0.to" router exact>
            <v-list-item-action>
              <v-icon>{{ item0.icon }}</v-icon>
            </v-list-item-action>
            <v-list-item-content>
              <v-list-item-title v-text="item0.title"></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-group v-else :key="item0.title" no-action>
            <template v-slot:activator>
              <v-list-item-title>{{ item0.title }}</v-list-item-title>
            </template>
            <v-list-item
              v-for="item1 in item0.lists"
              :key="item1.title"
              :to="item1.to"
              router
              exact
            >
              <!-- <v-list-item-action>
                <v-icon>{{ item1.icon }}</v-icon>
              </v-list-item-action>-->
              <v-list-item-content>
                <v-list-item-title v-text="item1.title"></v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-group>
        </template>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar color="primary" clipped-left fixed app dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />

      <v-spacer />

      <v-btn icon href="https://github.com/Maru0137/dqrta-info">
        <v-icon>mdi-github</v-icon>
      </v-btn>
      <v-btn icon href="https://twitter.com/Maru0137">
        <v-icon>mdi-twitter</v-icon>
      </v-btn>
    </v-app-bar>
    <v-content>
      <v-container>
        <v-breadcrumbs :items="bread" />
        <nuxt />
      </v-container>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import Vue, { PropOptions } from "vue";

export default Vue.extend({
  data() {
    console.log(this.$route.matched);
    return {
      drawer: false,
      fixed: false,
      lists: [
        {
          title: "SFC Dragon Quest 3",
          to: "",
          active: false,
          lists: [
            {
              title: "Rand Simulator",
              icon: "mdi-chart-bell-curve",
              to: "/dq3/rand_simulator"
            },
            {
              title: "Physical Damage Simulator",
              icon: "mdi-sword-cross",
              to: "/dq3/damage_simulator"
            }
          ]
        }
      ],
      miniVariant: false,
      title: "Dragon Quest RTA Tools"
    };
  },

  computed: {
    bread(): Object[] {
      const route: string = this.$route.path;
      const splited = route.split("/").slice(1);
      const items = splited.map(str => {
        return {
          text: str
        };
      });
      return items;
    }
  }
});
</script>
