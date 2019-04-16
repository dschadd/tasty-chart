<template>
  <div class="home">
    <input v-model="search" type="text" name="Search" placeholder="Search" />
    <button v-on:click="searchCompanies">Submit</button> {{ list }}
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
      list: [],
      search: ""
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
      const header = rowData[0].replace(/\"/g, "").split(",");
      // create array of objects that have a key of headers linked to specific company information
      const list = rowData.map(row => {
        // parse the rowData element to remove quotes ("") from the string
        // create new array which is an array containing arrays of each company's data
        const rowArray = row.replace(/\"/g, "").split(",");
        // initialize object for storing this item's data
        let obj = {};
        // set each header's key value pair to corresponding company data
        header.forEach((key, i) => {
          obj[key] = rowArray[i];
        });
        return obj;
      });
      // sets data list to array of company objects
      this.list = list;
      console.log(list);
    });
  },
  methods: {
    searchCompanies: function() {
      const companyNames = this.companyNames;
      console.log(companyNames);
    }
  },
  computed: {}
};
</script>
