<template>
    <div>
        <div>[{{selectedItem}}]</div>
        <div class="input-group-placeholder">
            <input 
                type="search" 
                name="search" 
                :placeholder="placeholderValue" 
                autocomplete="off" 
                required="required" 
                class="form-control placeholder" 
                tabindex="-1"/>
            <input :disabled="disabled" @click="emitClickInput" 
                @keyup="keyUpListener" 
                @focus="focusListener"
                v-model="textInput" 
                type="search" :name="name" placeholder="" 
                autocomplete="off" required="required" 
                class="form-control text-input" 
                :autofocus="autofocus"/>
           
                <div class="dropdown" v-bind:class="{ show : renderSuggestions}">
                    <div class="dropdown-menu" v-bind:class="{ show : renderSuggestions}" >
                        <li v-if="suggestions.length === 0">
                            <a class="dropdown-item">No results found</a>
                        </li>
                        <li v-for="item in suggestions">
                            <a class="dropdown-item" @click="select(item)"
                                v-html="highlightMatch(item[labelAttr])"></a>
                        </li>
                    </div>
                </div>
        </div>        
    </div>
</template>

<script>
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
    { key: "KY", label: "Kentucky" },
];

export default {
  name: 'HelloWorld',
  data() {
      return {
        inputChanged: false,
        interactive: false,
        autofocus: true,
        disabled: false,
        textInput: null,
        selectedItem: null,
        suggestions: [],
        renderSuggestions: false,
      }
  }, 
  props: { // pass these in?
    name: String,
    idAttr: {
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
            return this.letterProcess(this.suggestions[0]);
        } else if (this.textInput) {
            // no suggestions shown, but something has been typed
            return '';
        } else {
            return this.placeHolder;
        }
    }
  },
  methods: {
      emitClickInput: function() {
          // TODO: implement this
      },
      keyUpListener: function(event) {

        this.interactive = true;
        this.inputChanged = true;

        if (event.code == 'Enter') {
            if (this.suggestions.length > 0) {
                // perhaps catch 'highlighted index' here instead
                this.select(this.suggestions[0]);
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
          if (this.suggestions.length > 0) {
              this.showSuggestions();
          }
      },
      /*
      blurListener: function() {
          this.hideSuggestions();
          this.interactive = false;
          if (!this.selectedItem) {
              this.textInput = null;
          }
      }, */
      getSuggestions: function(queryText) {
        let matches = [];
        if (queryText) {
            // Note: This is doing a "startsWith" instead of an "indexOf"
            matches = RESULT_DATA.filter(e => 
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
      select: function(item) {
          this.selectedItem = item;

          this.textInput = item[this.labelAttr];
          // note: may want to recalculate this elsewhere
          this.placeHolder = item[this.labelAttr];
          
          this.hideSuggestions();
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
      // implement debounce
      // implement async query (pass as arg?)
      // implement key navigation of results
      // busy/loading message
      // implement carat up/down
      // test with vuelidate validators !!!
      // test for form submit?
      // click off behavior (@blur or whatevr)
      // paging of results (keys?)

      // DONE
      // implement mouse select
      // implement hover   
      // enter key behavior (autoselect?)   
      // labelAttr override
      // keyattr override
      // implement max results
      // implement highlighting of results 
      // cleanup placeholder stuff, override placeholder
      // match starts at zero for letter process?
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

</style>