<template>
  <fieldset
    :disabled="disabled"
    class="flex flex-col items-start overflow-visible text-left"
  >
    <!-- Label -->
    <label
      v-if="label"
      :for="reference"
      class="text-sm text-gray-500"
      @click.prevent
    >
      {{ label }}
    </label>

    <div :ref="reference + '-input-container'" class="relative w-full">
      <div class="relative w-full mt-1 rounded-md shadow-sm">
        <!-- Magnifying glass -->
        <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
          <svg class="w-5 h-5 text-gray-400" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" /></svg>
        </div>
        <!-- Input -->
        <input
          :id="reference"
          :ref="reference + '-input'"
          v-model="input"
          :placeholder="placeholder"
          autocomplete="off"
          :class="disabled && 'bg-gray-100'"
          class="block w-full pl-10 form-input border-gray-200 sm:text-sm sm:leading-5 rounded-md shadow-sm"
          @input="onInput"
          @focus="onFocus"
          @blur="onBlur"
          @keydown.esc="onEscape"
          @keydown.up="onArrowUp"
          @keydown.down="onArrowDown"
          @keydown.enter="onEnter"
        >
        <div class="absolute inset-y-0 right-0 flex items-center pr-3 space-x-2" @mousedown.prevent="">
          <!-- Async loader -->
          <span v-show="isFetching" class="flex items-center justify-center w-5 h-5">
            <div class="w-full h-full border-2 border-gray-200 rounded-full border-t-gray-400 animate-spin" />
          </span>
        </div>
      </div>
      <!-- Dropdown -->
      <transition
        enter-active-class="transition duration-100 ease-out"
        enter-class="transform scale-y-95 opacity-0"
        enter-to-class="transform scale-y-100 opacity-100"
        leave-active-class="transition duration-75 ease-in"
        leave-class="transform scale-y-100 opacity-100"
        leave-to-class="transform scale-y-95 opacity-0"
        @before-enter="handlePopper"
        @enter="handlePopper"
        @after-enter="handlePopper"
      >
        <div
          v-show="displayDropdown"
          class="absolute z-20 w-full rounded-md"
          @mouseout="focusedOptionIndex = null"
        >
          <div
            :ref="reference + '-dropdown'"
            class="w-full py-1 overflow-auto bg-white border border-gray-200 rounded-md shadow-md max-h-48"
            role="menu"
            aria-orientation="vertical"
            aria-labelledby="options-menu"
          >
            <template v-if="filteredOptions.length > 0">
              <!-- Options -->
              <template v-if="$scopedSlots.options">
                <slot
                  name="options"
                  :options="filteredOptions"
                  :selected="selected"
                  :onMouseDown="option => onSelect(option)"
                  :onMouseOver="index => focusedOptionIndex = index"
                  :focusedOptionIndex="focusedOptionIndex"
                />
              </template>
              <template v-else>
                <div
                  v-for="(option, index) in filteredOptions"
                  :key="index"
                  :class="[
                    index === focusedOptionIndex ? 'bg-blue-100 text-gray-600' :
                    'odd:bg-gray-50 text-gray-600 bg-white'
                  ]"
                  class="flex items-center justify-between px-4 py-2 text-sm leading-5 cursor-pointer focus:outline-none"
                  role="menuitem"
                  @mousedown.prevent="onSelect(option)"
                  @mouseover="focusedOptionIndex = index"
                >
                  <span class="flex-1">{{ getOptionLabel(option) }}</span>

                  <div class="relative pl-3 pointer-events-none">
                    <div class="absolute inset-0 z-200" />
                    <input v-model="selected" :value="option" type="checkbox" class="w-4 h-4 text-blue-600 transition duration-150 ease-in-out form-checkbox">
                  </div>
                </div>
              </template>
            </template>
            <template v-else-if="hasSearched && filteredOptions.length === 0">
              <p class="block px-4 py-2 text-sm leading-5 text-gray-600">
                {{ noResultsMessage }}
              </p>
            </template>
            <template v-else>
              <p class="block px-4 py-2 text-sm leading-5 text-gray-600">
                {{ beforeSearchMessage }}
              </p>
            </template>
          </div>
        </div>
      </transition>
    </div>

    <!-- Description -->
    <p
      v-if="description"
      class="mt-1 text-sm text-gray-500"
    >
      {{ description }}
    </p>
  </fieldset>
