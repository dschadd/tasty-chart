<!--
  TODO
  Fix variable names
  Ask Peter, Dan, and Jay what I can do to make the code better
  Reduce libraries and dependencies
  Testing?
-->
<template>
  <div class="home">
    <vue-bootstrap-typeahead
      v-model="search"
      :data="companies"
      :serializer="company => `${company.Name} (${company.Symbol})`"
      @hit="searchCompanies"
    />

    <highcharts :constructor-type="'stockChart'" :options="chartOptions"> </highcharts>
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
      header: [],
      companies: [],
      search: "",
      price: "",
      chartOptions: {
        series: [
          {
            type: "candlestick",
            data: []
          }
        ]
      }
    };
  },
  components: {
    VueBootstrapTypeahead,
    highcharts: Chart
  },
  created() {
    axios.get("./data.csv").then(res => {
      const rowData = res.data.split("\n");

      const header = rowData[0].replace(/"/g, "").split(",");

      const companyData = rowData.slice(1, rowData.length);

      const companies = companyData.map(row => {
        const rowArray = row.replace(/"/g, "").split(",");
        let obj = {};
        header.forEach((key, i) => {
          obj[key] = rowArray[i];
        });
        return obj;
      });

      this.companies = companies.filter(company => company.Name !== undefined);
    });
  },
  methods: {
    // Function to get array of company data objects
    getCompanies() {
      axios.get("./data.csv").then(res => {
        this.companies = this.parseCompanies(res.data);
      });
    },
    parseCompanies(data) {
      // Function to parse CSV from assets folder and return an array of
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
    searchCompanies(value) {
      const search = value;
      const symbol = search.Symbol;
      axios
        .get(`https://api.iextrading.com/1.0/stock/${symbol}/chart/1y`)
        .then(res => {
          console.log(res.data);
          this.fillData(res.data);
        })
        .catch(err => console.log("not found"));
    },
    fillData(data) {
      const chartData = data.map(day => {
        return [day.date, day.open, day.high, day.low, day.close];
      });
      console.log(chartData);

      const chartOptions = {
        series: [
          {
            type: "candlestick",
            data: chartData // sample data
          }
        ]
      };
      this.chartOptions = chartOptions;
    }
  },
  computed: {}
};
</script>
