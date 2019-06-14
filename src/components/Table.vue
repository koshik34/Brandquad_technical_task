<template>
  <div>
    <table class="tftable" border="1">
      <tr>
        <th v-for="field in fields" @click="sort_table(field)">{{field}}</th>
      </tr>
      <tr v-for="(item,i) in list" v-if="i>=selected_page*10-10&&i<selected_page*10">
        <td v-for="field in fields" >{{list[i][field]||field}}</td>
      </tr>
    </table>
    <div class="row">
      <div class='button' @click='show_modal_window=!show_modal_window'>Show JSON</div>
      <div class=''> {{show_modal_window}}</div>
      <div v-if="numbers_page>1" class='numbers_page'>
        <div class="number"  v-for="number in numbers_page" @click="selected_page=number" v-bind:class="{active:number==selected_page}">{{number}}</div>
      </div>
    </div>
    <div class='wrap_modal_window' v-if="show_modal_window">
      <div class='modal_window'>
        <div class='block_json'>{{JSON.stringify(list)}}</div>
        <div class='button'>Copy</div>
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
      selected_page:1,
      show_modal_window:false
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
.wrap_modal_window
{
  position: fixed;
  width: 100%;
  text-align: center;
  height: auto;
  top: 0px;
  bottom: 0px;
  left: 0px;
  right: 0px;
  box-sizing: border-box;
  z-index: 100;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
}
.modal_window
{
  width: 360px;
  border: 1px solid #bcbcbc;
  background-color: white;
}
.button
{
  border: 1px solid #bcbcbc;
  background-color: white;
  padding: 2px 10px;
  cursor: pointer;
  text-decoration: none;
  font-weight: bold;
  border-radius: 6px;
  transition: all 0.5s ease;
  box-sizing: border-box;
  align-items: center;
  color: #757575;
  display:flex;
  width:auto;
  align-items: center;
  justify-content: space-around;
}
.block_json
{
  padding: 15px;
    max-height: 300px;
    overflow-y: auto;
    overflow-x: hidden;
    text-align: left;
    width: 100%;
    box-sizing: border-box;
}
</style>
