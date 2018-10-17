<template>
  
  <div id="app">
    <link rel="stylesheet" href="https://unpkg.com/bootstrap@4.1.3/dist/css/bootstrap.min.css"/>
    <img alt="Vue logo" src="./assets/logo.png">

    <div class="container">

      <div class="row">
        <div class="col-md-6">
          <div>Selected Item: [{{selectValue}}]</div>
        </div>
        <div class="col-md-6">
          <div>Selected Item: [{{listFunctionValue}}]</div>
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
                v-model="selectValue"
                @select="selectHandler"
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
                v-model="selectValue"
                @select="listSelectHandler"
                />    		
            </div>
        </div>
      </div>
    </div>

	
  </div>
</template>

<script>
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
  			selectValue: null ,	
        listFunctionValue: null
  		}
	},
	methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async doSleep() {
      console.log("About to sleep")
      await sleep(2000)
      console.log("Slept, now resolve")
    },
		getListFunction(text) {
			return new Promise(function(resolve, reject) {
          // maybe do some async stuff in here
        console.log("Text is " + text )
        // simulate a slooooow server
        this.doSleep()
        
  			resolve(RESULT_DATA);
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
