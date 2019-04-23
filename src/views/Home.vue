<!--
  TODO
  Testing
-->
<template>
  <div class="home">
    <h1>Tasty Chart</h1>
    <p>Search for a stock!</p>

    <div class="container">
      <vue-bootstrap-typeahead
        v-model="search"
        :data="companies"
        :serializer="company => `${company.Name} (${company.Symbol})`"
        @hit="searchCompanies"
      />
    </div>

    <div v-if="search" class="container">
      <highcharts :constructor-type="'stockChart'" :options="chartOptions"> </highcharts>
    </div>
  </div>
</template>

<style></style>

<script>
import axios from "axios";
import VueBootstrapTypeahead from "vue-bootstrap-typeahead";
import Highcharts from "highcharts";
import stockInit from "highcharts/modules/stock";
import { Chart } from "highcharts-vue";

stockInit(Highcharts);

export default {
  data() {
    return {
      companies: [],
      search: "",
      chartOptions: {
        rangeSelector: {
          enabled: false
        },
        scrollbar: {
          enabled: false
        },
        navigator: {
          enabled: false
        }
      }
    };
  },
  components: {
    VueBootstrapTypeahead,
    highcharts: Chart
  },
  created() {
    this.getCompanies();
  },
  methods: {
    // Get array of company data objects
    getCompanies() {
      axios.get("./data.csv").then(res => {
        this.companies = this.parseCompanies(res.data);
      });
    },
    parseCompanies(data) {
      // Parse CSV from assets folder and return an array of
      // objects with key-value pair of a header and company data
      const rowData = data.split("\n");

      const headers = rowData[0].replace(/"/g, "").split(",");

      const companyData = rowData.slice(1, rowData.length);

      const companies = companyData.map(row => {
        const rowArray = row.replace(/"/g, "").split(",");
        let obj = {};
        headers.forEach((key, i) => {
          obj[key] = rowArray[i];
        });
        return obj;
      });
      return companies.filter(company => company.Name !== undefined);
    },
    searchCompanies(search) {
      // API call to get yearly stock data for the searched company
      const symbol = search.Symbol;
      axios
        .get(`https://api.iextrading.com/1.0/stock/${symbol}/chart/1m`)
        .then(res => {
          this.fillChartData(res.data);
        })
        .catch(err => console.log("not found"));
    },
    fillChartData(data) {
      // Formats API data for Highcharts Candlestick chart and then
      // fills chart options AND data
      const chartData = data.map(day => {
        return [day.date, day.open, day.high, day.low, day.close];
      });

      const chartOptions = {
        title: {
          text: `${this.search}`
        },
        rangeSelector: {
          enabled: false
        },
        scrollbar: {
          enabled: false
        },
        navigator: {
          enabled: false
        },
        xAxis: {
          labels: {
            enabled: false
          }
        },
        yAxis: {
          labels: {
            format: "${value}"
          }
        },
        series: [
          {
            type: "candlestick",
            data: chartData
          }
        ]
      };
      this.chartOptions = chartOptions;
    }
  },
  computed: {}
};
</script>
