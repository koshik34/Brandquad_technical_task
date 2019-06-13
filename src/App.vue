<template>
  <div>
    <my-table :fields="fields" :rows="rows" :meta="meta"></my-table>
  </div>
</template>

<script>
import Table from "./components/Table.vue";


export default {
  name: "app",
  data() {
    return {
      fields: [],
      rows: 5,
      meta: ["class_1", "class_2", "class_3"],
    };
  },
  
  methods: {
   
  },
  components: {
    "my-table": Table
  }
};
</script>

<style>

</style>
