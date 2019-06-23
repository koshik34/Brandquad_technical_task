<template>
	<div>
		<div>
			<div class='row f_s'>
				<div class='button' @click='show_modal_window=!show_modal_window'>Add table</div>
				<div class='loader' v-if='is_load'></div>
			</div>
			<div class='wrap_modal_window' v-if="show_modal_window" @click="show_modal_window=!show_modal_window; show_user_fields=false;">
				<div class='modal_window' @click.stop.prevent='' v-if='!show_user_fields'>
					<div>Load from</div>
					<div class='button' @click='load_data("little");'>Url 1</div>
					<div class='button' @click='load_data("big");'>Url 2</div>
					<div class='button' @click='show_user_fields="user_data"'>User data</div>
					<div class='button' @click='show_user_fields="json"'>JSON</div>
				</div>
				<div class='modal_window' @click.stop.prevent='' v-else>
					<div class="" v-if='show_user_fields==="user_data"'>
						<div>Name fields:</div>
						<textarea v-model="user_fields"> </textarea>
						<div>Count rows :</div>
						<input type="number" min="1" v-model="user_rows">
						<div>Class head :</div>
						<input v-model="user_meta_head">
						<div>Class even :</div>
						<input v-model="user_meta_even">
						<div>Class odd :</div>
						<input v-model="user_meta_odd">
						<div>
							<div class='button' @click=' load_data("user_data");show_user_fields=false'>Add table</div>
						</div>
					</div>
					<div class="" v-if='show_user_fields==="json"'>
						<textarea  v-model="user_json"> </textarea>
						<div class='button' @click='load_data("json");show_user_fields=false'>Add table</div>
					</div>
				</div>
			</div>
		</div>
		<div>
			<my-table v-for="(table,i) in all_tables" :key='i' :fields="table.fields" :rows="table.rows" :meta="table.meta" :list="table.list" :index_table="i" @remove_table='remove_table'></my-table>
		</div>
	</div>
</template>

<script>
import Table from "./components/Table.vue";
import axios from "axios";
import lodash from "lodash";

export default {
  name: "app",
  data() {
	return {
	  show_modal_window:false,
	  url:{
		little:'http://www.filltext.com/?rows=32&id={number|1000}&firstName={firstName}&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&adress={addressObject}&description={lorem|32}',
		big:'http://www.filltext.com/?rows=1000&id={number|1000}&firstName={firstName}&delay=3&lastName={lastName}&email={email}&phone={phone|(xxx)xxx-xx-xx}&adress={addressObject}&description={lorem|32}'
	  },
	  show_user_fields:false,
	  all_tables:[],
	  index_table:0,
	  user_fields:'',
	  user_rows:0,
	  user_meta_head:'th_style',
	  user_meta_even:'even_style',
	  user_meta_odd:'odd_style',
	  user_json:'',
	  is_load:false,
	};
  },
  
  methods: {
	get_fields(table)
	{
		var obj=table.list[0];
		for(var key in obj) {
			if (lodash.isObject(obj[key])){
				for (var innerKey in obj[key]){
				table.fields.push(key+'.'+innerKey);
				}
			}
			else
				table.fields.push(key);
		}
	}, 
	load_data(chose_load)
	{
		this.is_load=true;
		var table={fields:[],rows:0,meta:[],list:[]};
		if(chose_load==='user_data')
		{
			let str=this.user_fields.replace(/\s/g, '');
			table.fields=str.split(",");
			table.fields=lodash.remove(table.fields, function(el) {
				return el!=='';
			});
			table.rows=this.user_rows*1;
			if(table.rows<1)
			{
				this.load_data("little");
				return;
			}
			table.meta[0]=this.user_meta_head.replace(/\s/g, '');
			table.meta[1]=this.user_meta_even.replace(/\s/g, '');
			table.meta[2]=this.user_meta_odd.replace(/\s/g, '');
			for(let i=0;i<table.rows;i++)
			{
				var obj={}
				table.fields.forEach(field=>{
					lodash.set(obj, field,'');	
				});
				table.list.push(obj);
			}
			this.all_tables.push(table);
			this.is_load=false;
		}
		else if(chose_load==="json")
		{
			this.user_json=this.user_json+'';
			table.list=JSON.parse(this.user_json);
			this.get_fields(table);
			this.all_tables.push(table);
			this.is_load=false;
		}
		else
		{
			try{
				axios.get(this.url[chose_load]).then(response => {
				
					table.list=response.data;
					this.get_fields(table);
					this.all_tables.push(table);
					this.is_load=false;
				});
			}
			catch (err) {
				alert('CORS-ALLOW: *');
			}
		}
		this.show_modal_window=false;	
	},
	remove_table:function(index)
	{
		this.all_tables.splice(index, 1);
	}
  },
  components: {
	"my-table": Table
  },
  mounted() {
	this.load_data('little');	 
  },
};
</script>

