<template>
  <div
    class="w-100 flex-grow flex_col--jc"
    :class="{'input-group' : bootstrapStyling}"
  >
    <!-- Calendar Button -->
    <span
      v-if="calendarButton"
      class="vdp-datepicker__calendar-button"
      :class="{'input-group-prepend' : bootstrapStyling}"
      :style="{'cursor:not-allowed;' : disabled}"
      @click="showCalendar"
    >
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <input
      :id="id"
      :ref="refName"
      :type="inline ? 'hidden' : 'text'"
      class="date-input input-field w-100"
      :class="computedInputClass"
      :name="name"
      :value="formattedValue"
      :open-date="openDate"
      :placeholder="placeholder"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      autocomplete="off"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"
    >
    <div
      class="input-border"
      :class="{
        'is-active': calendarOpen,
        'is-filled': formattedValue
      }"
    />
    <!-- Clear Button -->
    <span
      v-if="clearButton && selectedDate"
      class="vdp-datepicker__clear-button"
      :class="{'input-group-append' : bootstrapStyling}"
      @click="clearDate()"
    >
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="clearButtonIcon">
          <span v-if="!clearButtonIcon">&times;</span>
        </i>
      </span>
    </span>
    <slot name="afterDateInput" />
  </div>
</template>
<script>
/* eslint-disable vue/require-default-prop */
import { makeDateUtils } from '../utils/DateUtils'

export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: String,
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean,
    calendarOpen: Boolean
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      input: null,
      typedDate: false,
      utils: constructedDateUtils
    }
  },
  computed: {
    formattedValue () {
      if (!this.selectedDate) {
        return null
      }
      if (this.typedDate) {
        return this.typedDate
      }
      return typeof this.format === 'function'
        ? this.format(this.selectedDate)
        : this.utils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    },

    computedInputClass () {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === 'string') {
          return [this.inputClass, 'form-control'].join(' ')
        }
        return { 'form-control': true, ...this.inputClass }
      }
      return this.inputClass
    }
  },
  watch: {
    resetTypedDate () {
      this.typedDate = false
    }
  },
  mounted () {
    this.input = this.$el.querySelector('input')
  },
  methods: {
    showCalendar () {
      this.$emit('showCalendar')
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate (event) {
      // close calendar if escape or enter are pressed
      if ([
        27, // escape
        13 // enter
      ].includes(event.keyCode)) {
        this.input.blur()
      }

      if (this.typeable) {
        const typedDate = Date.parse(this.input.value)
        if (!isNaN(typedDate)) {
          this.typedDate = this.input.value
          this.$emit('typedDate', new Date(this.typedDate))
        }
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred () {
      if (this.typeable && isNaN(Date.parse(this.input.value))) {
        this.clearDate()
        this.input.value = null
        this.typedDate = null
      }

      this.$emit('closeCalendar')
    },
    /**
     * emit a clearDate event
     */
    clearDate () {
      this.$emit('clearDate')
    }
  }
}
// eslint-disable-next-line

</script>
