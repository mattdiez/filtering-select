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
                                v-html="boldenSuggestion(item[labelAttr])"></a>
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
        if ((this.suggestions.length > 0) && (this.textInput)) {
            return this.letterProcess(this.suggestions[0]);
        } else if (this.textInput) {
            return '';
        } else {
            return this.placeHolder;
        }
    }
  },
  methods: {
      highlightMatch: function() {
          // TODO: Implement this
      },
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
    boldenSuggestion(value) {
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
          this.textInput = item[this.labelAttr];
          this.selectedItem = item;
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
      // match starts at zero for letter process?
      // cleanup placeholder stuff, override placeholder
      // implement debounce
      // implement query (pass as arg?)
      // implement key navigation of results
      // busy/loading
      // implement keyup
      // implement carat up/down
      // labelAttr override
      // keyattr override
      // validators !!!
      // test for submit?
      // implement max results
      // implement highlighting of results 
      // click off behavior (@blur or whatevr)
      // enter key behavior (autoselect?)
      // paging of results (keys?)

      // DONE
      // implement mouse select
      // implement hover      
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