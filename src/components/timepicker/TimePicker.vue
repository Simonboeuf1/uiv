<template>
  <section @click.stop>
    <table>
      <tbody>
      <tr v-if="controls" class="text-center">
        <td>
          <btn type="link" size="sm" @click="changeTime(1,1)" :disabled="readonly">
            <i :class="iconControlUp"></i>
          </btn>
        </td>
        <td>&nbsp;</td>
        <td>
          <btn type="link" size="sm" @click="changeTime(0,1)" :disabled="readonly">
            <i :class="iconControlUp"></i>
          </btn>
        </td>
        <td v-if="showMeridian">
        </td>
      </tr>
      <tr>
        <td class="form-group">
          <input
            ref="hoursInput"
            type="tel"
            pattern="\d*"
            class="form-control text-center"
            :style="`width: ${inputWidth}px`"
            @mouseup="selectInputValue"
            @keydown.prevent.up="changeTime(1, 1)"
            @keydown.prevent.down="changeTime(1, 0)"
            @wheel="onWheel($event, true)"
            placeholder="HH"
            v-model.lazy="hoursText"
            :readonly="readonly"
            maxlength="2"
            size="2">
        </td>
        <td>&nbsp;<b>:</b>&nbsp;</td>
        <td class="form-group">
          <input
            ref="minutesInput"
            type="tel"
            pattern="\d*"
            class="form-control text-center"
            :style="`width: ${inputWidth}px`"
            @mouseup="selectInputValue"
            @keydown.prevent.up="changeTime(0, 1)"
            @keydown.prevent.down="changeTime(0, 0)"
            @wheel="onWheel($event, false)"
            placeholder="MM"
            v-model.lazy="minutesText"
            :readonly="readonly"
            maxlength="2"
            size="2">
        </td>
        <td v-if="showMeridian">
          &nbsp;
          <btn
            data-action="toggleMeridian"
            :disabled="readonly"
            v-text="meridian?t('uiv.timePicker.am'):t('uiv.timePicker.pm')"
            @click="toggleMeridian"></btn>
        </td>
      </tr>
      <tr v-if="controls" class="text-center">
        <td>
          <btn type="link" size="sm" @click="changeTime(1,0)" :disabled="readonly">
            <i :class="iconControlDown"></i>
          </btn>
        </td>
        <td>&nbsp;</td>
        <td>
          <btn type="link" size="sm" @click="changeTime(0,0)" :disabled="readonly">
            <i :class="iconControlDown"></i>
          </btn>
        </td>
        <td v-if="showMeridian">
        </td>
      </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
  import Local from '../../mixins/localeMixin'
  import Btn from './../button/Btn'
  import {pad} from '../../utils/stringUtils'

  const maxHours = 23
  const zero = 0
  const maxMinutes = 59
  const cutUpAmAndPm = 12

  export default {
    components: {Btn},
    mixins: [Local],
    props: {
      value: {
        type: Date,
        required: true
      },
      showMeridian: {
        type: Boolean,
        default: true
      },
      min: Date,
      max: Date,
      hourStep: {
        type: Number,
        default: 1
      },
      minStep: {
        type: Number,
        default: 1
      },
      readonly: {
        type: Boolean,
        default: false
      },
      controls: {
        type: Boolean,
        default: true
      },
      iconControlUp: {
        type: String,
        default: 'glyphicon glyphicon-chevron-up'
      },
      iconControlDown: {
        type: String,
        default: 'glyphicon glyphicon-chevron-down'
      },
      inputWidth: {
        type: Number,
        default: 50
      }
    },
    data () {
      return {
        hours: 0,
        minutes: 0,
        meridian: true,
        hoursText: '',
        minutesText: ''
      }
    },
    mounted () {
      this.updateByValue(this.value)
    },
    watch: {
      value (value) {
        this.updateByValue(value)
      },
      showMeridian (value) {
        this.setTime()
      },
      hoursText (value) {
        if (this.hours === 0 && value === '') {
          // Prevent a runtime reset from being overwritten
          return
        }
        let hour = parseInt(value)
        if (this.showMeridian) {
          if (hour >= 1 && hour <= cutUpAmAndPm) {
            if (this.meridian) {
              this.hours = hour === cutUpAmAndPm ? 0 : hour
            } else {
              this.hours = hour === cutUpAmAndPm ? cutUpAmAndPm : hour + cutUpAmAndPm
            }
          }
        } else if (hour >= zero && hour <= maxHours) {
          this.hours = hour
        }
        this.setTime()
      },
      minutesText (value) {
        if (this.minutes === 0 && value === '') {
          // Prevent a runtime reset from being overwritten
          return
        }
        let minutesStr = parseInt(value)
        if (minutesStr >= zero && minutesStr <= maxMinutes) {
          this.minutes = minutesStr
        }
        this.setTime()
      }
    },
    methods: {
      updateByValue (value) {
        if (isNaN(value.getTime())) {
          this.hours = 0
          this.minutes = 0
          this.hoursText = ''
          this.minutesText = ''
          this.meridian = true
          return
        }
        this.hours = value.getHours()
        this.minutes = value.getMinutes()
        if (!this.showMeridian) {
          this.hoursText = pad(this.hours, 2)
        } else {
          if (this.hours >= cutUpAmAndPm) {
            if (this.hours === cutUpAmAndPm) {
              this.hoursText = this.hours + ''
            } else {
              this.hoursText = pad(this.hours - cutUpAmAndPm, 2)
            }
            this.meridian = false
          } else {
            if (this.hours === zero) {
              this.hoursText = cutUpAmAndPm.toString()
            } else {
              this.hoursText = pad(this.hours, 2)
            }
            this.meridian = true
          }
        }
        this.minutesText = pad(this.minutes, 2)
        // lazy model won't update when using keyboard up/down
        this.$refs.hoursInput.value = this.hoursText
        this.$refs.minutesInput.value = this.minutesText
      },
      addHour (step) {
        step = step || this.hourStep
        this.hours = this.hours >= maxHours ? zero : this.hours + step
      },
      reduceHour (step) {
        step = step || this.hourStep
        this.hours = this.hours <= zero ? maxHours : this.hours - step
      },
      addMinute () {
        if (this.minutes >= maxMinutes) {
          this.minutes = zero
          this.addHour(1)
        } else {
          this.minutes += this.minStep
        }
      },
      reduceMinute () {
        if (this.minutes <= zero) {
          this.minutes = maxMinutes + 1 - this.minStep
          this.reduceHour(1)
        } else {
          this.minutes -= this.minStep
        }
      },
      changeTime (isHour, isPlus) {
        if (!this.readonly) {
          if (isHour && isPlus) {
            this.addHour()
          } else if (isHour && !isPlus) {
            this.reduceHour()
          } else if (!isHour && isPlus) {
            this.addMinute()
          } else {
            this.reduceMinute()
          }
          this.setTime()
        }
      },
      toggleMeridian () {
        this.meridian = !this.meridian
        if (this.meridian) {
          this.hours -= cutUpAmAndPm
        } else {
          this.hours += cutUpAmAndPm
        }
        this.setTime()
      },
      onWheel (e, isHour) {
        if (!this.readonly) {
          e.preventDefault()
          this.changeTime(isHour, e.deltaY < 0)
        }
      },
      setTime () {
        let time = this.value
        if (isNaN(time.getTime())) {
          time = new Date()
          time.setHours(0)
          time.setMinutes(0)
        }
        time.setHours(this.hours)
        time.setMinutes(this.minutes)
        if (this.max) {
          let max = new Date(time)
          max.setHours(this.max.getHours())
          max.setMinutes(this.max.getMinutes())
          time = time > max ? max : time
        }
        if (this.min) {
          let min = new Date(time)
          min.setHours(this.min.getHours())
          min.setMinutes(this.min.getMinutes())
          time = time < min ? min : time
        }
        this.$emit('input', new Date(time))
      },
      selectInputValue (e) {
        // mouseup should be prevented!
        // See various comments in https://stackoverflow.com/questions/3272089/programmatically-selecting-text-in-an-input-field-on-ios-devices-mobile-safari
        e.target.setSelectionRange(0, 2)
      }
    }
  }
</script>
