<template lang="pug">
.v-autocomplete
  .v-autocomplete-input-group(:class="{'v-autocomplete-selected': value}")
    input.v-autocomplete-input(type='search'
      v-model='searchText'
      :placeholder='placeholder'
      @blur='blur'
      @focus='focus'
      @input='inputChange'
    )
  .v-autocomplete-list(v-if='showList && internalItems')
    .v-autocomplete-list-item(v-for='item in internalItems', @click='onSelectItem(item)')
      div(:is='componentItem', :item='item')
</template>

<script>
import Item from './Item.vue'
import utils from './utils.js'

export default {
  name: 'v-autocomplete',
  props: {
    componentItem: { default: () => Item },
    placeholder: String,
    minLen: { type: Number, default: utils.minLen },
    wait: { type: Number, default: utils.wait },
    value: null,
    getLabel: {
      type: Function,
      default: item => item
    },
    items: Array,
  },
  data () {
    return {
      searchText: '',
      showList: false,
      internalItems: this.items || []
    }
  },
  methods: {
    inputChange () {
      this.showList = true
      this.onSelectItem(null)
      utils.callUpdateItems(this.searchText, this.updateItems)
      let item = utils.findItem(this.internalItems, this.searchText, this.getLabel)
      if (item) {
        this.onSelectItem(item)
      }
      this.$emit('change', this.searchText)
    },

    updateItems () {
      this.$emit('updateItems', this.searchText)
    },

    focus () {
      this.showList = true
    },

    blur () {
      setTimeout( () => this.showList = false, 200)
    },

    onSelectItem (item) {
      if (item) {
        this.internalItems = [item]
        this.searchText = this.getLabel(item)
      } else {
        this.setItems(this.items)
      }
      this.$emit('input', item)
    },

    setItems (items) {
      this.internalItems = items || []
    }

  },
  created () {
    utils.minLen = this.minLen
    utils.wait = this.wait
    this.onSelectItem(this.value)
  },
  watch: {
    items (newValue) {
      this.setItems(newValue)
    },
    value (newValue) {
      this.onSelectItem(newValue)
    }
  }
}
</script>

<style lang="stylus">
.v-autocomplete
  position relative
  .v-autocomplete-list
    position absolute
    .v-autocomplete-list-item
      cursor pointer
</style>