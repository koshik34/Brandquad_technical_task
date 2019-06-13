<template>
  <div>
    <table class="tftable" border="1">
      <tr>
        <th v-for="field in fields" @click="sort_table(field)">{{field}}</th>
      </tr>
      <tr v-for="(item,i) in list">
        <td v-for="field in fields" v-if="i>=selected_page*10-10&&i<selected_page*10">{{list[i][field]||field}}</td>
      </tr>
    </table>
    <div class="row">
      <div v-if="numbers_page>1" class='numbers_page'>
        <div class="number"  v-for="number in numbers_page" @click="selected_page=number" v-bind:class="{active:number==selected_page}">{{number}}</div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import lodash from "lodash";
export default {
  props: {
    fields: Array,
    rows: Number,
    meta: Array
  },
  data() {
    return {
      list:[],
      numbers_page:null,
      selected_page:1
    };
  },
  methods:{
    add_row(){
      list.push()
    },
    sort_table(name){

    },
    set_page(number){
      this.selected_page=number;
    }
  },
  mounted() {
    if(this.fields.length===0||!this.rows||this.meta.length===0)
    {
      // http://www.filltext.com/?rows=1000&id={number|1000}&firstName={firstName}&delay=3&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&adress={addressObject}&description={lorem|32} 
      axios.get('http://www.filltext.com/?rows=32&id={number|1000}&firstName={firstName}&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&adress={addressObject}&description={lorem|32}').then(response => {
       
        this.list=response.data;
        this.rows=this.list.length;
        console.log(this.list);
        this.numbers_page=Math.floor(this.list.length/10);
        var obj=response.data[0];
        for(var key in obj) {
          if (lodash.isObject(obj[key])){
            for (var innerKey in obj[key]){
              this.fields.push(key+'.'+innerKey);
            }
          }
          else
           this.fields.push(key);
        }
      });
    }
  },
};
</script>

<style>
.row
{
  margin-top:2em;
  width:100%;
  display:flex;
  align-items: center;
  justify-content: space-between;
}
.tftable {
  border-width: 1px;
  border-color: black;
  border-collapse: collapse;
}
.tftable th 
{
  cursor:pointer;
}
.numbers_page
{
  display:flex;
}
.number
{
  width: 1.5em;
  height:1.5em;
  display:flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  border:1px solid black;
  cursor:pointer;
}
.number.active
{
  background-color: aqua;
}
</style>
