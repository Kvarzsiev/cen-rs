<template>
  <div class="container">
    <Bar v-if="loaded" :chart-data="chartData" :chart-options="chartOptions" />
  </div>
</template>
<script>
import { toNumber } from "@vue/shared";
import axios from "axios";
import {
  BarElement,
  CategoryScale,
  Chart as ChartJS,
  Legend,
  LinearScale,
  Title,
  Tooltip,
} from "chart.js";
import { inject } from "vue";
import { Bar } from "vue-chartjs";
ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
);

export default {
  name: "RelativeChartComponent",
  data: () => ({
    loaded: false,
    chartData: null,
    chartOptions: {
      responsive: true,
      maintainAspectRatio: false,
    },
  }),
  components: {
    Bar,
  },
  props: {
    chartId: {
      type: String,
      default: "bar-chart",
    },
    width: {
      type: Number,
      default: 190,
    },
    height: {
      type: Number,
      default: 390,
    },
    cssClasses: {
      default: "",
      type: String,
    },
  },
  async mounted() {
    const entidades = this.getEntities();
    var dataSets = [];
    for (let entidade of await entidades) {
      let dataSet = {
        label: entidade.nom_entidade,
        backgroundColor: `#${Math.floor(
          toNumber(String(entidade.cod_entidade) * 200)
        ).toString(16)}`,
        data: [
          toNumber(
            String(entidade.pop_relativa_urbana_total).replaceAll(" ", "")
          ),
          toNumber(
            String(entidade.pop_relativa_urbana_sede).replaceAll(" ", "")
          ),
        ],
      };
      if (dataSets.length < 11) {
        dataSets.push(dataSet);
      } else {
        break;
      }
    }
    const chartData = {
      labels: ["Pop. Relativa Urbana", "Pop. Rel. Urbana na Sede Municipal"],
      datasets: [...dataSets],
    };
    this.chartData = chartData;
    this.loaded = true;
  },
  methods: {
    async getEntities() {
      let filtros = await inject("filtros").value;
      var params = {
        entity_codes: filtros.entity_codes,
      };
      const response = await axios.get(
        "http://localhost:2512/cen-rs/entidades",
        {
          headers: { "Access-Control-Allow-Origin": "*" },
          params: {
            ...params,
          },
        }
      );
      return response.data;
    },
  },
};
</script>
