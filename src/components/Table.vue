<template>
	<div>
		<input class='search' v-model='str_search' placeholder='Search' @keyup='find_something' @focus='focus_search'>	
		<table class="tftable" border="1">
			<tr>
				<th v-for="(field,i) in fields" :class="meta[0]" :key="i" @click="sort_table(field)">{{field}}</th>
			</tr>
			<tr v-for="(item,i) in clone_list" :key="i" :class="i%2===0?meta[2]:meta[1]" v-if="show_row(i)">
				<td v-for="(field,index) in fields"  :key="index" @dblclick="show_change_field_input">{{getObjectValueByPath(clone_list[i], field)}}
					<input class='none new_td'  @keyup="keyup_new_td(clone_list[i], field)" @blur="on_blur_input_new_td(clone_list[i], field)">
				</td>
			</tr>
		</table>
		<div class="row wrap">
			<div class='button' @click='show_modal_window=!show_modal_window'>Show JSON</div>
			<div v-if="!is_search">
				<input type="number" v-model="select_row_paste">
				<div class='button' @click='add_row(select_row_paste)'>Add row</div>
			</div>
			<div class='button' @click='reset_table()'>Reset</div>
			<div class='button' @click='remove_table()'>Remove</div>
			
			<div v-if="numbers_page>1" class='numbers_page'>
				<div class="number"  v-for="number in numbers_page" :key="number.id" @click="selected_page=number" :class="{active:number===selected_page}">{{number}}</div>
			</div>
		</div>
		<div class='wrap_modal_window' v-if="show_modal_window" @click="show_modal_window=!show_modal_window">
			<div class='modal_window' id='modal_window'>
				<div class='block_json'>{{clone_list}}</div>
				<div class='button' @click="copy_json">Copy</div>
			</div>
		</div>
	</div>
</template>

<script>
import axios from "axios";
import lodash from "lodash";
export default {
	props: {
		fields:{type:Array,default:[]},
		rows:{type:Number,default:0} ,
		meta: {type:Array,default:[]},
		list:{type:Array,default: () => []},
		index_table:{type:Number,default:0} 
	},
	data() {
		return {
			selected_page:1,
			is_ask:true,
			show_modal_window:false,
			select_row_paste:null,
			new_json:'',
			str_search:'',
			is_esc:false,
			clone_list:this.list,
			clone_for_search:[],
			name_sort_fields:{},
			is_search:false
		};
	},
	computed: {
		numbers_page:function()
		{
			return Math.floor(this.clone_list.length/10);
		},
		numbers_rows:function()
		{
			return this.clone_list.length;
		}
	},
	methods:{
		show_change_field_input(event)
		{
			var td=event.target;
			var input=td.children[0];
			if(input)
			{
				input.classList.remove('none');
				input.focus();
			}
		},
		keyup_new_td(object,path)
		{
			this.is_esc=false;
			var input=event.target;
			if(event.keyCode===13||event.keyCode===27)
			{
				if(event.keyCode===13)
					this.setObjectValueByPath(object,path,event.target.value);
				input.classList.add('none');
				if(event.keyCode===27)
				{
					this.is_esc=true;
				}
			}		
		},
		on_blur_input_new_td(object,path)
		{
			var input=event.target;
			if(!this.is_esc)
				this.setObjectValueByPath(object,path,event.target.value);
			event.target.classList.add('none');
		},
		focus_search(event)
		{
			if(event.target.value==='')
				this.clone_for_search=this.clone_list;
		},
		find_something(event)
		{
			if(event.keyCode===32)
				return;
			var ar=lodash.filter(this.clone_for_search, item=>{
				return lodash.some(this.fields,  el=>{
					return this.getObjectValueByPath(item, el).toString().indexOf(event.target.value)>=0
				});
			});
			if(event.target.value==='')
			{
				this.is_search=false;
				this.clone_list=this.clone_for_search;
			}
			else
			{
				this.is_search=true;
				this.clone_list=ar;
			}	
		},
		getObjectValueByPath(object, path)
		{
			return lodash.get(object, path);
		},
		setObjectValueByPath(object, path,value)
		{
			return lodash.set(object, path,value);
		},
		remove_table()
		{
			this.$emit('remove_table',this.index_table);
		},
		reset_table()
		{
			for(var key in this.clone_list) 
			{
				if (lodash.isObject(this.clone_list[key]))
				{
					for (var innerKey in this.clone_list[key])
					{
                        this.clone_list[key][innerKey]=''
                    }
                }
                else
                    this.clone_list[key]=''
            }
		},
		copy_json(element)
		{
			var parent=document.getElementById('modal_window');
			var text=JSON.stringify(this.clone_list);
			var textArea=document.createElement("textarea");
			textArea.value=text;
			parent.appendChild(textArea);
			textArea.focus();
			textArea.select();

			try {
				var successful=document.execCommand('copy');
				var msg=successful ? 'successful' : 'unsuccessful';
				console.log('Fallback: Copying text command was ' + msg);
			} catch (err) {
				console.error('Fallback: Oops, unable to copy', err);
			}
			parent.removeChild(textArea);
		},
		add_row(index)
		{
			if(!index||index===''||index<0||index>this.numbers_rows)
				index=this.numbers_rows*1;
			var obj={};
			this.fields.forEach(field=>{
				this.setObjectValueByPath(obj,field,'');
			});
			this.clone_list.splice(index, 0, obj);
		},
		show_row(i)
		{
			return i>=this.selected_page*10-10&&i<this.selected_page*10
		},
		sort_table(name)
		{
			if(!(name in this.name_sort_fields))
				this.name_sort_fields[name]='asc';
			else{
				if(this.name_sort_fields[name]==='asc')
					this.name_sort_fields[name]='desc';
				else
					this.name_sort_fields[name]='asc';
			}
			this.clone_list=lodash.orderBy(this.clone_list,[name],this.name_sort_fields[name]);
		},
		set_page(number)
		{
			this.selected_page=number;
		}
	}
 
};
</script>

<style>

</style>
