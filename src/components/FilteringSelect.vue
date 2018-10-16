<template>
    <div>        
        		<!-- for debugging purposes -->
			<div>Selected Item: [{{selectedItem}}]</div>
        <div class="input-group-placeholder" v-bind:class="{ expanded : renderSuggestions}">
            <input 
                type="search" 
                name="search" 
                :placeholder="placeholderValue" 
                autocomplete="off" 
                required="required" 
                class="form-control placeholder" 
                tabindex="-1"/>
            <input :disabled="disabled" @click="emitClickInput"
            	@keydown="keyDownListener" 
                @keyup="keyUpListener"                
                @focus="focusListener"
                @blur="blurListener"
                v-model="textInput"
                type="search" :name="name" placeholder="" 
                autocomplete="off" required="required" 
                class="form-control text-input" 
                />
           
			<div class="dropdown" v-bind:class="{ show : renderSuggestions}"
					@mouseenter="hoverList(true)"
					@mouseleave="hoverList(false)">
                    <div class="dropdown-menu" v-bind:class="{ show : renderSuggestions}" >
                    	<li v-if="loadingResponse">
                    		<a class="dropdown-item">Loading...</a>
                    	</li>
						<li v-if="!loadingResponse && suggestions.length === 0">
							<a class="dropdown-item">No results found</a>
						</li>
						<li v-for="suggestion in suggestions">
							<a class="dropdown-item" @click="suggestionClick(suggestion)"
								v-html="highlightMatch(suggestion[labelAttr])"
								:class="[
									{
            							selected: selectedItem && (valueProperty(suggestion) == valueProperty(selectedItem)),
            							hover: hoveredItem && (valueProperty(hoveredItem) == valueProperty(suggestion))
            						}]"></a>
                        </li>
                    </div>
                </div>
        </div>        
    </div>
</template>

<script>