</template>

<script>
import isEqual from 'lodash.isequal'
import { createPopper } from '@popperjs/core'
export default {
  name: 'SearchDropdownToggle',
  components: {},
  mixins: [],
  props: {
    label: { type: String, default: '' },
    placeholder: { type: String, default: 'Type to begin searching' },
    description: { type: String, default: '' },
    disabled: { type: Boolean, default: false },
    defaultOption: { type: [Object, String, Number], default: undefined },
    commonLabelPath: { type: String, default: '' },
    commonValuePath: { type: String, default: '' },
    optionLabelPath: { type: String, default: '' },
    optionValuePath: { type: String, default: '' },
    selectedLabelPath: { type: String, default: '' },
    selectedValuePath: { type: String, default: '' },
    getCustomLabel: { type: Function, default: undefined },
    selected: { type: Array, required: true },
    // @select - callback for when selecting an option, entire option is emitted
    // @clear - callback for when selected is cleared

    // Synchronous
    searchMultipleKeys: { type: Array, default: () => [] },
    options: { type: Array, default: () => [] },
    preloadOptions: { type: Function, default: undefined },
    preloadOptionsWhenPropChanges: { type: Boolean, default: false },
    preloadWatchedProp: { type: [Array, Object, Number, String, Boolean], default: undefined },
    preloadOnlyWhenTruthy: { type: Boolean, default: false },

    // Asynchronous
    getOptions: { type: Function, default: () => {} },
    isAsync: { type: Boolean, default: false },
    searchOnFocus: { type: Boolean, default: false },
    searchDebounceDelay: { type: Number, default: 500 },

    // State messages
    beforeSearchMessage: { type: String, default: 'Type to begin searching' },
    defaultHasNoMatchMessage: { type: String, default: 'No match found for current value' },
    noResultsMessage: { type: String, default: 'No results were found' },
    clearedOnDisableMessage: { type: String, default: '' }
    // selectedHasNoMatchMessage: { type: String, default: '' }
  },
  data () {
    return {
      filteredOptions: [],
      preloadedOptions: [],
      timeout: null,
      input: '',
      hasInitPopper: false,
      displayDropdown: false,
      hasFocus: false,
      isFetching: false,
      hasSearched: false,
      focusedOptionIndex: null,
      hasPreloaded: false,
      popperInstance: null
    }
  },
  computed: {
    reference () {
      return String(Date.now() + (Math.random() * 1000))
    }
  },
  watch: {
    defaultOption: {
      deep: true,
      handler () {
        this.handleDefaultOption()
      }
    },

    preloadWatchedProp (newValue) {
      if (this.preloadOnlyWhenTruthy && newValue && this.preloadOptionsWhenPropChanges) {
        this.handlePreloadOptions()
      } else if (!this.preloadOnlyWhenTruthy && this.preloadOptionsWhenPropChanges) {
        this.handlePreloadOptions()
      }
    }
  },

  // Lifecycle Hooks
  beforeCreate () {},
  created () {},
  beforeMount () {},
  mounted () {
    if (this.defaultOption) {
      this.handleDefaultOption()
    }

    if (this.preloadOptions) {
      this.handlePreloadOptions()
    }
  },
  beforeUpdate () {},
  updated () {},
  beforeDestroy () {},
  destroyed () {},
  methods: {
    isEqual,

    async handleAsyncSearch () {
      if (this.isAsync && !this.disabled) {
        this.isFetching = true
        const response = await this.getOptions(this.input)
        this.isFetching = false
        this.filteredOptions = response
        this.hasSearched = true
        this.displayDropdown = true
      }
    },

    handleSyncSearch () {
      if (!this.isAsync) {
        const options = this.preloadOptions ? this.preloadedOptions : this.options
        this.filteredOptions = options.filter((option) => {
          if (this.searchMultipleKeys.length > 1) {
            let isValid = false
            for (const index in this.searchMultipleKeys) {
              const comparator = this.getValueFromNested(option, this.searchMultipleKeys[index])
              const result = comparator.toLowerCase().includes(this.input.toLowerCase())
              if (result) {
                isValid = true
                break
              }
            }

            return isValid
          } else {
            let comparator = option
            if (this.commonLabelPath) {
              comparator = this.getValueFromNested(option, this.commonLabelPath)
            } else if (this.optionLabelPath) {
              comparator = this.getValueFromNested(option, this.optionLabelPath)
            }

            return comparator.toLowerCase().includes(this.input.toLowerCase())
          }
        })

        this.hasSearched = true
        this.displayDropdown = true
      }
    },

    onFocus () {
      this.hasFocus = true

      if (this.isAsync && this.searchOnFocus) {
        this.handleAsyncSearch()
      } else {
        this.handleSyncSearch()
      }
    },

    onBlur () {
      this.hasFocus = false
      this.displayDropdown = false
      this.focusedOptionIndex = null
    },

    onInput () {
      if (this.isAsync) {
        if (this.searchDebounceDelay > 0) {
          this.debounce(this.handleAsyncSearch)
        } else {
          this.handleAsyncSearch()
        }
      } else {
        this.handleSyncSearch()
      }
    },

    onSelect (option) {
      this.$emit('select', option)
    },

    onEscape () {
      this.forceBlur()
    },

    onArrowUp () {
      const length = this.filteredOptions.length
      if (length > 0) {
        if (this.focusedOptionIndex > 0) {
          this.focusedOptionIndex -= 1
        } else if (this.focusedOptionIndex === 0) {
          this.focusedOptionIndex = null
        } else if (this.focusedOptionIndex === null) {
          this.focusedOptionIndex = length - 1
        }

        this.handleArrowKeysScroll()
      }
    },

    onArrowDown () {
      const length = this.filteredOptions.length
      if (length > 0) {
        if (this.focusedOptionIndex !== null && this.focusedOptionIndex < length - 1) {
          this.focusedOptionIndex += 1
        } else if (this.focusedOptionIndex === null) {
          this.focusedOptionIndex = 0
        } else if (this.focusedOptionIndex === length - 1) {
          this.focusedOptionIndex = null
        }

        this.handleArrowKeysScroll()
      }
    },

    onEnter () {
      const option = this.filteredOptions[this.focusedOptionIndex]
      if (option) {
        this.onSelect(option)
      }
    },

    cancelTimeout () {
      clearTimeout(this.timeout)
      this.timeout = null
    },

    debounce (callback) {
      if (this.timeout) {
        this.cancelTimeout()
      }

      this.timeout = setTimeout(callback, this.searchDebounceDelay)
    },

    getValueFromNested (nested, path) {
      const pathArray = path.split('.')
      let currentValue = nested
      let currentPath = null

      for (let i = 0; i < pathArray.length; i++) {
        currentPath = pathArray[i]
        currentValue = currentValue[currentPath]
      }

      return currentValue
    },

    handleArrowKeysScroll () {
      if (this.focusedOptionIndex !== null) {
        const dropdown = this.$refs[this.reference + '-dropdown']
        const collection = dropdown.children
        const array = Array.prototype.map.call(collection, element => element)
        const activeOption = array[this.focusedOptionIndex]
        activeOption.scrollIntoView({ block: 'nearest' })
      }
    },

    handleDefaultOption () {
      this.onSelect(this.defaultOption)
    },

    async handlePreloadOptions () {
      if (!this.disabled) {
        this.hasPreloaded = false
        this.isFetching = true
        const response = await this.preloadOptions()
        this.isFetching = false
        this.preloadedOptions = response
        this.hasPreloaded = true
      }
    },

    // forceFocus () {
    //   const inputElement = this.$refs[this.reference + '-input']
    //   inputElement.focus()
    // },

    forceBlur () {
      document.activeElement.blur()
    },

    getOptionLabel (option) {
      if (this.getCustomLabel) {
        return this.getCustomLabel(option)
      } else if (this.commonLabelPath) {
        return this.getValueFromNested(option, this.commonLabelPath)
      } else if (this.optionLabelPath) {
        return this.getValueFromNested(option, this.optionLabelPath)
      } else {
        return option
      }
    },

    handlePopper () {
      if (!this.popperInstance) {
        const trigger = this.$refs[this.reference + '-input-container']
        const dropdown = this.$refs[this.reference + '-dropdown']

        this.popperInstance = createPopper(trigger, dropdown, {
          placement: 'bottom',
          modifiers: [
            {
              name: 'offset',
              options: {
                offset: [0, 5]
              }
            }
          ]
        })
      } else if (this.popperInstance) {
        this.popperInstance.update()
      }
    }
  }
}
</script>

<style scoped>
</style>
