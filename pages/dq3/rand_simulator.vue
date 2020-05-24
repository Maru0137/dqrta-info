<template>
  <v-layout column justify-center align-centor>
    <v-container fluiid>
      <h1>Rand Simulator</h1>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>Parameters</v-card-title>
            <v-card-text>
              <v-form ref="form" v-model="validForm">
                <v-row>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="upper"
                      label="Upper value in a trial"
                      type="number"
                      :min="0"
                      :max="255"
                      :rules="inboundRules(upper, 0, 255)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="trial"
                      label="The number of trials"
                      type="number"
                      :min="1"
                      :rules="inboundRules(trial, 1)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="offset"
                      label="offset value"
                      type="number"
                      :min="0"
                      :max="255"
                      :rules="inboundRules(offset, 0, 255)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="sample"
                      label="The number of samples"
                      type="number"
                      :min="1"
                      :rules="inboundRules(sample, 1)"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="6">
                    <v-btn
                      color="success"
                      @click="simulateHistogram"
                      :disabled="!validForm"
                    >Simulate</v-btn>
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
      upper: 0xff,
      trial: 1,
      offset: 0,
      sample: 1000000,
      dists: new Array<number>()
    };
  },
  methods: {
    simulateHistogram() {
      const modulo = 256;
      const bin_num = Math.min(modulo, this.upper * this.trial + 1);
      let histogram = new Array<number>(bin_num).fill(0);

      let rng = dq3.Rng.init();
      for (let i = 0; i < this.sample; ++i) {
        let rng_in_sample = rng;

        let sum: number = this.offset as number;
        for (let j = 0; j < this.trial; ++j) {
          sum += rng_in_sample.rand_by_multiply(this.upper);
        }

        const ans = sum % modulo;
        histogram[ans] += 1;
        rng.rand();
      }

      this.dists = histogram.map(v => (v / this.sample) * 100);
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
        labels: [...this.dists.keys()],
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
