<template>
  <v-layout column justify-center align-centor>
    <v-container fluiid>
      <h1>Encount Simulator</h1>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>Parameters</v-card-title>
            <v-card-text>
              <v-form ref="form" v-model="validForm">
                <v-row>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="tableId"
                      label="Encount table id"
                      type="number"
                      :min="0"
                      :max="84"
                      :rules="inboundRules(tableId, 0, 84)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="sample"
                      label="The number of sample"
                      type="number"
                      :min="1"
                      :rules="inboundRules(sample, 1)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="6">
                    <v-btn color="success" @click="simulateEncounts" :disabled="!validForm">Simulate</v-btn>
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
import * as dq5 from "@maru0137/ps2dq5";

export default Vue.extend({
  components: {
    BarChart
  },
  data() {
    return {
      validForm: true,
      tableId: 0,
      sample: 1000000,
      labels: new Array<string>(),
      dists: new Array<number>()
    };
  },
  methods: {
    simulateEncounts() {
      const result = dq5.encount_simulation_to_js(this.tableId, this.sample);
      console.log(result);

      const monsters = result.monster_dists;
      this.labels = monsters.map((x: any) =>
        dq5.monster_name_from_id(x[0] as number)
      );
      this.dists = monsters.map((x: any) => x[1] as number);
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
            backgroundColor: [
              colors.red.lighten2,
              colors.orange.lighten2,
              colors.yellow.lighten2,
              colors.green.lighten2,
              colors.blue.lighten2,
              colors.purple.lighten2,
              colors.deepPurple.lighten2
            ],
            data: this.dists
          }
        ]
      };
    },
    options(): Chart.ChartOptions {
      const percentFormat = (v: number) => (v * 100).toFixed(3) + "%";
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
        },
        legend: {
          display: false
        }
      };
    }
  }
});
</script>
