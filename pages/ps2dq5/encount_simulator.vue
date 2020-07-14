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
                  <v-col cols="8">
                    <v-select 
                      v-model="tableId"
                      label="Area"
                      :items="areas"
                      item-text="display"
                      item-value="id"
                      dense
                      required
                      ></v-select>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      v-model.number="sample"
                      label="The number of sample"
                      type="number"
                      :min="1"
                      :rules="inboundRules(sample, 1)"
                      dense
                      required
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row>
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
      dists: new Array<number>(),
      areaNames: [
        "サンタローズの洞窟 1F [幼年期]",
        "サンタローズの洞窟 B1~B2F [幼年期]",
        "フィールド アルカパ周辺 [幼年期]",
        "フィールド レヌール城周辺 [幼年期]",
        "レヌール城 1大広間~3F",
        "レヌール城 1F西の塔~B1F",
        "妖精界 南部",
        "妖精界 北部",
        "ドワーフの洞窟 B1~B2F",
        "ドワーフの洞窟 B3F",
        "氷の館",
        "フィールド サンタローズ周辺 [幼年期]",
        "フィールド ラインハット周辺 [幼年期]",
        "フィールド ラインハット東部 [幼年期]",
        "フィールド 古代の遺跡周辺 [幼年期]",
        "古代の洞窟 広間",
        "古代の洞窟 水路",
        "フィールド オラクルベリー周辺 [青年期前半]",
        "フィールド サンタローズ周辺  [青年期]",
        "フィールド レヌール城西部  [青年期]",
        "サンタローズの洞窟 1~B1F  [青年期]",
        "サンタローズの洞窟 B2~B4F  [青年期]",
        "フィールド ラインハット周辺  [青年期]",
        "ラインハット地下",
        "フィールド 神の塔周辺",
        "神の塔 1~2F",
        "神の塔 3~4F",
        "フィールド ポートセルミ周辺",
        "フィールド ルラフェン周辺",
        "魔物の住処 B1F",
        "魔物の住処 B2~B3F",
        "フィールド ルラフェン北部",
        "フィールド ルラフェン西部",
        "フィールド うわさのほこら周辺",
        "ルラフェン南の洞窟",
        "フィールド サラボナ周辺",
        "フィールド 死の火山周辺",
        "死の火山 B1F",
        "死の火山 B2~B3F",
        "滝の洞窟 B1~B4F",
        "滝の洞窟 B5~B7F",
        "滝の洞窟 最深部",
        "フィールド テルパドール周辺",
        "フィールド メダル王の城周辺",
        "フィールド ネッドの宿屋周辺",
        "チゾットへの山道 外観",
        "チゾットへの山道 内部",
        "グランバニア山の洞窟 6~10F",
        "グランバニア山の洞窟 4~5F",
        "グランバニア山の洞窟 1~3F",
        "フィールド グランバニア周辺",
        "試練の洞窟",
        "デモンズタワー 1~6F",
        "デモンズタワー 8~9F",
        "フィールド 名産品博物館周辺 [青年期前半]",
        "フィールド エルヘブン周辺",
        "フィールド エルヘブン北部",
        "海の神殿",
        "フィールド 天空への塔周辺 [青年期後半]",
        "天空への塔 外観&1F",
        "天空への塔 内部",
        "地下遺跡の洞窟 1~B1F",
        "地下遺跡の洞窟 B2~B3F",
        "フィールド 迷いの森周辺 [青年期後半]",
        "迷いの森",
        "フィールド 封印の洞窟周辺 [青年期後半]",
        "未使用",
        "ボブルの塔 地上",
        "ボブルの塔 地下",
        "封印の洞窟 B1~B3F",
        "封印の洞窟 B4F",
        "大神殿",
        "魔界 ジャハンナ周辺",
        "魔界 エビルマウンテン周辺",
        "エビルマウンテン 1~4F(外観以前)",
        "エビルマウンテン 外観&5F",
        "エビルマウンテン 4F移動床フロア",
        "エビルマウンテン 3~4F(水差し~落とし穴フロア)",
        "エビルマウンテン 谷~祭壇",
        "謎の洞窟 B1~B2F",
        "謎の洞窟 B3~B4F",
        "フィールド サラボナ~滝の洞窟の内海 [青年期前半]",
        "フィールド 内海 [青年期前半]",
        "フィールド 海全域 [青年期後半]",
        "フィールド 天空への塔周辺海域 [青年期後半]"
      ]
    };
  },
  methods: {
    simulateEncounts() {
      const result = dq5.encount_simulation_to_js(this.tableId, this.sample);

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
    areas(): Object[] {
      const displays = this.areaNames.map((v: String, i: Number) => i.toString() + " - " + v);
      return displays.map(function(v: String, i: Number) {
        return {
          id: i, 
          display: v
        };
      });
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
