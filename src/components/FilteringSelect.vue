<template>
	<!-- TODO: paging of results (keys?) -->
    <div>        
        <!-- for debugging purposes -->
        		
		<div>[{{textInput}}] -- [{{selectedValue}}] - [[{{targetValue}}]</div>
        <div class="input-group-placeholder"  v-bind:class="{ expanded : renderSuggestions, 'is-invalid' : hasError}">        	
            <input 
                type="text" 
                name="none" 
                :placeholder="placeholderValue"          
                class="form-control placeholder" 
                tabindex="-1"/>
			
            <!-- //form-control text-input    -->
            <input 
            	:disabled="disabled" 
            	@click="emitClickInput" 
                @keydown="keyDownListener"                
                @focus="focusListener"
                @blur="blurListener"
                @input="inputListener"
                @compositionend="evt => inCompositionMode = false"
                @compositionstart="evt => inCompositionMode = true"
                type="text"
                :id="id" 
                :name="name" 
                placeholder="" 
                :required="required"
                :tabindex="tabindex"
				ref="inputBox"
                class="form-control text-input"                                
                v-bind:class="{'is-invalid' : hasError}"
                />
		</div>                    
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
								:class="[
									{
            							selected: selectedItem && (valueProperty(suggestion) == valueProperty(selectedItem)),
            							hover: hoveredItem && (valueProperty(hoveredItem) == valueProperty(suggestion))
            						}]">
            					<slot
            						:suggestion="suggestion"
            						:highlightMatch="highlightMatch" 
            						name="suggestion-item">
            						<span v-html="highlightMatch(suggestion[labelAttr])"></span>
            					</slot>		
            				</a>
							      						            				
                        </li>
                        
						<li v-if="(suggestions.length == 0) && !loadingResponse">
							<a class="dropdown-item">No results found</a>
						</li>
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
			typingForward: true,
			selectedValue: this.value, 
			selectedItem: null,
			inCompositionMode: null ,/* event for chrome on mobile */
			targetValue: null	
		}
  	}, 
	props: { // these are passed in
		id: {
			type: String,
			default: 'filtering-select-id'
		},	
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
		placeholder: {
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
		hasError: {
			type: Boolean,
			default: null
		},
		required: {
			type: Boolean,
			default: false
		},
		simpleSelect: {
			type: Boolean,
			default: false
		},
		tabindex: {
			type: Number,
			default: 0
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
	            return this.placeholder;
	        }
	    },
		hoveredIndex () {
			return this.suggestions.findIndex(el => this.hoveredItem && (this.valueProperty(this.hoveredItem) == this.valueProperty(el)))
		},    
	},
	watch: {
		value: {
			// watch changes in the v-model property	
			/*		
			if ((!this.listIsRequest()) && (this.value)) {
				// TODO: This is called in created() AND here, maybe merge
				let item = this.list.find(item => this.valueProperty(item) === this.value)
				if (item)
  					this.select(item)  				
  			} else {
  				this.selectedValue = this.value;
			  }*/
			handler(current) {
				this.selectedValue = current;
			},
			immediate: true
		},
		selectedItem(newItem) {
			// watch changes on suggestion select/click
			this.selectedValue = newItem ? newItem[this.valueAttr] : null;
		},
		selectedValue(newValue) {
			this.$emit('input', this.selectedValue)
		},
	},
	methods: {
		emitClickInput () {
			// TODO: implement this
		},		
		keyDownListener (event) {
			this.typingForward = true;
																	
			if (['Delete', 'Backspace'].indexOf(event.code) != -1) {
				this.typingForward = false;
			}
		
			if (['NumpadEnter', 'Enter'].indexOf(event.code) != -1) {			
				if ((this.renderSuggestions) && (this.suggestions.length > 0) ){								
					event.preventDefault();
					this.select(this.suggestions[this.hoveredIndex != -1 ? this.hoveredIndex : 0], true);				
				}
			}
		
			this.moveSelection(event);
		},
		inputListener(event) {			
			const targetValue = !event.target ? event : event.target.value
			
			this.targetValue = targetValue;
			
			if (this.inCompositionMode) {
				// in composition mode under google chrome on mobile
				// https://github.com/vuejs/vue/issues/8231	
				this.typingForward = true;
				if (targetValue && this.textInput && (targetValue.length < this.textInput.length)) {
					this.typingForward = false;
				}
			}

			if (this.textInput === targetValue) { return }
		
			this.textInput = targetValue
			  
			if (this.selectedItem) {		
				this.clearSelection();
			}
			if (this.listIsRequest() && this.debounce) {
				clearTimeout(this.timeoutInstance)
				this.timeoutInstance = setTimeout(this.research, this.debounce)
			} else {
				this.research()
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
				this.showList()
		
				// Autoselect first/only item				
				if ((this.suggestions.length === 1) && (this.typingForward) && this.autoselectSingle) {									
					this.select(this.suggestions[0], true)
				}
	        	return this.suggestions
      		}
    	},
		async focusListener (event) {
			// autoselect input
			event.target.select();
			if (this.suggestions.length === 0) {
				await this.research()
			}
			this.showList()
		},
		hoverList (isOverList) {
			this.isOverList = isOverList
		},      
		blurListener () {
			if (!this.isOverList) {
				if (!this.selectedItem) {
					this.textInput = null;
					this.$refs.inputBox.value = null;
					this.clearSuggestions();
					this.clearSelection();
				}							
				this.hideList();
			}			
		}, 
		async getSuggestions () {
			let matches = [];
			let results = [];			
			let queryText = this.textInput;

			if (queryText) {
				if ((queryText) && (queryText.length < this.minLength)) {
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
	
				matches = results.filter(e => 
					e[this.labelAttr].toLowerCase().indexOf(queryText.toLowerCase()) != -1 
				);			
			} else {
				// clear out selected item
				this.clearSelection();
				
				if (!this.listIsRequest()) {
					// not a request? just a list?
					matches = this.list.filter(e => true);
				}
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
				if (
					(this.textInput && (this.textInput.length >= this.minLength))
					|| (this.simpleSelect)) {
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
			this.select(item, true);
		},
		select (item, interactive = false) {
			this.selectedItem = item;
			
			// reduce suggestion list to selected value
			this.suggestions = [item];
			
			this.hideList();
			
			// TODO: Move this to a watch/listener event?
			let value = item[this.valueAttr];
			let label = item[this.labelAttr];

			this.$emit('input', value)
			this.$refs.inputBox.value = label;
			this.textInput = label;
			

			if (interactive) {			
				this.$emit('select', this.textInput)
			}
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
			return obj[this.valueAttr];
	    }
		
	},
	mounted() {
		if ((!this.listIsRequest()) && (this.value)) {
			let item = this.list.find(item => this.valueProperty(item) === this.value);
			if (item)
				this.select(item)
		}
	},
}
</script>

<style>

.input-group-placeholder {
    width: 100%;
    height: 100%;
    position: relative;
}

.input-group-placeholder.is-invalid ~ .invalid-feedback {
  display: block;
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
  /*  background-color: #2832D5; */
  color: #fff;
}
</style>