<style>
.th_style
{
	background-color: #FFC0CB;
}
.even_style
{
	background-color:#66A1D2;
	color:white;
}
.odd_style
{
	background-color:#0B5FA5;
	color:white;
}
.none
{
	display:none;
}
td
{
  height: 20px;
  position: relative;
}
.new_td
{
	position: absolute;
	top:0px;
	bottom:0px;
	left:0px;
	right:0px;
	width: 100%;
    height: 100%;
    border-width: 0px;
    box-sizing: border-box;
}
.row
{
  margin-top:2em;
  width:100%;
  display:flex;
  align-items: center;
  justify-content: space-between;
}
.row.wrap
{
	flex-wrap:wrap;
}
.row.f_s
{
	justify-content: flex-start;
}
.tftable {
  border-width: 1px;
  border-color: black;
  border-collapse: collapse;
  width:100%;
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
  	padding: 15px;
	box-sizing: border-box;
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
  display: inline-block;
  margin: 20px 0px;
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
input[type='number']
{
	width: 30px;
}
input.search
{
	margin:20px 0px;
}
.loader
{
	height:24px;
	width:24px;
	background-position: 50% 50%;
	background-size: auto 100%;
	background-repeat: no-repeat;
	background-image:url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/Pgo8IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDIwMDEwOTA0Ly9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9UUi8yMDAxL1JFQy1TVkctMjAwMTA5MDQvRFREL3N2ZzEwLmR0ZCI+CjxzdmcgdmVyc2lvbj0iMS4wIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciCiB3aWR0aD0iNjAyLjAwMDAwMHB0IiBoZWlnaHQ9IjYwMi4wMDAwMDBwdCIgdmlld0JveD0iMCAwIDYwMi4wMDAwMDAgNjAyLjAwMDAwMCIKIHByZXNlcnZlQXNwZWN0UmF0aW89InhNaWRZTWlkIG1lZXQiPgo8Y2lyY2xlIGN4PSIzMDEiIGN5PSIzMDAiIHI9IjIyMiIgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjZmY5ZTAyIiBzdHJva2Utd2lkdGg9IjU2cHgiIHN0cm9rZS1kYXNoYXJyYXk9IjIzMiUiIHN0cm9rZS1kYXNob2Zmc2V0PSIyMzIlIiBzdHlsZT0idHJhbnNmb3JtLW9yaWdpbjogNTAlIDUwJTt0cmFuc2Zvcm06cm90YXRlKC05MGRlZykiPgogIDxhbmltYXRlIGlkPSJwMSIgYXR0cmlidXRlTmFtZT0ic3Ryb2tlLWRhc2hvZmZzZXQiIGJlZ2luPSIwcztwMS5lbmQiIHZhbHVlcz0iMjMyJTsgMCIgZHVyPSIzcyIgIGNhbGNNb2RlPSJwYWNlZCIvPgo8L2NpcmNsZT4KPC9zdmc+Cg==);
}
</style>
