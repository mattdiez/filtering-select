<template>
	
	
  <div id="app">
    
    <link rel="stylesheet" href="https://unpkg.com/bootstrap@4.1.3/dist/css/bootstrap.min.css"/>

<form novalidate autocomplete="off">
    <div class="container">
		
      <div class="row">
        <div class="col-md-6">
          <div>Selected Value: [{{select1Value}}] - Selected Item: [{{selected1Item}}]</div>
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
                ref="select1"
                placeHolder="Please select a State"
                :required="true"
                :list="myList"
                :autoselect-single="true"
                :min-length="1"
                :has-error="$v.select1Value.$error"
                v-model="$v.select1Value.$model"
                @select="onSelect1Change"
                >
                <div class="invalid-feedback" v-if="!$v.select1Value.itemSelected">This field is required</div>
                
                <div slot="suggestion-item" slot-scope="{suggestion, highlightMatch}">                	
            		<span v-html="highlightMatch(suggestion.label)"/> - {{suggestion.key}}					            						       
            	</div>         
			</FilteringSelect>
          </div>
        </div>
        <div class="col-md-6">
            <div class="form-group">
            <label class="form-label">List Function Test</label>
            <FilteringSelect 
                name="crudbucks" 
                placeHolder="Please select a State"
                :list="getListFunction"
                :autoselect-single="true"
                :min-length="2"
                v-model="select2Value"
                />    		
            </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6">
          <div>Selected Value: [{{select3Value}}] - Selected Item: [{{selected3Item}}]</div>
        </div>

	    </div>      
        <div class="row">
        <div class="col-md-6">
          <div class="form-group">
            <label class="form-label">Simple Select Test</label>
            <FilteringSelect 
                ref="select3"
                placeHolder="Please select a State"
                :required="true"
                :list="myList"
                :autoselect-single="true"
                :simple-select="true"
                v-model="select3Value"
                @select="onSelect3Change"
                >                
                <div slot="suggestion-item" slot-scope="{suggestion, highlightMatch}">                	
            		<span v-html="highlightMatch(suggestion.label)"/> - {{suggestion.key}}					            						       
            	</div>         
			</FilteringSelect>
          </div>
        </div>
       </div>
    </div>
	<button type="submit">Click</button>
	
  	<button type="submit" @click.prevent="select3Value='KY'">Set KY</button>
  	<button type="submit" @click.prevent="select3Value=null">Null Out KY</button>
	</form>
	
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
    { key: "MO", label: "Missouri" },
    { key: "MT", label: "Montaha" },
    { key: "NJ", label: "New Jersey" },
    { key: "NM", label: "New Mexico" },
    { key: "NY", label: "New York" },
    { key: "NC", label: "North Carolina" },
    { key: "VA", label: "Virginia" },
    { key: "WA", label: "Washington" },
    { key: "WV", label: "West Virginia" },
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
        	selected1Item: null,
        	
			select2Value: null,
			select2Item: null,	
			listFunctionValue: null,
			select3Value: 'CA',
			selected3Item: null
  		}
  },
	validations: {
		select1Value: {			
			itemSelected: function(val) {
				return val != null;
			}
		}
	},  
  	methods: {
		getListFunction(text) {
      let count = 0;
			return new Promise(function(resolve, reject) {
        setTimeout(function(){
          // simulate a slooooow server
          resolve(RESULT_DATA);
        }, count);
			});
		},
		onSelect1Change(val) {
			this.selected1Item = this.$refs.select1.selectedItem;
		},
		onSelect3Change(val) {			
			this.selected3Item = this.$refs.select3.selectedItem;
		}
	},
	mounted() {
		this.select3Value = 'CA';
	}
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
