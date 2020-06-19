<!-- *************************************************************************

This File will be used to show calendar

************************************************************************* -->

<template>
  <span :class="containerClass">
    <div class="inputbox">
      <CalendarInputText
        v-if="!inline"
        ref="input"
        v-model="inputFieldValue"
        type="text"
        v-bind="$attrs"
        :readonly="!manualInput"
        :aria-labelledby="ariaLabelledBy"
        v-on="listeners"
        :placeholder="placeholder"
      />

      <transition
        name="p-input-overlay"
        @enter="onOverlayEnter"
        @after-enter="onOverlayEnterComplete"
        @leave="onOverlayLeave"
      >
        <div
          v-if="inline ? true : overlayVisible"
          ref="overlay"
          :class="panelStyleClass"
          role="dialog"
          :aria-labelledby="ariaLabelledBy"
        >
          <template v-if="!timeOnly">
            <div
              v-for="(month, groupIndex) of months"
              :key="month.month + month.year"
              class="p-datepicker-group"
            >
              <div
                :class="{
                        'p-datepicker-header': !isDarkTheme,
                        'p-datepicker-header darktheme' : isDarkTheme
                      }"
              >
                <slot name="header"></slot>
                <div class="p-datepicker-title">
                  <span
                    v-if="!monthNavigator && view !== 'month'"
                    class="p-datepicker-month"
                  >{{ localeData.monthNames[month.month] }}</span>
                  <select
                    v-if="
                    monthNavigator && view !== 'month' && numberOfMonths === 1
                  "
                    class="p-datepicker-month"
                    @change="onMonthDropdownChange($event.target.value)"
                  >
                    <option
                      v-for="(monthName, index) of localeData.monthNames"
                      :key="monthName"
                      :value="index"
                      :selected="index === month.month"
                    >{{ monthName }}</option>
                  </select>
                  <span
                    v-if="!yearNavigator"
                    class="p-datepicker-year"
                  >{{ view === 'month' ? currentYear : month.year }}</span>
                  <select
                    v-if="yearNavigator && numberOfMonths === 1"
                    class="p-datepicker-year"
                    @change="onYearDropdownChange($event.target.value)"
                  >
                    <option
                      v-for="year of yearOptions"
                      :key="year"
                      :value="year"
                      :selected="year === currentYear"
                    >{{ year }}</option>
                  </select>
                </div>
                <button
                  v-if="groupIndex === 0"
                  class="p-datepicker-prev p-link"
                  type="button"
                  @click="onPrevButtonClick"
                  @keydown="onContainerButtonKeydown"
                >
                  <span class="p-datepicker-prev-icon pi pi-chevron-left"></span>
                </button>

                <button
                  v-if="
                  numberOfMonths === 1
                    ? true
                    : groupIndex === numberOfMonths - 1
                "
                  class="p-datepicker-next p-link"
                  type="button"
                  @click="onNextButtonClick"
                  @keydown="onContainerButtonKeydown"
                >
                  <span class="p-datepicker-next-icon pi pi-chevron-right"></span>
                </button>
              </div>
              <div v-if="view === 'date'" class="p-datepicker-calendar-container">
                <table class="p-datepicker-calendar">
                  <thead>
                    <tr>
                      <th v-if="showWeek" scope="col" class="p-datepicker-weekheader p-disabled">
                        <span>{{ localeData['weekHeader'] }}</span>
                      </th>
                      <th v-for="weekDay of weekDays" :key="weekDay" scope="col">
                        <span>{{ weekDay }}</span>
                      </th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(week, i) of month.dates" :key="week[0].day + '' + week[0].month">
                      <td v-if="showWeek" class="p-datepicker-weeknumber">
                        <span class="p-disabled">
                          <span v-if="month.weekNumbers[i] < 10" style="visibility:hidden">0</span>
                          {{ month.weekNumbers[i] }}
                        </span>
                      </td>
                      <td
                        v-for="date of week"
                        :key="date.day + '' + date.month"
                        :class="{
                        'p-datepicker-other-month': date.otherMonth,
                        'p-datepicker-today': date.today
                      }"
                      >
                        <span
                          :class="{
                          'p-highlight': isSelected(date),
                          'p-disabled': !date.selectable
                        }"
                          draggable="false"
                          @click="onDateSelect($event, date)"
                          @keydown="onDateCellKeydown($event, date, groupIndex)"
                        >
                          <slot name="date" :date="date">{{ date.day }}</slot>
                        </span>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <div v-if="view === 'month'" class="p-monthpicker">
              <span
                v-for="(m, i) of monthPickerValues"
                :key="m"
                class="p-monthpicker-month"
                :class="{ 'p-highlight': isMonthSelected(i) }"
                @click="onMonthSelect($event, i)"
                @keydown="onMonthCellKeydown($event, i)"
              >{{ m }}</span>
            </div>
          </template>
          <div v-if="showTime || timeOnly" class="p-timepicker">
            <div class="p-hour-picker" @click.prevent="updateModelTime">
              <button
                class="p-link"
                type="button"
                @mousedown="onTimePickerElementMouseDown($event, 0, 1)"
                @mouseup="onTimePickerElementMouseUp($event)"
                @keydown="onContainerButtonKeydown"
                @mouseleave="onTimePickerElementMouseLeave"
                @keydown.enter="onTimePickerElementMouseDown($event, 0, 1)"
                @keyup.enter="onTimePickerElementMouseUp($event)"
              >
                <span class="pi pi-chevron-up"></span>
              </button>
              <span :style="{ display: 'none' }">0</span>
              <span>{{ currentHour }}</span>
              <button
                class="p-link"
                type="button"
                @mousedown="onTimePickerElementMouseDown($event, 0, -1)"
                @mouseup="onTimePickerElementMouseUp($event)"
                @keydown="onContainerButtonKeydown"
                @mouseleave="onTimePickerElementMouseLeave"
                @keydown.enter="onTimePickerElementMouseDown($event, 0, -1)"
                @keyup.enter="onTimePickerElementMouseUp($event)"
              >
                <span class="pi pi-chevron-down"></span>
              </button>
            </div>
            <div class="p-separator">
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
              <span>{{ timeSeparator }}</span>
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
            </div>
            <div class="p-minute-picker" @click.prevent="updateModelTime">
              <button
                class="p-link"
                type="button"
                @mousedown="onTimePickerElementMouseDown($event, 1, 1)"
                @mouseup="onTimePickerElementMouseUp($event)"
                @keydown="onContainerButtonKeydown"
                @mouseleave="onTimePickerElementMouseLeave"
                @keydown.enter="onTimePickerElementMouseDown($event, 1, 1)"
                @keyup.enter="onTimePickerElementMouseUp($event)"
              >
                <span class="pi pi-chevron-up"></span>
              </button>
              <span :style="{ display: currentMinute < 10 ? 'inline' : 'none' }">0</span>
              <span>{{ currentMinute }}</span>
              <button
                class="p-link"
                type="button"
                @mousedown="onTimePickerElementMouseDown($event, 1, -1)"
                @mouseup="onTimePickerElementMouseUp($event)"
                @keydown="onContainerButtonKeydown"
                @mouseleave="onTimePickerElementMouseLeave"
                @keydown.enter="onTimePickerElementMouseDown($event, 1, -1)"
                @keyup.enter="onTimePickerElementMouseUp($event)"
              >
                <span class="pi pi-chevron-down"></span>
              </button>
            </div>
            <div v-if="showSeconds" class="p-separator">
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
              <span>{{ timeSeparator }}</span>
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
            </div>
            <div v-if="hourFormat === '12'" class="p-separator">
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
              <span>{{ timeSeparator }}</span>
              <span class="p-separator-spacer">
                <span class="pi pi-chevron-up"></span>
              </span>
            </div>

            <div v-if="hourFormat === '12'" class="p-ampm-picker" @click.prevent="updateModelTime">
              <button class="p-link" type="button" @click="toggleAMPM($event)">
                <span class="pi pi-chevron-up"></span>
              </button>
              <span>{{ pm ? 'PM' : 'AM' }}</span>
              <button class="p-link" type="button" @click="toggleAMPM($event)">
                <span class="pi pi-chevron-down"></span>
              </button>
            </div>
          </div>
          <div class="p-datepicker-buttonbar" v-if="showButtonBar">
            <CalendarButton
              type="button"
              :label="localeData['today']"
              @click="onTodayButtonClick($event)"
              class="p-button-secondary"
              @keydown="onContainerButtonKeydown"
            />
            <CalendarButton
              type="button"
              :label="localeData['clear']"
              @click="onClearButtonClick($event)"
              class="p-button-secondary"
              @keydown="onContainerButtonKeydown"
            />
          </div>
          <slot name="footer"></slot>
        </div>
      </transition>
      <!-- <img
      src="./images/dissmiss.svg"
      width="12"
      class="pointer"
      alt
      @click.prevent="onClearButtonClick"
      />-->
    </div>
  </span>
