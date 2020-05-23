<template>
  <v-layout column justify-center align-centor>
    <v-container fluiid>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>Parameters</v-card-title>
            <v-card-text>
              <v-form ref="form" v-model="validForm">
                <v-row>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="atk"
                      label="攻撃力"
                      type="number"
                      :min="0"
                      :max="65535"
                      :rules="inboundRules(atk, 0, 65535)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="def"
                      label="守備力"
                      type="number"
                      :min="0"
                      :max="65535"
                      :rules="inboundRules(def, 0, 65535)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="sample"
                      label="サンプル数"
                      type="number"
                      :min="1"
                      :rules="inboundRules(sample, 1)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-checkbox v-model="twinhits" label="バイキルト" dense></v-checkbox>
                  </v-col>
                  <!-- <v-col cols="4">
                    <v-checkbox v-model="defence" label="防御" dense></v-checkbox>
                  </v-col>-->
                  <v-col cols="4">
                    <v-checkbox v-model="byMonster" label="モンスターの攻撃" dense></v-checkbox>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-btn color="success" @click="simulateDamage" :disabled="!validForm">Simulate</v-btn>
                  </v-col>
                </v-row>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>Results</v-card-title>
            <v-card-text>
              <BarChart :chartData="chartData" :options="options"></BarChart>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-layout>
</template>

<script lang="ts">
import Vue, { PropOptions } from "vue";
import colors from "vuetify/lib/util/colors";
import BarChart from "~/components/BarChart.vue";
import Chart from "chart.js";
import * as dq3 from "@maru0137/dq3";

export default Vue.extend({
  components: {
    BarChart
  },
  data() {
    return {
      validForm: true,
      atk: 360,
      def: 110,
      sample: 1000000,
      twinhits: false,
      defence: false,
      byMonster: false,
      labels: new Array<string>(),
      dists: new Array<number>()
    };
  },
  methods: {
    simulateDamage() {
      let battle = new dq3.Battle(dq3.Rng.init());
      let histogram: { [value: number]: number } = {};
      let attacker = this.byMonster
        ? dq3.Character.Monster
        : dq3.Character.Player;

      for (let i = 0; i < this.sample; ++i) {
        const damage = battle.physical_damage(
          this.atk,
          this.def,
          this.twinhits,
          attacker
        );

        if (!histogram[damage]) {
          histogram[damage] = 0;
        }
        histogram[damage] += 1;
      }

      let keys = Object.keys(histogram).map(v => Number.parseInt(v));
      let values = Object.values(histogram);

      const min = keys.reduce((acc, v) => Math.min(acc, v));
      const max = keys.reduce((acc, v) => Math.max(acc, v));
      const extent = max - min + 1;

      this.labels = [...Array(extent).keys()].map(v => (min + v).toFixed());
      this.dists = [...Array(extent).keys()].map(
        v => ((histogram[min + v] ? histogram[min + v] : 0) / this.sample) * 100
      );
    },

    inboundRules(
      v: number,
      min: number = Number.NEGATIVE_INFINITY,
      max: number = Number.POSITIVE_INFINITY
    ): (string | true)[] {
      return [(min <= v && v <= max) || `Value must be in [${min}, ${max}]`];
    }
  },
  computed: {
    validateForm(): Vue & { validate: () => boolean } {
      return this.$refs.form as Vue & { validate: () => boolean };
    },

    chartData(): Chart.ChartData {
      return {
        labels: this.labels,
        datasets: [
          {
            label: "Probability",
            backgroundColor: colors.blue.base,
            data: this.dists
          }
        ]
      };
    },

    options(): Chart.ChartOptions {
      const percentFormat = (v: number) => v.toFixed(3) + "%";
      return {
        scales: {
          xAxes: [
            {
              gridLines: {
                display: false
              }
            }
          ],
          yAxes: [
            {
              ticks: {
                beginAtZero: true,
                callback: (label: number, index: number, labels: number[]) =>
                  percentFormat(label)
              }
            }
          ]
        },
        tooltips: {
          callbacks: {
            label: (
              tooltipItem: Chart.ChartTooltipItem,
              data: Chart.ChartData
            ) => {
              if (typeof tooltipItem.yLabel === "number") {
                return percentFormat(tooltipItem.yLabel);
              } else if (typeof tooltipItem.yLabel === "undefined") {
                return "undefined";
              }
              return tooltipItem.yLabel;
            }
          }
        }
      };
    }
  }
});
</script>
