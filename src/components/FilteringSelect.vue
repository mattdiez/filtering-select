<template>
    <div>        
        <!-- for debugging purposes -->		
        <div class="input-group-placeholder" v-bind:class="{ expanded : renderSuggestions}">        	
            <input 
                type="search" 
                name="search" 
                :placeholder="placeholderValue" 
                autocomplete="off" 
                required="required" 
                class="form-control placeholder" 
                tabindex="-1"/>
			
            <!-- //form-control text-input    -->
            <input :disabled="disabled" @click="emitClickInput"
            	@keydown="keyDownListener" 
                @keyup="keyUpListener"                
                @focus="focusListener"
                @blur="blurListener"
                v-model="textInput"
                type="search" 
                :name="name" 
                placeholder="" 
                autocomplete="off" 
                required="required"
                class="form-control text-input"                                
                :class="classExpression"
  				:state="stateExpression" 
                />
        	<!-- slot for feedback -->
			<slot></slot>
			<!-- slot for feedback -->                                  						
			<div class="dropdown" v-bind:class="{ show : renderSuggestions}"
					@mouseenter="hoverList(true)"
					@mouseleave="hoverList(false)">
                    <div class="dropdown-menu" v-bind:class="{ show : renderSuggestions}" >
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
						<li v-if="(suggestions.length == 0) && !loadingResponse">
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
		name: {
			type: String,
			default: 'filtering-select-name'
		},
		value: {
			type: [Object, String],
			default: () => {}
		},
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
		minLength: {
			type: Number,
			default: 1
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
		},
		autoselectSingle: {
			type: Boolean,
			default: false
		},
		classExpression: {
			type: String,
			default: null
		},
		stateExpression: {
			type: String,
			default: null
		}
 	}, 
	computed: {
		selectedItem: {
			get() {
				return this.value;
			},
			set(newValue) {
				this.$emit('input', newValue)
				this.$emit('select', newValue)
			}
		},
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
			this.clearSelection();
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
          this.$set(this, 'suggestions', await this.getSuggestions())
        }
      }
      catch (e) {
        this.clearSuggestions()
        throw e
      }
      finally {
        this.canSend = true
				
		this.showList();
		
		// Autoselect first/only item
		if ((this.suggestions.length === 1) && (this.typingForward) && this.autoselectSingle) {
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
				
			
			this.hideList();
		}
	}, 
	async getSuggestions () {
		let matches = [];
		let results = [];
		let queryText = this.textInput;

		if (queryText) {
			if (queryText.length < this.minLength) {
          		this.hideList()
				return []
			}

			this.clearSuggestions();

			if (this.listIsRequest()) {
				this.loadingResponse = true;
				this.showList(); 
				
				results =  await (this.list(queryText)) || []
				
				this.loadingResponse = false;				
			} else {
				results = this.list;
			}	

			/*	
			Note: This is doing a "startsWith" instead of an "indexOf"	
			matches = results.filter(e => 
				e[this.labelAttr].toLowerCase().startsWith(queryText.toLowerCase()) 
			);
			*/
			matches = results.filter(e => 
				e[this.labelAttr].toLowerCase().indexOf(queryText.toLowerCase()) != -1 
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
		if (!this.renderSuggestions) {
			if (this.textInput && (this.textInput.length >= this.minLength)) {
				this.renderSuggestions = true
				this.$emit('show-list')
        	}
      	}
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
	},
	letterProcess (item) {
		if (item[this.labelAttr].toLowerCase().startsWith(this.textInput.toLowerCase())) {
			var remoteText = item[this.labelAttr].split('')
			var inputText = this.textInput.split('')		
			inputText.forEach(function (letter, key) {
				if (letter !== remoteText[key]) {
					remoteText[key] = letter
				}
			})
			return remoteText.join('')
		}
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
	

	// model-back-and-forth
	// test with vuelidate validators !!!
	// test for form submit?
	// template slot (!)
	// paging of results (keys?)
	// DONE
	// disable research on arrow keys (!)
	// implement debounce
	// implement async query (pass as arg?)
	// busy/loading message
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
	top: 16px;
	/* margin: auto; */
	/* bottom: 0; */
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
  /*  background-color: #2832D5; */
  color: #fff;
}
</style>