</template>

<!-- *************************************************************************

SCRIPT

************************************************************************* -->

<script>
// For load input text compoennt
import InputText from "./InputText.vue";

// For load Button compoennt
import Button from "./Button.vue";

import DomHandler from "./helpers/domhandler.js";
import moment from "moment";
import "primevue/resources/primevue.min.css";
import "primeicons/primeicons.css";
export default {
  components: {
    CalendarInputText: InputText,
    CalendarButton: Button
  },
  inheritAttrs: false,
  props: {
    id: {
      type: String,
      default: ""
    },
    value: null,
    selectionMode: {
      type: String,
      default: "single"
    },
    isDarkTheme: {
      type: Boolean,
      default: false
    },
    dateFormat: {
      type: String,
      default: "mm/dd/yy"
    },
    inline: {
      type: Boolean,
      default: false
    },
    showOtherMonths: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: "Time Picker"
    },
    selectOtherMonths: {
      type: Boolean,
      default: false
    },
    showIcon: {
      type: Boolean,
      default: false
    },
    icon: {
      type: String,
      default: "pi pi-calendar"
    },
    numberOfMonths: {
      type: Number,
      default: 1
    },
    view: {
      type: String,
      default: "date"
    },
    touchUI: {
      type: Boolean,
      default: false
    },
    monthNavigator: {
      type: Boolean,
      default: false
    },
    yearNavigator: {
      type: Boolean,
      default: false
    },
    yearRange: {
      type: String,
      default: "1960:2050"
    },
    panelClass: {
      type: String,
      default: null
    },
    panelStyle: {
      type: String,
      default: null
    },
    minDate: {
      type: Date,
      value: null
    },
    maxDate: {
      type: Date,
      value: null
    },
    disabledDates: {
      type: Array,
      value: null
    },
    disabledDays: {
      type: Array,
      value: null
    },
    maxDateCount: {
      type: Number,
      value: null
    },
    showOnFocus: {
      type: Boolean,
      default: false
    },
    autoZIndex: {
      type: Boolean,
      default: false
    },
    baseZIndex: {
      type: Number,
      default: 0
    },
    showButtonBar: {
      type: Boolean,
      default: false
    },
    shortYearCutoff: {
      type: String,
      default: "+10"
    },
    timeOnly: {
      type: Boolean,
      default: false
    },
    showTime: {
      type: Boolean,
      default: true
    },
    hourFormat: {
      type: String,
      default: "12"
    },
    stepHour: {
      type: Number,
      default: 1
    },
    stepMinute: {
      type: Number,
      default: 1
    },
    stepSecond: {
      type: Number,
      default: 1
    },
    showSeconds: {
      type: Boolean,
      default: false
    },
    hideOnDateTimeSelect: {
      type: Boolean,
      default: false
    },
    timeSeparator: {
      type: String,
      default: ":"
    },
    showWeek: {
      type: Boolean,
      default: false
    },
    manualInput: {
      type: Boolean,
      default: false
    },
    tabindex: {
      type: String
    },
    locale: {
      type: String,
      default: "en"
    },
    ariaLabelledBy: {
      type: String,
      default: null
    }
  },
  oldViewDate: null,
  navigationState: null,
  data() {
    return {
      currentMonth: null,
      currentYear: null,
      currentHour: null,
      currentMinute: null,
      currentSecond: null,
      pm: null,
      focused: false,
      overlayVisible: false,
      inputFieldValue: null,
      localeData: {}
    };
  },
  computed: {
    listeners() {
      const $vm = this;
      return {
        ...$vm.$listeners,
        input: val => {
          if (!$vm.isKeydown) {
            return;
          }
          $vm.isKeydown = false;
          try {
            const value = $vm.parseValueFromString(val);
            if ($vm.isValidSelection(value)) {
              $vm.updateModel(value);
            }
          } catch (err) {
            // invalid date
            $vm.updateModel(null);
            $vm.inputFieldValue = val;
          }
        },
        focus: event => {
          $vm.focus = true;
          if ($vm.showOnFocus && $vm.isEnabled()) {
            $vm.overlayVisible = true;
          }
          $vm.overlayVisible = true;
          $vm.focused = true;
          $vm.$emit("focus", event);
        },
        blur: event => {
          try {
            const value = $vm.parseValueFromString(event.target.value);
            if ($vm.isValidSelection(value)) {
              $vm.updateInputFieldValue(value);
            }
          } catch (err) {
            // invalid date
            $vm.inputFieldValue = null;
          }
          $vm.focused = false;
          $vm.$emit("blur", event);
        },
        keydown: event => {
          $vm.isKeydown = true;
          switch (event.which) {
            // escape
            case 27: {
              $vm.overlayVisible = false;
              break;
            }
            // tab
            case 9: {
              if ($vm.touchUI) {
                $vm.disableModality();
              }
              if (event.shiftKey) {
                $vm.overlayVisible = false;
              }
              break;
            }
            default:
              // no op
              break;
          }
          $vm.$emit("keydown", event);
        }
      };
    },
    viewDate() {
      const $vm = this;
      let propValue = this.value;
      if (propValue && Array.isArray(propValue)) {
        propValue = propValue[0];
      }
      const vDate = propValue || $vm.oldViewDate || new Date();
      $vm.oldViewDate = vDate;
      return vDate;
    },
    containerClass() {
      return [
        "p-calendar",
        {
          "p-calendar-w-btn": this.showIcon,
          "p-calendar-timeonly": this.timeOnly,
          "p-inputwrapper-filled": this.value,
          "p-inputwrapper-focus": this.focused
        }
      ];
    },
    panelStyleClass() {
      return [
        "p-datepicker p-component",
        {
          "p-disabled": this.$attrs.disabled,
          "p-datepicker-timeonly": this.timeOnly,
          "p-datepicker-multiple-month": this.numberOfMonths > 1,
          "p-datepicker-monthpicker": this.view === "month",
          "p-datepicker-touch-ui": this.touchUI,
          "p-Datepicker-darktheme": this.isDarkTheme
        }
      ];
    },
    months() {
      const months = [];
      for (let i = 0; i < this.numberOfMonths; i++) {
        let month = this.currentMonth + i;
        let year = this.currentYear;
        if (month > 11) {
          month = (month % 11) - 1;
          year = year + 1;
        }
        const dates = [];
        const firstDay = this.getFirstDayOfMonthIndex(month, year);
        const daysLength = this.getDaysCountInMonth(month, year);
        const prevMonthDaysLength = this.getDaysCountInPrevMonth(month, year);
        let dayNo = 1;
        const today = new Date();
        const weekNumbers = [];
        const monthRows = Math.ceil((daysLength + firstDay) / 7);
        for (let i = 0; i < monthRows; i++) {
          const week = [];
          if (i === 0) {
            for (
              let j = prevMonthDaysLength - firstDay + 1;
              j <= prevMonthDaysLength;
              j++
            ) {
              const prev = this.getPreviousMonthAndYear(month, year);
              week.push({
                day: j,
                month: prev.month,
                year: prev.year,
                otherMonth: true,
                today: this.isToday(today, j, prev.month, prev.year),
                selectable: this.isSelectable(j, prev.month, prev.year, true)
              });
            }
            const remainingDaysLength = 7 - week.length;
            for (let j = 0; j < remainingDaysLength; j++) {
              week.push({
                day: dayNo,
                month: month,
                year: year,
                today: this.isToday(today, dayNo, month, year),
                selectable: this.isSelectable(dayNo, month, year, false)
              });
              dayNo++;
            }
          } else {
            for (let j = 0; j < 7; j++) {
              if (dayNo > daysLength) {
                const next = this.getNextMonthAndYear(month, year);
                week.push({
                  day: dayNo - daysLength,
                  month: next.month,
                  year: next.year,
                  otherMonth: true,
                  today: this.isToday(
                    today,
                    dayNo - daysLength,
                    next.month,
                    next.year
                  ),
                  selectable: this.isSelectable(
                    dayNo - daysLength,
                    next.month,
                    next.year,
                    true
                  )
                });
              } else {
                week.push({
                  day: dayNo,
                  month: month,
                  year: year,
                  today: this.isToday(today, dayNo, month, year),
                  selectable: this.isSelectable(dayNo, month, year, false)
                });
              }
              dayNo++;
            }
          }
          if (this.showWeek) {
            weekNumbers.push(
              this.getWeekNumber(
                new Date(week[0].year, week[0].month, week[0].day)
              )
            );
          }
          dates.push(week);
        }
        months.push({
          month: month,
          year: year,
          dates: dates,
          weekNumbers: weekNumbers
        });
      }
      return months;
    },
    weekDays() {
      const weekDays = [];
      let dayIndex = this.localeData.firstDayOfWeek;
      for (let i = 0; i < 7; i++) {
        weekDays.push(this.localeData.dayNamesMin[dayIndex]);
        dayIndex = dayIndex === 6 ? 0 : ++dayIndex;
      }
      return weekDays;
    },
    ticksTo1970() {
      return (
        ((1970 - 1) * 365 +
          Math.floor(1970 / 4) -
          Math.floor(1970 / 100) +
          Math.floor(1970 / 400)) *
        24 *
        60 *
        60 *
        10000000
      );
    },
    sundayIndex() {
      return this.localeData.firstDayOfWeek > 0
        ? 7 - this.localeData.firstDayOfWeek
        : 0;
    },
    datePattern() {
      return this.dateFormat || this.localeData.dateFormat;
    },
    yearOptions() {
      if (this.yearRange) {
        const $vm = this;
        const years = this.yearRange.split(":");
        const yearStart = parseInt(years[0]);
        const yearEnd = parseInt(years[1]);
        const yearOptions = [];
        if (this.currentYear < yearStart) {
          $vm.currentYear = yearEnd;
        } else if (this.currentYear > yearEnd) {
          $vm.currentYear = yearStart;
        }
        for (let i = yearStart; i <= yearEnd; i++) {
          yearOptions.push(i);
        }
        return yearOptions;
      } else {
        return null;
      }
    },
    monthPickerValues() {
      const monthPickerValues = [];
      for (let i = 0; i <= 11; i++) {
        monthPickerValues.push(this.localeData.monthNamesShort[i]);
      }
      return monthPickerValues;
    }
  },
  watch: {
    value(newValue) {
      this.updateCurrentMetaData();
      this.updateInputFieldValue(newValue || this.inputFieldValue);
    }
  },
  created() {
    this.updateCurrentMetaData();
    this.updateInputFieldValue(this.value);
  },
  mounted() {
    moment.locale(this.locale);
    this.localeData = {
      firstDayOfWeek: 0,
      dayNames: moment.weekdays(),
      dayNamesShort: moment.weekdaysShort(),
      dayNamesMin: moment.weekdaysMin(),
      monthNames: moment.months(),
      monthNamesShort: moment.monthsShort(),
      today: "Today",
      clear: "Clear",
      dateFormat: moment.localeData("it").longDateFormat("L"),
      weekHeader: "Wk"
    };
    this.initFocusableCell();
  },
  updated() {
    if (this.$refs.overlay) {
      this.updateFocus();
    }
  },
  beforeDestroy() {
    if (this.timePickerTimer) {
      clearTimeout(this.timePickerTimer);
    }
    if (this.mask) {
      this.disableModality();
      this.mask = null;
    }
    this.unbindOutsideClickListener();
  },
  outsideClickListener: null,
  maskClickListener: null,
  mask: null,
  timePickerTimer: null,
  isKeydown: false,
  methods: {
    /**
     * Method to check date is selected or not
     */
    isSelected(dateMeta) {
      if (this.value) {
        if (this.isSingleSelection()) {
          return this.isDateEquals(this.value, dateMeta);
        } else if (this.isMultipleSelection()) {
          let selected = false;
          for (const date of this.value) {
            selected = this.isDateEquals(date, dateMeta);
            if (selected) {
              break;
            }
          }
          return selected;
        } else if (this.isRangeSelection()) {
          if (this.value[1])
            return (
              this.isDateEquals(this.value[0], dateMeta) ||
              this.isDateEquals(this.value[1], dateMeta) ||
              this.isDateBetween(this.value[0], this.value[1], dateMeta)
            );
          else {
            return this.isDateEquals(this.value[0], dateMeta);
          }
        }
      } else {
        return false;
      }
    },
    /**
     * Method to check month is selected or not
     */
    isMonthSelected(month) {
      return this.value
        ? this.value.getMonth() === month &&
            this.value.getFullYear() === this.currentYear
        : false;
    },
    /**
     * Method to check date is equal or not
     */
    isDateEquals(value, dateMeta) {
      if (value)
        return (
          value.getDate() === dateMeta.day &&
          value.getMonth() === dateMeta.month &&
          value.getFullYear() === dateMeta.year
        );
      else return false;
    },
    /**
     * Method to check date is between selected date period or not
     */
    isDateBetween(start, end, dateMeta) {
      const between = false;
      if (start && end) {
        const date = new Date(dateMeta.year, dateMeta.month, dateMeta.day);
        return (
          start.getTime() <= date.getTime() && end.getTime() >= date.getTime()
        );
      }
      return between;
    },
    /**
     * Method to get first day of month
     */
    getFirstDayOfMonthIndex(month, year) {
      const day = new Date();
      day.setDate(1);
      day.setMonth(month);
      day.setFullYear(year);
      const dayIndex = day.getDay() + this.sundayIndex;
      return dayIndex >= 7 ? dayIndex - 7 : dayIndex;
    },
    /**
     * Method to get days count in month
     */
    getDaysCountInMonth(month, year) {
      return (
        32 - this.daylightSavingAdjust(new Date(year, month, 32)).getDate()
      );
    },
    /**
     * Method to get days count in previous month
     */
    getDaysCountInPrevMonth(month, year) {
      const prev = this.getPreviousMonthAndYear(month, year);
      return this.getDaysCountInMonth(prev.month, prev.year);
    },
    /**
     * Method to get previous month and year
     */
    getPreviousMonthAndYear(month, year) {
      let m, y;
      if (month === 0) {
        m = 11;
        y = year - 1;
      } else {
        m = month - 1;
        y = year;
      }
      return { month: m, year: y };
    },
    /**
     * Method to get next month and year
     */
    getNextMonthAndYear(month, year) {
      let m, y;
      if (month === 11) {
        m = 0;
        y = year + 1;
      } else {
        m = month + 1;
        y = year;
      }
      return { month: m, year: y };
    },
    /**
     * Method to save daylight adjustment
     */
    daylightSavingAdjust(date) {
      if (!date) {
        return null;
      }
      date.setHours(date.getHours() > 12 ? date.getHours() + 2 : 0);
      return date;
    },
    /**
     * Method to date is of oday or not
     */
    isToday(today, day, month, year) {
      return (
        today.getDate() === day &&
        today.getMonth() === month &&
        today.getFullYear() === year
      );
    },
    /**
     * Method to check date is selectable or not
     */
    isSelectable(day, month, year, otherMonth) {
      let validMin = true;
      let validMax = true;
      let validDate = true;
      let validDay = true;
      if (otherMonth && !this.selectOtherMonths) {
        return false;
      }
      if (this.minDate) {
        if (this.minDate.getFullYear() > year) {
          validMin = false;
        } else if (this.minDate.getFullYear() === year) {
          if (this.minDate.getMonth() > month) {
            validMin = false;
          } else if (this.minDate.getMonth() === month) {
            if (this.minDate.getDate() > day) {
              validMin = false;
            }
          }
        }
      }
      if (this.maxDate) {
        if (this.maxDate.getFullYear() < year) {
          validMax = false;
        } else if (this.maxDate.getFullYear() === year) {
          if (this.maxDate.getMonth() < month) {
            validMax = false;
          } else if (this.maxDate.getMonth() === month) {
            if (this.maxDate.getDate() < day) {
              validMax = false;
            }
          }
        }
      }
      if (this.disabledDates) {
        validDate = !this.isDateDisabled(day, month, year);
      }
      if (this.disabledDays) {
        validDay = !this.isDayDisabled(day, month, year);
      }
      return validMin && validMax && validDate && validDay;
    },
    /**
     * On overlay enter
     */
    onOverlayEnter() {
      if (this.autoZIndex) {
        this.$refs.overlay.style.zIndex = String(
          this.baseZIndex + DomHandler.generateZIndex()
        );
      }
      this.alignOverlay();
      this.$emit("show");
    },
    /**
     * On overlay enter complete
     */
    onOverlayEnterComplete() {
      this.bindOutsideClickListener();
    },
    /**
     * On overlay enter leave
     */
    onOverlayLeave() {
      this.unbindOutsideClickListener();
      this.$emit("hide");
    },
    /**
     * On previous button click
     */
    onPrevButtonClick(event) {
      this.navigationState = { backward: true, button: true };
      this.navBackward(event);
    },
    /**
     * On next button click
     */
    onNextButtonClick(event) {
      this.navigationState = { backward: false, button: true };
      this.navForward(event);
    },
    /**
     * On backward navigation
     */
    navBackward(event) {
      event.preventDefault();
      if (!this.isEnabled()) {
        return;
      }
      if (this.view === "month") {
        this.decrementYear();
      } else {
        if (this.currentMonth === 0) {
          this.currentMonth = 11;
          this.decrementYear();
        } else {
          this.currentMonth--;
        }
        this.$emit("month-change", {
          month: this.currentMonth,
          year: this.currentYear
        });
      }
    },
    /**
     * On forward navigation
     */
    navForward(event) {
      event.preventDefault();
      if (!this.isEnabled()) {
        return;
      }
      if (this.view === "month") {
        this.incrementYear();
      } else {
        if (this.currentMonth === 11) {
          this.currentMonth = 0;
          this.incrementYear();
        } else {
          this.currentMonth++;
        }
        this.$emit("month-change", {
          month: this.currentMonth,
          year: this.currentYear
        });
      }
    },
    /**
     * Method to decrement year
     */
    decrementYear() {
      this.currentYear--;
    },
    /**
     * Method to increment year
     */
    incrementYear() {
      this.currentYear++;
    },
    /**
     * Method to check enabled or not
     */
    isEnabled() {
      return !this.$attrs.disabled && !this.$attrs.readonly;
    },
    /**
     * Method to update current time meta
     */
    updateCurrentTimeMeta(date) {
      const hours = date.getHours();
      if (this.hourFormat === "12") {
        this.pm = hours > 11;
        if (hours >= 12) this.currentHour = hours === 12 ? 12 : hours - 12;
        else this.currentHour = hours === 0 ? 12 : hours;
      } else {
        this.currentHour = date.getHours();
      }
      this.currentMinute = date.getMinutes();
      this.currentSecond = date.getSeconds();
    },
    /**
     * Method to update current time meta
     */
    bindOutsideClickListener() {
      if (!this.outsideClickListener) {
        this.outsideClickListener = event => {
          if (this.overlayVisible && this.isOutsideClicked(event)) {
            this.overlayVisible = false;
          }
        };
        document.addEventListener("mousedown", this.outsideClickListener);
      }
    },
    /**
     * Method to update current time meta
     */
    unbindOutsideClickListener() {
      if (this.outsideClickListener) {
        document.removeEventListener("mousedown", this.outsideClickListener);
        this.outsideClickListener = null;
      }
    },
    isOutsideClicked(event) {
      return !(
        this.$el.isSameNode(event.target) ||
        this.isNavIconClicked(event) ||
        this.$el.contains(event.target) ||
        (this.$refs.overlay && this.$refs.overlay.contains(event.target))
      );
    },
    isNavIconClicked(event) {
      return (
        DomHandler.hasClass(event.target, "p-datepicker-prev") ||
        DomHandler.hasClass(event.target, "p-datepicker-prev-icon") ||
        DomHandler.hasClass(event.target, "p-datepicker-next") ||
        DomHandler.hasClass(event.target, "p-datepicker-next-icon")
      );
    },
    alignOverlay() {
      if (this.touchUI) {
        this.enableModality();
      } else if (this.$refs.overlay) {
        DomHandler.relativePosition(this.$refs.overlay, this.$el);
      }
    },
    onButtonClick() {
      if (this.isEnabled()) {
        if (!this.overlayVisible) {
          this.$refs.input.$el.focus();
          this.overlayVisible = true;
        } else {
          this.overlayVisible = false;
        }
      }
    },
    isDateDisabled(day, month, year) {
      if (this.disabledDates) {
        for (const disabledDate of this.disabledDates) {
          if (
            disabledDate.getFullYear() === year &&
            disabledDate.getMonth() === month &&
            disabledDate.getDate() === day
          ) {
            return true;
          }
        }
      }
      return false;
    },
    isDayDisabled(day, month, year) {
      if (this.disabledDays) {
        const weekday = new Date(year, month, day);
        const weekdayNumber = weekday.getDay();
        return this.disabledDays.indexOf(weekdayNumber) !== -1;
      }
      return false;
    },
    onMonthDropdownChange(value) {
      this.currentMonth = parseInt(value);
      this.$emit("month-change", {
        month: this.currentMonth + 1,
        year: this.currentYear
      });
    },
    onYearDropdownChange(value) {
      this.currentYear = parseInt(value);
      this.$emit("year-change", {
        month: this.currentMonth + 1,
        year: this.currentYear
      });
    },
    onDateSelect(event, dateMeta) {
      if (this.$attrs.disabled || !dateMeta.selectable) {
        return;
      }
      DomHandler.find(
        this.$refs.overlay,
        ".p-datepicker-calendar td span:not(.p-disabled)"
      ).forEach(cell => (cell.tabIndex = -1));
      if (event) {
        event.currentTarget.focus();
      }
      if (this.isMultipleSelection() && this.isSelected(dateMeta)) {
        const newValue = this.value.filter(
          date => !this.isDateEquals(date, dateMeta)
        );
        this.updateModel(newValue);
        this.updateInputFieldValue(newValue);
      } else {
        if (this.shouldSelectDate(dateMeta)) {
          if (dateMeta.otherMonth) {
            this.currentMonth = dateMeta.month;
            this.currentYear = dateMeta.year;
            this.selectDate(dateMeta);
          } else {
            this.selectDate(dateMeta);
          }
        }
      }
      if (
        this.isSingleSelection() &&
        (!this.showTime || this.hideOnDateTimeSelect)
      ) {
        setTimeout(() => {
          this.overlayVisible = false;
          if (this.mask) {
            this.disableModality();
          }
        }, 150);
      }
    },
    selectDate(dateMeta) {
      let date = new Date(dateMeta.year, dateMeta.month, dateMeta.day);
      if (this.showTime) {
        if (this.hourFormat === "12" && this.pm && this.currentHour !== 12)
          date.setHours(this.currentHour + 12);
        else date.setHours(this.currentHour);
        date.setMinutes(this.currentMinute);
        date.setSeconds(this.currentSecond);
      }
      if (this.minDate && this.minDate > date) {
        date = this.minDate;
        this.currentHour = date.getHours();
        this.currentMinute = date.getMinutes();
        this.currentSecond = date.getSeconds();
      }
      if (this.maxDate && this.maxDate < date) {
        date = this.maxDate;
        this.currentHour = date.getHours();
        this.currentMinute = date.getMinutes();
        this.currentSecond = date.getSeconds();
      }
      let modelVal = null;
      if (this.isSingleSelection()) {
        modelVal = date;
      } else if (this.isMultipleSelection()) {
        modelVal = this.value ? [...this.value, date] : [date];
      } else if (this.isRangeSelection()) {
        if (this.value && this.value.length) {
          let startDate = this.value[0];
          let endDate = this.value[1];
          if (!endDate && date.getTime() >= startDate.getTime()) {
            endDate = date;
          } else {
            startDate = date;
            endDate = null;
          }
          modelVal = [startDate, endDate];
        } else {
          modelVal = [date, null];
        }
      }
      if (modelVal !== null) {
        this.updateModel(modelVal);
        this.updateInputFieldValue(modelVal);
      }
      this.$emit("date-select", date);
    },
    updateModel(value) {
      this.$emit("input", value);
    },
    updateInputFieldValue(date) {
      let formattedValue = "";
      if (date) {
        try {
          if (this.isSingleSelection()) {
            formattedValue = this.formatDateTime(date);
          } else if (this.isMultipleSelection()) {
            for (let i = 0; i < date.length; i++) {
              const dateAsString = this.formatDateTime(date[i]);
              formattedValue += dateAsString;
              if (i !== date.length - 1) {
                formattedValue += ", ";
              }
            }
          } else if (this.isRangeSelection()) {
            if (date && date.length) {
              const startDate = date[0];
              const endDate = date[1];
              formattedValue = this.formatDateTime(startDate);
              if (endDate) {
                formattedValue += " - " + this.formatDateTime(endDate);
              }
            }
          }
        } catch (err) {
          formattedValue = date;
        }
      }
      this.inputFieldValue = formattedValue;
    },
    shouldSelectDate() {
      if (this.isMultipleSelection())
        return this.maxDateCount != null
          ? this.maxDateCount > (this.value ? this.value.length : 0)
          : true;
      else return true;
    },
    isSingleSelection() {
      return this.selectionMode === "single";
    },
    isRangeSelection() {
      return this.selectionMode === "range";
    },
    isMultipleSelection() {
      return this.selectionMode === "multiple";
    },
    formatDateTime(date) {
      let formattedValue = null;
      if (date) {
        if (this.timeOnly) {
          formattedValue = this.formatTime(date);
        } else {
          formattedValue = this.formatDate(date, this.datePattern);
          if (this.showTime) {
            formattedValue += " " + this.formatTime(date);
          }
        }
      }
      return formattedValue;
    },
    formatDate(date, format) {
      if (!date) {
        return "";
      }
      let iFormat;
      const lookAhead = match => {
        const matches =
          iFormat + 1 < format.length && format.charAt(iFormat + 1) === match;
        if (matches) {
          iFormat++;
        }
        return matches;
      };
      const formatNumber = (match, value, len) => {
        let num = "" + value;
        if (lookAhead(match)) {
          while (num.length < len) {
            num = "0" + num;
          }
        }
        return num;
      };
      const formatName = (match, value, shortNames, longNames) => {
        return lookAhead(match) ? longNames[value] : shortNames[value];
      };
      let output = "";
      let literal = false;
      if (date) {
        for (iFormat = 0; iFormat < format.length; iFormat++) {
          if (literal) {
            if (format.charAt(iFormat) === "'" && !lookAhead("'")) {
              literal = false;
            } else {
              output += format.charAt(iFormat);
            }
          } else {
            switch (format.charAt(iFormat)) {
              case "d":
                output += formatNumber("d", date.getDate(), 2);
                break;
              case "D":
                output += formatName(
                  "D",
                  date.getDay(),
                  this.localeData.dayNamesShort,
                  this.localeData.dayNames
                );
                break;
              case "o":
                output += formatNumber(
                  "o",
                  Math.round(
                    (new Date(
                      date.getFullYear(),
                      date.getMonth(),
                      date.getDate()
                    ).getTime() -
                      new Date(date.getFullYear(), 0, 0).getTime()) /
                      86400000
                  ),
                  3
                );
                break;
              case "m":
                output += formatNumber("m", date.getMonth() + 1, 2);
                break;
              case "M":
                output += formatName(
                  "M",
                  date.getMonth(),
                  this.localeData.monthNamesShort,
                  this.localeData.monthNames
                );
                break;
              case "y":
                output += lookAhead("y")
                  ? date.getFullYear()
                  : (date.getFullYear() % 100 < 10 ? "0" : "") +
                    (date.getFullYear() % 100);
                break;
              case "@":
                output += date.getTime();
                break;
              case "!":
                output += date.getTime() * 10000 + this.ticksTo1970;
                break;
              case "'":
                if (lookAhead("'")) {
                  output += "'";
                } else {
                  literal = true;
                }
                break;
              default:
                output += format.charAt(iFormat);
            }
          }
        }
      }
      return output;
    },
    formatTime(date) {
      if (!date) {
        return "";
      }
      let output = "";
      let hours = date.getHours();
      const minutes = date.getMinutes();
      const seconds = date.getSeconds();
      if (this.hourFormat === "12" && hours > 11 && hours !== 12) {
        hours -= 12;
      }
      if (this.hourFormat === "12") {
        output += hours === 0 ? 12 : hours < 10 ? "0" + hours : hours;
      } else {
        output += hours < 10 ? "0" + hours : hours;
      }
      output += ":";
      output += minutes < 10 ? "0" + minutes : minutes;
      if (this.showSeconds) {
        output += ":";
        output += seconds < 10 ? "0" + seconds : seconds;
      }
      if (this.hourFormat === "12") {
        output += date.getHours() > 11 ? " PM" : " AM";
      }
      return output;
    },
    onTodayButtonClick(event) {
      const date = new Date();
      const dateMeta = {
        day: date.getDate(),
        month: date.getMonth(),
        year: date.getFullYear(),
        otherMonth:
          date.getMonth() !== this.currentMonth ||
          date.getFullYear() !== this.currentYear,
        today: true,
        selectable: true
      };
      this.onDateSelect(null, dateMeta);
      this.$emit("today-click", date);
      event.preventDefault();
    },
    onClearButtonClick(event) {
      this.inputFieldValue = "";
      const balnkArray = [];
      this.updateModel(balnkArray);
      this.overlayVisible = false;
      event.preventDefault();
    },
    onTimePickerElementMouseDown(event, type, direction) {
      if (this.isEnabled()) {
        this.repeat(event, null, type, direction);
        event.preventDefault();
      }
    },
    onTimePickerElementMouseUp(event) {
      if (this.isEnabled()) {
        this.clearTimePickerTimer();
        this.updateModelTime();
        event.preventDefault();
      }
    },
    onTimePickerElementMouseLeave() {
      this.clearTimePickerTimer();
    },
    repeat(event, interval, type, direction) {
      const i = interval || 500;
      this.clearTimePickerTimer();
      this.timePickerTimer = setTimeout(() => {
        this.repeat(event, 100, type, direction);
      }, i);
      switch (type) {
        case 0:
          if (direction === 1) this.incrementHour(event);
          else this.decrementHour(event);
          break;
        case 1:
          if (direction === 1) this.incrementMinute(event);
          else this.decrementMinute(event);
          break;
        case 2:
          if (direction === 1) this.incrementSecond(event);
          else this.decrementSecond(event);
          break;
      }
    },
    incrementHour(event) {
      const prevHour = this.currentHour;
      const newHour = this.currentHour + this.stepHour;
      if (this.validateHour(newHour)) {
        if (this.hourFormat === "24")
          this.currentHour = newHour >= 24 ? newHour - 24 : newHour;
        else if (this.hourFormat === "12") {
          // Before the AM/PM break, now after
          if (prevHour < 12 && newHour > 11) {
            this.pm = !this.pm;
          }
          this.currentHour = newHour >= 13 ? newHour - 12 : newHour;
        }
      }
      event.preventDefault();
    },
    decrementHour(event) {
      const newHour = this.currentHour - this.stepHour;
      if (this.validateHour(newHour)) {
        if (this.hourFormat === "24")
          this.currentHour = newHour < 0 ? 24 + newHour : newHour;
        else if (this.hourFormat === "12") {
          // If we were at noon/midnight, then switch
          if (this.currentHour === 12) {
            this.pm = !this.pm;
          }
          this.currentHour = newHour <= 0 ? 12 + newHour : newHour;
        }
      }
      event.preventDefault();
    },
    validateHour(hour) {
      let valid = true;
      let value = this.value;
      if (this.isRangeSelection()) {
        value = this.value[1] || this.value[0];
      }
      if (this.isMultipleSelection()) {
        value = this.value[this.value.length - 1];
      }
      const valueDateString = value ? value.toDateString() : null;
      if (
        this.minDate &&
        valueDateString &&
        this.minDate.toDateString() === valueDateString
      ) {
        if (this.minDate.getHours() > hour) {
          valid = false;
        }
      }
      if (
        this.maxDate &&
        valueDateString &&
        this.maxDate.toDateString() === valueDateString
      ) {
        if (this.maxDate.getHours() < hour) {
          valid = false;
        }
      }
      return valid;
    },
    incrementMinute(event) {
      const newMinute = this.currentMinute + this.stepMinute;
      if (this.validateMinute(newMinute)) {
        this.currentMinute = newMinute > 59 ? newMinute - 60 : newMinute;
      }
      event.preventDefault();
    },
    decrementMinute(event) {
      let newMinute = this.currentMinute - this.stepMinute;
      newMinute = newMinute < 0 ? 60 + newMinute : newMinute;
      if (this.validateMinute(newMinute)) {
        this.currentMinute = newMinute;
      }
      event.preventDefault();
    },
    validateMinute(minute) {
      let valid = true;
      let value = this.value;
      if (this.isRangeSelection()) {
        value = this.value[1] || this.value[0];
      }
      if (this.isMultipleSelection()) {
        value = this.value[this.value.length - 1];
      }
      const valueDateString = value ? value.toDateString() : null;
      if (
        this.minDate &&
        valueDateString &&
        this.minDate.toDateString() === valueDateString
      ) {
        if (value.getHours() === this.minDate.getHours()) {
          if (this.minDate.getMinutes() > minute) {
            valid = false;
          }
        }
      }
      if (
        this.maxDate &&
        valueDateString &&
        this.maxDate.toDateString() === valueDateString
      ) {
        if (value.getHours() === this.maxDate.getHours()) {
          if (this.maxDate.getMinutes() < minute) {
            valid = false;
          }
        }
      }
      return valid;
    },
    incrementSecond(event) {
      const newSecond = this.currentSecond + this.stepSecond;
      if (this.validateSecond(newSecond)) {
        this.currentSecond = newSecond > 59 ? newSecond - 60 : newSecond;
      }
      event.preventDefault();
    },
    decrementSecond(event) {
      let newSecond = this.currentSecond - this.stepSecond;
      newSecond = newSecond < 0 ? 60 + newSecond : newSecond;
      if (this.validateSecond(newSecond)) {
        this.currentSecond = newSecond;
      }
      event.preventDefault();
    },
    validateSecond(second) {
      let valid = true;
      let value = this.value;
      if (this.isRangeSelection()) {
        value = this.value[1] || this.value[0];
      }
      if (this.isMultipleSelection()) {
        value = this.value[this.value.length - 1];
      }
      const valueDateString = value ? value.toDateString() : null;
      if (
        this.minDate &&
        valueDateString &&
        this.minDate.toDateString() === valueDateString
      ) {
        if (this.minDate.getSeconds() > second) {
          valid = false;
        }
      }
      if (
        this.maxDate &&
        valueDateString &&
        this.maxDate.toDateString() === valueDateString
      ) {
        if (this.maxDate.getSeconds() < second) {
          valid = false;
        }
      }
      return valid;
    },
    updateModelTime() {
      let value = this.value;
      if (this.isRangeSelection()) {
        value = this.value[1] || this.value[0];
      }
      if (this.isMultipleSelection()) {
        value = this.value[this.value.length - 1];
      }
      value = value ? new Date(value.getTime()) : new Date();
      if (this.hourFormat === "12") {
        if (this.currentHour === 12) value.setHours(this.pm ? 12 : 0);
        else value.setHours(this.pm ? this.currentHour + 12 : this.currentHour);
      } else {
        value.setHours(this.currentHour);
      }
      value.setMinutes(this.currentMinute);
      value.setSeconds(this.currentSecond);
      if (this.isRangeSelection()) {
        if (this.value[1]) value = [this.value[0], value];
        else value = [value, null];
      }
      if (this.isMultipleSelection()) {
        value = [...this.value.slice(0, -1), value];
      }
      this.updateModel(value);
      this.updateInputFieldValue(value);
      this.$emit("date-select", value);
    },
    toggleAMPM(event) {
      this.pm = !this.pm;
      this.updateModelTime();
      event.preventDefault();
    },
    clearTimePickerTimer() {
      if (this.timePickerTimer) {
        clearInterval(this.timePickerTimer);
      }
    },
    onMonthSelect(event, index) {
      this.onDateSelect(event, {
        year: this.currentYear,
        month: index,
        day: 1,
        selectable: true
      });
    },
    enableModality() {
      if (!this.mask) {
        this.mask = document.createElement("div");
        this.mask.style.zIndex = String(
          parseInt(this.$refs.overlay.style.zIndex, 10) - 1
        );
        DomHandler.addMultipleClasses(
          this.mask,
          "p-component-overlay p-datepicker-mask p-datepicker-mask-scrollblocker"
        );
        this.maskClickListener = () => {
          this.disableModality();
        };
        this.mask.addEventListener("click", this.maskClickListener);
        document.body.appendChild(this.mask);
        DomHandler.addClass(document.body, "p-overflow-hidden");
      }
    },
    disableModality() {
      if (this.mask) {
        this.mask.removeEventListener("click", this.maskClickListener);
        this.maskClickListener = null;
        document.body.removeChild(this.mask);
        this.mask = null;
        const bodyChildren = document.body.children;
        let hasBlockerMasks;
        for (let i = 0; i < bodyChildren.length; i++) {
          const bodyChild = bodyChildren[i];
          if (
            DomHandler.hasClass(bodyChild, "p-datepicker-mask-scrollblocker")
          ) {
            hasBlockerMasks = true;
            break;
          }
        }
        if (!hasBlockerMasks) {
          DomHandler.removeClass(document.body, "p-overflow-hidden");
        }
        this.overlayVisible = false;
      }
    },
    updateCurrentMetaData() {
      const viewDate = this.viewDate;
      this.currentMonth = viewDate.getMonth();
      this.currentYear = viewDate.getFullYear();
      if (this.showTime || this.timeOnly) {
        this.updateCurrentTimeMeta(viewDate);
      }
    },
    isValidSelection(value) {
      let isValid = true;
      if (this.isSingleSelection()) {
        if (
          !this.isSelectable(
            value.getDate(),
            value.getMonth(),
            value.getFullYear(),
            false
          )
        ) {
          isValid = false;
        }
      } else if (
        value.every(v =>
          this.isSelectable(v.getDate(), v.getMonth(), v.getFullYear(), false)
        )
      ) {
        if (this.isRangeSelection()) {
          isValid = !!(value.length > 1 && value[1] > value[0]);
        }
      }
      return isValid;
    },
    parseValueFromString(text) {
      if (!text || text.trim().length === 0) {
        return null;
      }
      let value;
      if (this.isSingleSelection()) {
        value = this.parseDateTime(text);
      } else if (this.isMultipleSelection()) {
        const tokens = text.split(",");
        value = [];
        for (const token of tokens) {
          value.push(this.parseDateTime(token.trim()));
        }
      } else if (this.isRangeSelection()) {
        const tokens = text.split(" - ");
        value = [];
        for (let i = 0; i < tokens.length; i++) {
          value[i] = this.parseDateTime(tokens[i].trim());
        }
      }
      return value;
    },
    parseDateTime(text) {
      let date;
      const parts = text.split(" ");
      if (this.timeOnly) {
        date = new Date();
        this.populateTime(date, parts[0], parts[1]);
      } else {
        const dateFormat = this.datePattern;
        if (this.showTime) {
          date = this.parseDate(parts[0], dateFormat);
          this.populateTime(date, parts[1], parts[2]);
        } else {
          date = this.parseDate(text, dateFormat);
        }
      }
      return date;
    },
    populateTime(value, timeString, ampm) {
      if (this.hourFormat === "12" && !ampm) {
        // throw 'Invalid Time'
      }
      this.pm = ampm === "PM" || ampm === "pm";
      const time = this.parseTime(timeString);
      value.setHours(time.hour);
      value.setMinutes(time.minute);
      value.setSeconds(time.second);
    },
    parseTime(value) {
      const tokens = value.split(":");
      const validTokenLength = this.showSeconds ? 3 : 2;
      if (tokens.length !== validTokenLength) {
        // throw 'Invalid time'
      }
      let h = parseInt(tokens[0]);
      const m = parseInt(tokens[1]);
      const s = this.showSeconds ? parseInt(tokens[2]) : null;
      if (
        isNaN(h) ||
        isNaN(m) ||
        h > 23 ||
        m > 59 ||
        (this.hourFormat === "12" && h > 12) ||
        (this.showSeconds && (isNaN(s) || s > 59))
      ) {
        // throw 'Invalid time'
      } else {
        if (this.hourFormat === "12" && h !== 12 && this.pm) {
          h += 12;
        }
        return { hour: h, minute: m, second: s };
      }
    },
    parseDate(value, format) {
      if (format == null || value == null) {
        // throw 'Invalid arguments'
      }
      value = typeof value === "object" ? value.toString() : value + "";
      if (value === "") {
        return null;
      }
      let iFormat;
      let dim;
      let extra;
      let iValue = 0;
      const shortYearCutoff =
        typeof this.shortYearCutoff !== "string"
          ? this.shortYearCutoff
          : (new Date().getFullYear() % 100) +
            parseInt(this.shortYearCutoff, 10);
      let year = -1;
      let month = -1;
      let day = -1;
      let doy = -1;
      let literal = false;
      let date;
      const lookAhead = match => {
        const matches =
          iFormat + 1 < format.length && format.charAt(iFormat + 1) === match;
        if (matches) {
          iFormat++;
        }
        return matches;
      };
      const getNumber = match => {
        const isDoubled = lookAhead(match);
        const size =
          match === "@"
            ? 14
            : match === "!"
            ? 20
            : match === "y" && isDoubled
            ? 4
            : match === "o"
            ? 3
            : 2;
        const minSize = match === "y" ? size : 1;
        const digits = new RegExp("^\\d{" + minSize + "," + size + "}");
        const num = value.substring(iValue).match(digits);
        if (!num) {
          // throw 'Missing number at position ' + iValue
        }
        iValue += num[0].length;
        return parseInt(num[0], 10);
      };
      const getName = (match, shortNames, longNames) => {
        let index = -1;
        const arr = lookAhead(match) ? longNames : shortNames;
        const names = [];
        for (let i = 0; i < arr.length; i++) {
          names.push([i, arr[i]]);
        }
        names.sort((a, b) => {
          return -(a[1].length - b[1].length);
        });
        for (let i = 0; i < names.length; i++) {
          const name = names[i][1];
          if (
            value.substr(iValue, name.length).toLowerCase() ===
            name.toLowerCase()
          ) {
            index = names[i][0];
            iValue += name.length;
            break;
          }
        }
        if (index !== -1) {
          return index + 1;
        } else {
          // throw 'Unknown name at position ' + iValue
        }
      };
      const checkLiteral = () => {
        if (value.charAt(iValue) !== format.charAt(iFormat)) {
          // throw 'Unexpected literal at position ' + iValue
        }
        iValue++;
      };
      if (this.view === "month") {
        day = 1;
      }
      for (iFormat = 0; iFormat < format.length; iFormat++) {
        if (literal) {
          if (format.charAt(iFormat) === "'" && !lookAhead("'")) {
            literal = false;
          } else {
            checkLiteral();
          }
        } else {
          switch (format.charAt(iFormat)) {
            case "d":
              day = getNumber("d");
              break;
            case "D":
              getName(
                "D",
                this.localeData.dayNamesShort,
                this.localeData.dayNames
              );
              break;
            case "o":
              doy = getNumber("o");
              break;
            case "m":
              month = getNumber("m");
              break;
            case "M":
              month = getName(
                "M",
                this.localeData.monthNamesShort,
                this.localeData.monthNames
              );
              break;
            case "y":
              year = getNumber("y");
              break;
            case "@":
              date = new Date(getNumber("@"));
              year = date.getFullYear();
              month = date.getMonth() + 1;
              day = date.getDate();
              break;
            case "!":
              date = new Date((getNumber("!") - this.ticksTo1970) / 10000);
              year = date.getFullYear();
              month = date.getMonth() + 1;
              day = date.getDate();
              break;
            case "'":
              if (lookAhead("'")) {
                checkLiteral();
              } else {
                literal = true;
              }
              break;
            default:
              checkLiteral();
          }
        }
      }
      if (iValue < value.length) {
        extra = value.substr(iValue);
        if (!/^\s+/.test(extra)) {
          // throw 'Extra/unparsed characters found in date: ' + extra
        }
      }
      if (year === -1) {
        year = new Date().getFullYear();
      } else if (year < 100) {
        year +=
          new Date().getFullYear() -
          (new Date().getFullYear() % 100) +
          (year <= shortYearCutoff ? 0 : -100);
      }
      if (doy > -1) {
        month = 1;
        day = doy;
        do {
          dim = this.getDaysCountInMonth(year, month - 1);
          if (day <= dim) {
            break;
          }
          month++;
          day -= dim;
          // eslint-disable-next-line
        } while (true);
      }
      date = this.daylightSavingAdjust(new Date(year, month - 1, day));
      if (
        date.getFullYear() !== year ||
        date.getMonth() + 1 !== month ||
        date.getDate() !== day
      ) {
        // throw 'Invalid date' // E.g. 31/02/00
      }
      return date;
    },
    getWeekNumber(date) {
      const checkDate = new Date(date.getTime());
      checkDate.setDate(checkDate.getDate() + 4 - (checkDate.getDay() || 7));
      const time = checkDate.getTime();
      checkDate.setMonth(0);
      checkDate.setDate(1);
      return (
        Math.floor(Math.round((time - checkDate.getTime()) / 86400000) / 7) + 1
      );
    },
    onDateCellKeydown(event, date, groupIndex) {
      const cellContent = event.currentTarget;
      const cell = cellContent.parentElement;
      switch (event.which) {
        // down arrow
        case 40: {
          cellContent.tabIndex = "-1";
          const cellIndex = DomHandler.index(cell);
          const nextRow = cell.parentElement.nextElementSibling;
          if (nextRow) {
            const focusCell = nextRow.children[cellIndex].children[0];
            if (DomHandler.hasClass(focusCell, "p-disabled")) {
              this.navigationState = { backward: false };
              this.navForward(event);
            } else {
              nextRow.children[cellIndex].children[0].tabIndex = "0";
              nextRow.children[cellIndex].children[0].focus();
            }
          } else {
            this.navigationState = { backward: false };
            this.navForward(event);
          }
          event.preventDefault();
          break;
        }
        // up arrow
        case 38: {
          cellContent.tabIndex = "-1";
          const cellIndex = DomHandler.index(cell);
          const prevRow = cell.parentElement.previousElementSibling;
          if (prevRow) {
            const focusCell = prevRow.children[cellIndex].children[0];
            if (DomHandler.hasClass(focusCell, "p-disabled")) {
              this.navigationState = { backward: true };
              this.navBackward(event);
            } else {
              focusCell.tabIndex = "0";
              focusCell.focus();
            }
          } else {
            this.navigationState = { backward: true };
            this.navBackward(event);
          }
          event.preventDefault();
          break;
        }
        // left arrow
        case 37: {
          cellContent.tabIndex = "-1";
          const prevCell = cell.previousElementSibling;
          if (prevCell) {
            const focusCell = prevCell.children[0];
            if (DomHandler.hasClass(focusCell, "p-disabled")) {
              this.navigateToMonth(true, groupIndex);
            } else {
              focusCell.tabIndex = "0";
              focusCell.focus();
            }
          } else {
            this.navigateToMonth(true, groupIndex);
          }
          event.preventDefault();
          break;
        }
        // right arrow
        case 39: {
          cellContent.tabIndex = "-1";
          const nextCell = cell.nextElementSibling;
          if (nextCell) {
            const focusCell = nextCell.children[0];
            if (DomHandler.hasClass(focusCell, "p-disabled")) {
              this.navigateToMonth(false, groupIndex);
            } else {
              focusCell.tabIndex = "0";
              focusCell.focus();
            }
          } else {
            this.navigateToMonth(false, groupIndex);
          }
          event.preventDefault();
          break;
        }
        // enter
        case 13: {
          this.onDateSelect(event, date);
          event.preventDefault();
          break;
        }
        // escape
        case 27: {
          this.overlayVisible = false;
          event.preventDefault();
          break;
        }
        // tab
        case 9: {
          this.trapFocus(event);
          break;
        }
        default:
          // no op
          break;
      }
    },
    navigateToMonth(prev, groupIndex) {
      if (prev) {
        if (this.numberOfMonths === 1 || groupIndex === 0) {
          this.navigationState = { backward: true };
          this.navBackward(event);
        } else {
          const prevMonthContainer = this.$refs.overlay.children[
            groupIndex - 1
          ];
          const cells = DomHandler.find(
            prevMonthContainer,
            ".p-datepicker-calendar td span:not(.p-disabled)"
          );
          const focusCell = cells[cells.length - 1];
          focusCell.tabIndex = "0";
          focusCell.focus();
        }
      } else {
        if (
          this.numberOfMonths === 1 ||
          groupIndex === this.numberOfMonths - 1
        ) {
          this.navigationState = { backward: false };
          this.navForward(event);
        } else {
          const nextMonthContainer = this.$refs.overlay.children[
            groupIndex + 1
          ];
          const focusCell = DomHandler.findSingle(
            nextMonthContainer,
            ".p-datepicker-calendar td span:not(.p-disabled)"
          );
          focusCell.tabIndex = "0";
          focusCell.focus();
        }
      }
    },
    onMonthCellKeydown(event, index) {
      const cell = event.currentTarget;
      switch (event.which) {
        // arrows
        case 38:
        case 40: {
          cell.tabIndex = "-1";
          var cells = cell.parentElement.children;
          var cellIndex = DomHandler.index(cell);
          const nextCell =
            cells[event.which === 40 ? cellIndex + 3 : cellIndex - 3];
          if (nextCell) {
            nextCell.tabIndex = "0";
            nextCell.focus();
          }
          event.preventDefault();
          break;
        }
        // left arrow
        case 37: {
          cell.tabIndex = "-1";
          const prevCell = cell.previousElementSibling;
          if (prevCell) {
            prevCell.tabIndex = "0";
            prevCell.focus();
          }
          event.preventDefault();
          break;
        }
        // right arrow
        case 39: {
          cell.tabIndex = "-1";
          const nextCell = cell.nextElementSibling;
          if (nextCell) {
            nextCell.tabIndex = "0";
            nextCell.focus();
          }
          event.preventDefault();
          break;
        }
        // enter
        case 13: {
          this.onMonthSelect(event, index);
          event.preventDefault();
          break;
        }
        // escape
        case 27: {
          this.overlayVisible = false;
          event.preventDefault();
          break;
        }
        // tab
        case 9: {
          this.trapFocus(event);
          break;
        }
        default:
          // no op
          break;
      }
    },
    updateFocus() {
      let cell;
      if (this.navigationState) {
        if (this.navigationState.button) {
          this.initFocusableCell();
          if (this.navigationState.backward)
            DomHandler.findSingle(
              this.$refs.overlay,
              ".p-datepicker-prev"
            ).focus();
          else
            DomHandler.findSingle(
              this.$refs.overlay,
              ".p-datepicker-next"
            ).focus();
        } else {
          if (this.navigationState.backward) {
            const cells = DomHandler.find(
              this.$refs.overlay,
              ".p-datepicker-calendar td span:not(.p-disabled)"
            );
            cell = cells[cells.length - 1];
          } else {
            cell = DomHandler.findSingle(
              this.$refs.overlay,
              ".p-datepicker-calendar td span:not(.p-disabled)"
            );
          }
          if (cell) {
            cell.tabIndex = "0";
            cell.focus();
          }
        }
        this.navigationState = null;
      } else {
        this.initFocusableCell();
      }
    },
    initFocusableCell() {
      let cell;
      if (this.view === "month") {
        const cells = DomHandler.find(
          this.$refs.overlay,
          ".p-monthpicker .p-monthpicker-month"
        );
        const selectedCell = DomHandler.findSingle(
          this.$refs.overlay,
          ".p-monthpicker .p-monthpicker-month.p-highlight"
        );
        cells.forEach(cell => (cell.tabIndex = -1));
        cell = selectedCell || cells[0];
      } else {
        cell = DomHandler.findSingle(this.$refs.overlay, "span.p-highlight");
        if (!cell) {
          const todayCell = DomHandler.findSingle(
            this.$refs.overlay,
            "td.p-datepicker-today span:not(.p-disabled)"
          );
          if (todayCell) cell = todayCell;
          else
            cell = DomHandler.findSingle(
              this.$refs.overlay,
              ".p-datepicker-calendar td span:not(.p-disabled)"
            );
        }
      }
      if (cell) {
        cell.tabIndex = "0";
      }
    },
    trapFocus(event) {
      event.preventDefault();
      const focusableElements = DomHandler.getFocusableElements(
        this.$refs.overlay
      );
      if (focusableElements && focusableElements.length > 0) {
        if (!document.activeElement) {
          focusableElements[0].focus();
        } else {
          const focusedIndex = focusableElements.indexOf(
            document.activeElement
          );
          if (event.shiftKey) {
            if (focusedIndex === -1 || focusedIndex === 0)
              focusableElements[focusableElements.length - 1].focus();
            else focusableElements[focusedIndex - 1].focus();
          } else {
            if (
              focusedIndex === -1 ||
              focusedIndex === focusableElements.length - 1
            )
              focusableElements[0].focus();
            else focusableElements[focusedIndex + 1].focus();
          }
        }
      }
    },
    onContainerButtonKeydown(event) {
      switch (event.which) {
        // tab
        case 9:
          this.trapFocus(event);
          break;
        // escape
        case 27:
          this.overlayVisible = false;
          event.preventDefault();
          break;
        default:
          // Noop
          break;
      }
    }
  }
};
</script>

<style scoped>
@import "./css/calendar.css";
</style>