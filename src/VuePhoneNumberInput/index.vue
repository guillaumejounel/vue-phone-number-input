<template>
  <div class="vue-phone-number-input flex">
    <div class="select-country-container">
      <SelectCountry
        v-model="codeCountry"
        :items="codesCountries"
        label="Country Code"
      />
    </div>
    <div class="flex-1">
      <VueInputUI
        v-model="phoneNumber"
        label="Phone number"
        :error="!numberIsValid"
        :hint="codeCountry ? phoneNumberHint : 'Choose country'"
        class="input-text-phone"
      />
    </div>
  </div>
</template>
<script>
  /* eslint-disable */
  import CodesCountries from './assets/js/phoneCodeCountries.js'
  import { parseNumber, format, isValidNumber, AsYouType } from 'libphonenumber-js'
  import VueInputUI from 'vue-input-ui'
  import 'vue-input-ui/dist/vue-input-ui.css'
  import SelectCountry from './_subs/CountrySelector'

  export default {
    name: 'VuePhoneNumberInput',
    components: {
      VueInputUI,
      SelectCountry
    },
    props: {
      value: { type: Object, default: Object }
    },
    data () {
      return {
        phoneNumberHint: '',
        formatter: null,
        numberIsValid: true
      }
    },
    created () {
      this.formatter = new AsYouType(this.codeCountry)
    },
    computed: {
      codeCountry: {
        get () {
          return this.value.code
        },
        set (country) {
          this.$emit('input', { phoneNumber: this.phoneNumber, code: country})
          // this.formatter = new AsYouType(country)
          // if (this.phoneNumber) {
          //   this.updateNumberAfterChangeCountry(this.phoneNumber)
          // }
        }
      },
      codesCountries () {
        return CodesCountries
      },
      phoneNumber: {
        get () {
          return this.value.phoneNumber
        },
        set (newPhone) {
          this.$emit('input', { phoneNumber: newPhone, code: this.codeCountry})
          // const inputNumber = this.updateFormatNumber(newPhone)
          // this.formatNumberLogic(inputNumber, newPhone)
        }
      }
    },
    methods: {
      resetFormatNumber (val) {
        this.formatter.reset()
        let format
        let chars = val.split('')
        chars.map((charac) => {
          format = this.formatter.input(charac)
        })
        return format
      },
      updateFormatNumber (val) {
        return this.formatter.input(val.slice(-1))
      },
      updateNumberAfterChangeCountry (val) {
        this.formatNumberLogic(this.resetFormatNumber(val), this.phoneNumber)
      },
      formatNumberLogic (inputNumber, val) {
        const numberParsed = parseNumber(val, this.codeCountry)
        if (Object.keys(numberParsed).length) {
          this.numberIsValid = isValidNumber(numberParsed)
          if (this.numberIsValid) {
            this.$emit('update-phone', {val: format(numberParsed.phone, this.codeCountry, 'International'), isValid: true})
            this.phoneNumberHint = format(numberParsed.phone, this.codeCountry, 'International')
          }
        } else if (val.length) {
          this.$emit('update-phone', {val: this.phoneNumber, isValid: false})
          this.phoneNumberHint = format(val, this.codeCountry, 'International')
          this.numberIsValid = false
        } else {
          this.$emit('update-phone', {val: null, isValid: false})
          this.phoneNumberHint = ''
          this.numberIsValid = true
        }
      }
    }
  }
</script>
<style lang="scss">
  @import "./assets/css/flexbox-helper.scss";
  @import "./assets/flags/flags.css";
  *, *::before, *::after {
    box-sizing: border-box;
  }
  .vue-phone-number-input {
    .select-country-container {
      flex: 0 0 130px;
    }
    .country-selector {
      cursor: pointer;
      &:hover {
        background: #CCC;
      }
    }
  }
</style>