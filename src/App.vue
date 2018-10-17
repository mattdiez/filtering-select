<template>
  
  <div id="app">
    <link rel="stylesheet" href="https://unpkg.com/bootstrap@4.1.3/dist/css/bootstrap.min.css"/>
	<pre>{{$v}}</pre>

    <div class="container">

      <div class="row">
        <div class="col-md-6">
          <div>Selected Item: [{{select1Value}}]</div>
        </div>
        <div class="col-md-6">
          <div>Selected Item: [{{select2Value}}]</div>
        </div>
	    </div>

      <div class="row">
        <div class="col-md-6">
          <div class="form-group">
            <label class="form-label">Static Data Test</label>
            <FilteringSelect 
                name="zip" 
                placeHolder="Please select a ZIP"
                :list="myList"
                :autoselect-single="true"
                :min-length="1"
                :classExpression="$v.select1Value.$error ? 'is-invalid' : null"
                :stateExpression="$v.select1Value.$error ? 'invalid' : null"
                v-model="$v.select1Value.$model"
                />    		
          </div>
        </div>
        <div class="col-md-6">
            <div class="form-group">
            <label class="form-label">List Function Test</label>
            <FilteringSelect 
                name="zip" 
                placeHolder="Please select a ZIP"
                :list="getListFunction"
                :autoselect-single="true"
                :min-length="2"
                v-model="select2Value"
                @select="listSelectHandler"
                />    		
            </div>
        </div>
      </div>
    </div>

	
  </div>
</template>

<script>



import Vue from 'vue'
import Vuelidate from 'vuelidate'
Vue.use(Vuelidate)

const { required, minLength, helpers } = require('vuelidate/lib/validators')
import HelloWorld from './components/HelloWorld.vue'
import FilteringSelect from './components/FilteringSelect.vue'

let RESULT_DATA = [
    { key: "AL", label: "Alabama" },
    { key: "AK", label: "Alaska" },
    { key: "CA", label: "California" },
    { key: "CO", label: "Colorado" },
    { key: "CT", label: "Connecticut" },
    { key: "DE", label: "Delaware" },
    { key: "FL", label: "Florida" },
    { key: "GA", label: "Georgia" },
    { key: "HI", label: "Hawaii" },
    { key: "IL", label: "Illinois" },
    { key: "IN", label: "Indiana" },
    { key: "KY", label: "Kentucky" },
    { key: "LA", label: "Louisiana" },
    { key: "ME", label: "Maine" },
    { key: "MA", label: "Massachusetts" },
    { key: "MI", label: "Michigan" },
    { key: "MT", label: "Montaha" },
    { key: "NJ", label: "New Jersey" },
    { key: "NM", label: "New Mexico" },
    { key: "NY", label: "New York" },
    { key: "NC", label: "North Carolina" },
];

export default {
  name: 'app',
  components: {
    HelloWorld,
    FilteringSelect
  },
	data() {
  		return {
  			myList: RESULT_DATA,
        select1Value: null ,
        select2Value: null,	
        listFunctionValue: null
  		}
  },
	validations: {
		select1Value: {
			/*
			itemSelected: function(val) {
				return val != null;
			}
			*/
		}
	},  
  	methods: {
		getListFunction(text) {
			return new Promise(function(resolve, reject) {
        setTimeout(function(){
          // simulate a slooooow server
          resolve(RESULT_DATA);
        }, 2000);
			});
		},
		selectHandler(item) {
			this.selectValue = item;
    },
    listSelectHandler(item) {
			this.listFunctionValue = item;
		}
	}
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
