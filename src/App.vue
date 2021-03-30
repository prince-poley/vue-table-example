<template>
  <div id="app">
    <div class="container mt-5">
      <div class="table-data-choice d-flex justify-content-between">
        
        <h2>Размер данных для загрузки</h2>
        
        <div class="table-data-choice-buttons">
          <button 
            @click="getData('small')"
            type="button" 
            class="btn btn-primary btn-sm"
          >Маленький</button>

          <button
            @click="getData('large')"
            type="button" 
            class="btn btn-primary btn-sm"
          >Большой</button>
        </div>
      </div>

      <transition name="fade" mode="out-in">
        <loader v-if="isLoading" />
        <data-table v-else :tableData="tableData" />
      </transition>
    </div>
  </div>
</template>

<script>

import DataTable from "@/components/DataTable/DataTable.vue"
import Loader from "@/components/Loader/Loader.vue"
import axios from "axios"

export default {
  name: 'App',
  components: {
    DataTable,
    Loader
  },
  data() {
    return {
      tableData: [],
      isLoading: false
    }
  },
  methods: {
    getData(size) {
      this.isLoading = true;

      let loadDataUrl = size === "large"
        ? "http://www.filltext.com/?rows=1000&id={number|1000}&firstName={firstName}&delay=3&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&address={addressObject}&description={lorem|32}"
        : "http://www.filltext.com/?rows=32&id={number|1000}&firstName={firstName}&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&address={addressObject}&description={lorem|32}"
        
        axios.get(loadDataUrl)
          .then(({ data }) => {
            this.tableData = data;
            this.isLoading = false;
          })
          .catch(error => {
            console.log(error.message);
          })
    }
  },
}
</script>

<style scoped>
  .table-data-choice {
    margin-bottom: 1rem;
  }

  .table-data-choice button {
    margin-right: .5rem;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>