export default {
  	name: 'filtering-select',
	model: {
		prop: 'value',
    	get event() { return event }
	},  
	data() {
      return {
			inputChanged: false,
			disabled: false,
			textInput: null,
			selectedItem: null,
			hoveredItem: null,
			suggestions: [],
			renderSuggestions: false,
			isOverList: false,
			loadingResponse: false
		}
  	}, 
	props: { // these are passed in
		name: String,
		valueAttr: {
		    type: String,
		    default: 'key'
		},
		labelAttr: {
		    type: String,
		    default: 'label'
		},
		placeHolder: {
		    type: String,
		    default: 'Please input a value'
		},
		maxResults: {
		    type: Number,
		    default: 10
		},
		list: {
			type: [Function, Array],
			default: () => []
		}
 	}, 
	computed: {
	    placeholderValue: function() {
	        if (this.selectedItem) {
	            // hide the placeholder
	            return ''; 
	        }
	        else if ((this.suggestions.length > 0) && (this.textInput)) {
	            // match up with first in input box
	            return this.letterProcess(this.suggestions[this.hoveredIndex != -1 ? this.hoveredIndex : 0]);
	        } else if (this.textInput) {
	            // no suggestions shown, but something has been typed
	            return '';
	        } else {
	            return this.placeHolder;
	        }
	    },
		hoveredIndex () {
			return this.suggestions.findIndex(el => this.hoveredItem && (this.valueProperty(this.hoveredItem) == this.valueProperty(el)))
		},    
	},
  methods: {
	emitClickInput: function() {
		// TODO: implement this
	},
	keyDownListener: function(event) {
		this.moveSelection(event);
	},
	keyUpListener: function(event) {
		this.inputChanged = true;

		if (event.code == 'Backspace') {
			this.clearSelection();
		}
		
		if (event.code == 'Enter') {
			if (this.suggestions.length > 0) {
				// perhaps catch 'highlighted index' here instead
				this.select(this.suggestions[this.hoveredIndex != -1 ? this.hoveredIndex : 0]);
			}
		}
		else {
			// TODO: get all with a method. page through these?
			this.suggestions = this.getSuggestions(this.textInput);
						
			// Think through this 
			this.renderSuggestions = (this.suggestions.length > 0);
		}
	},
	focusListener: function() {
		this.showSuggestions();
	},
	hoverList (isOverList) {
		this.isOverList = isOverList
	},      
	blurListener: function() {
		if (!this.isOverList) {
			this.hideSuggestions();
			if (!this.selectedItem) {
				this.textInput = null;
			}
		}
	}, 
	getSuggestions: function(queryText) {
		let matches = [];
		let results = [];
		if (queryText) {
			// Note: This is doing a "startsWith" instead of an "indexOf"
			if (this.listIsRequest()) {
				// do ajax debouncing 
				// and promise/await
				this.loadingResponse = true;
				results =  this.list(queryText) || []
				
				this.loadingResponse = false;				
			} else {
				results = this.list;
			}			
			matches = results.filter(e => 
				e[this.labelAttr].toLowerCase().startsWith(queryText.toLowerCase()) 
			);
		} else {
			// clear out selected item
			this.clearSelection();
		}
		// consider paging at some time
		return matches.splice(0, this.maxResults);
    },
    highlightMatch: function(value) {
        if (!value) return '-';

		let result = value;
		let query = this.textInput;

		if (!query) return result;
        
		const texts = query.split(/[\s-_/\\|\.]/gm).filter(t => !!t) || [''];
		return result.replace(new RegExp('(.*?)(' + texts.join('|') + ')(.*?)','gi'), '$1<b>$2</b>$3');
    },
	clearSelection: function() {
		this.selectedItem = null;
	},
	showSuggestions: function() {
		this.renderSuggestions = true;
	},
	hideSuggestions: function() {
		this.renderSuggestions = false;
	},
	suggestionClick: function(item) {
		this.isOverList = false;
		this.select(item);
	},
	select: function(item) {
		this.selectedItem = item;
		this.textInput = item[this.labelAttr];		         
		this.hideSuggestions();
		// update suggestions behind the scenes to match
		this.suggestions = this.getSuggestions(this.textInput);
	},
	letterProcess (item) {
		var remoteText = item[this.labelAttr].split('')
		var inputText = this.textInput.split('')
		inputText.forEach(function (letter, key) {
			if (letter !== remoteText[key]) {
				remoteText[key] = letter
			}
		})
		return remoteText.join('')
	},
	moveSelection (e) {
		
		if ((e.code == 'ArrowDown') || (e.code == 'ArrowUp')) {
			debugger;;
        	e.preventDefault()
			this.showSuggestions()
			
			const isMovingDown = e.code == 'ArrowDown';
			const direction = isMovingDown * 2 - 1
			const listEdge = isMovingDown ? 0 : this.suggestions.length - 1
			const hoversBetweenEdges = isMovingDown ? this.hoveredIndex < this.suggestions.length - 1 : this.hoveredIndex > 0
			
			let item = null
			
			if (!this.hoveredItem) {
			  item = this.selectedItem || this.suggestions[listEdge]
			} else if (hoversBetweenEdges) {
			  item = this.suggestions[this.hoveredIndex + direction]
			} else /* if hovers on edge */ {
			  item = this.suggestions[listEdge]
			}
			
			this.hover(item)
      	}
	},
	hover (item, elem) {
		this.hoveredItem = item
		if (this.hovered != null) {
			this.$emit('hover', item, elem)
		}
    },
	listIsRequest () {
		return typeof this.list === 'function'
	},    
    valueProperty (obj) {
		//return this.getPropertyByAttribute(obj, this.valueAttribute)
		return obj[this.valueAttr];
    },
      
	// implement debounce
	// implement async query (pass as arg?)
	// busy/loading message
	// model-back-and-forth
	// test with vuelidate validators !!!
	// test for form submit?
	// paging of results (keys?)
	// template slot (!)
	      	
	// DONE
	// click off behavior (@blur or whatevr)      
	// implement carat up/down      
	// implement mouse select
	// implement hover   
	// enter key behavior (autoselect?)   
	// labelAttr override
	// keyattr override
	// implement max results
	// implement highlighting of results 
	// cleanup placeholder stuff, override placeholder
	// match starts at zero for letter process?
	// implement key navigation of results	
  }
}
</script>

<style>

.input-group-placeholder {
    width: 100%;
    height: 100%;
    position: relative;
}

.input-group-placeholder .form-control {
    width: 100%;
}

.input-group-placeholder .form-control.placeholder {
    top: 0;
    left: 0;
    position: absolute;
}

.input-group-placeholder .form-control.text-input {
    z-index: 10;
    background-color: rgba(255,255,255,0);
    position: relative;
}

.input-group-placeholder:after {
	content: "";
	position: absolute;
	width: 0;
	height: 0;
	margin: auto;
	top: 0;
	bottom: 0;
	right: 0.75em;
	border-top: 5px solid #000;
	border-right: 5px solid transparent;
	border-left: 5px solid transparent;
	pointer-events: none;
}

.input-group-placeholder.expanded:after { 
	border-bottom: 5px solid #000; 
	border-top: 0; 
}

.dropdown-menu {
    width: 100%;
}

.dropdown-item.hover {
  background-color: #2874D5 !important;
  color: #fff !important;
}

.dropdown-item.selected {
  background-color: #2832D5;
  color: #fff;
}
</style>