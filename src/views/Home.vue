<!--
  TODO
  Fix variable names
  Dynamic typeahead search
  Chart candlestick price
  Ask Peter, Dan, and Jay what I can do to make the code better
  Testing?z
-->
<template>
  <div class="home">
    <!-- <label for="input">Companies:</label> -->
    <!-- <input id="input" class="form-control" type="text" placeholder="Type to search..." /> -->
    <!-- <typeahead v-model="search" target="#input" :data="list" item-key="name" /> -->
    <!-- <alert v-show="search">You selected {{ search }}</alert> -->

    <select v-model="search">
      <option v-for="company in companies" v-bind:value="company">{{ company.Name }}</option>
    </select>
    <button v-on:click="searchCompanies">Search</button>

    <h1>{{ search.Name }}</h1>
    <h2>{{ search.Symbol }}</h2>
    <h2>${{ price }}</h2>
  </div>
</template>

<style></style>

<script>
import axios from "axios";

export default {
  data: function() {
    return {
      companyNames: ["Apple", "Google", "Microsoft"],
      header: [],
      companies: [],
      search: "",
      price: ""
    };
  },
  created: function() {
    // TODO: remove empty last element of header array
    // import the CSV of listed stocks
    axios.get("./data.csv").then(res => {
      // parse CSV to store separate companies as elements of an array
      const rowData = res.data.split("\n");
      // parse the first element of the rowData array to remove quotes ("") from the string
      // create new header array from first row of CSV data
      const header = rowData[0].replace(/"/g, "").split(",");
      // create array of objects that have a key of headers linked to specific company information
      const companies = rowData.map(row => {
        // parse the rowData element to remove quotes ("") from the string
        // create new array which is an array containing arrays of each company's data
        const rowArray = row.replace(/"/g, "").split(",");
        // initialize object for storing this item's data
        let obj = {};
        // set each header's key value pair to corresponding company data
        header.forEach((key, i) => {
          obj[key] = rowArray[i];
        });
        return obj;
      });
      // sets data list to array of company objects
      this.companies = companies;
    });
  },
  methods: {
    searchCompanies: function() {
      const search = this.search;
      axios.get(`https://api.iextrading.com/1.0/stock/${search.Symbol}/chart/1m`).then(res => {
        this.price = res.data[0]["close"];
      });
    }
  },
  computed: {}
};
</script>
