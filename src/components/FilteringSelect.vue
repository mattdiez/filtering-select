<template>
    <div>        
        		<!-- for debugging purposes -->		
		
        <div class="input-group-placeholder" v-bind:class="{ expanded : showSuggestions}">
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
           
			<div class="dropdown" v-bind:class="{ show : showSuggestions}"
					@mouseenter="hoverList(true)"
					@mouseleave="hoverList(false)">
                    <div class="dropdown-menu" v-bind:class="{ show : showSuggestions}" >
                    	<li v-if="loadingResponse">
                    		<a class="dropdown-item">Loading...</a>
                    	</li>

				
						<li v-if="suggestions.length > 0" v-for="(suggestion, index) in suggestions">
							<a class="dropdown-item" 
								@click="suggestionClick(suggestion)"
								v-html="highlightMatch(suggestion[labelAttr])"
								:class="[
									{
            							selected: selectedItem && (valueProperty(suggestion) == valueProperty(selectedItem)),
            							hover: hoveredItem && (valueProperty(hoveredItem) == valueProperty(suggestion))
            						}]"></a>
                        </li>
						<li v-else-if="!loading">
							<a class="dropdown-item">No results found</a>
						</li>
                    </div>
                </div>
        </div>        
    </div>
</template>

<script>

export default {
  	name: 'filtering-select',
	data() {
      return {
			disabled: false,
			textInput: null,
			selectedItem: null,
			hoveredItem: null,
			suggestions: [],
			renderSuggestions: false,
			isOverList: false,
			loadingResponse: false,
			canSend: true,
			typingForward: true
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
		},
		debounce: {
			type: Number,
			default: 350
		}
 	}, 
	computed: {
		showSuggestions: function() {
			return  this.suggestions.length > 0 || this.loadingResponse;
		},
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
	emitClickInput () {
		// TODO: implement this
	},
	keyDownListener (event) {
		this.moveSelection(event);
	},
	keyUpListener (event) {
		this.typingForward = true;

		if (event.code == 'Backspace') {
			this.clearSelection();
			this.typingForward = false;
		}
		
		if (event.code == 'Enter') {
			if (this.suggestions.length > 0) {
				this.select(this.suggestions[this.hoveredIndex != -1 ? this.hoveredIndex : 0]);
			}
		}
		else if ((event.code == 'ArrowDown') || (event.code == 'ArrowUp')) {
			// catch these and do nothing
			// this is ugly, refactor this
			this.typingForward = false;
		}
		else {
 			if (this.debounce) {
 				clearTimeout(this.timeoutInstance)
				this.timeoutInstance = setTimeout(this.research, this.debounce)
			} else {
				this.research()
			}
		}
	},
    async research () {
      try {
        if (this.canSend) {
          this.canSend = false
          this.$set(this, 'suggestions', await this.getSuggestions(this.textInput))
        }
      }
      catch (e) {
        this.clearSuggestions()
        throw e
      }
      finally {
        this.canSend = true

		if (this.suggestions.length === 0) {
        	this.hideList()
        } else {
        	this.showList()
		}
		// Autoselect first/only item
		if ((this.suggestions.length === 1) && (this.typingForward)) {
			this.select(this.suggestions[0])
		}
        return this.suggestions
      }
    },
	focusListener () {
		this.showList();
	},
	hoverList (isOverList) {
		this.isOverList = isOverList
	},      
	blurListener () {
		if (!this.isOverList) {
			if (!this.selectedItem) {
				this.textInput = null;
				this.clearSuggestions();
			}
		}
	}, 
	async getSuggestions (queryText) {
		let matches = [];
		let results = [];
		if (queryText) {
			// Note: This is doing a "startsWith" instead of an "indexOf"
			if (this.listIsRequest()) {
				// do ajax debouncing 
				// and promise/await
				this.loadingResponse = true;
				this.clearSuggestions();
				
				results =  await (this.list(queryText)) || []
				
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
    highlightMatch (value) {
        if (!value) return '-';

		let result = value;
		let query = this.textInput;

		if (!query) return result;
        
		const texts = query.split(/[\s-_/\\|\.]/gm).filter(t => !!t) || [''];
		return result.replace(new RegExp('(.*?)(' + texts.join('|') + ')(.*?)','gi'), '$1<b>$2</b>$3');
    },
	clearSelection () {
		this.selectedItem = null;
		this.$emit('select', null)
	},
	showList () {
		this.renderSuggestions = true;
	},
	hideList () {
		this.renderSuggestions = false;
	},
	clearSuggestions () {
		this.suggestions = [];
	},
	suggestionClick (item) {
		this.isOverList = false;
		this.select(item);
	},
	select (item) {
		this.selectedItem = item;
		this.textInput = item[this.labelAttr];		

		// reduce suggestion list to selected val
		this.suggestions = [item];

		this.hideList();

		this.$emit('select', item)
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
			e.preventDefault()
			this.showList()

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
	
	// disable research on arrow keys (!)
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