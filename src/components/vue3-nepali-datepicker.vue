<template>
  <div class="datepicker" @click.stop>
    <input
      type="text"
      :class="this.classValue"
      v-model="formatedValue"
      @focus="show()"
      :placeholder="this.placeholder"
    />
    <div v-if="visible" :class="['calendar', { show: visible }]">
      <div class="calendar__header">
        <div class="calendar__year">{{ formatedYear }}</div>
        <div class="calendar__date">{{ formatedDate }}</div>
      </div>
      <div class="calendar__body">
        <!-- Month Container -->
        <div class="calendar__month">
          <button class="calendar__month__prev" @click="prev()">
            <b>&lt;</b>
          </button>
          <span>{{ formatedYearOrMonth }}</span>
          <select
            @change="monthSelectChange"
            v-model="monthValue"
            size="mini"
            v-if="this.monthSelect"
            style=""
          >
            <option
              style="text-align-last: center"
              v-for="(month, index) in getMonthsList"
              :key="month"
              :label="month"
              :value="index"
            ></option>
          </select>
          <select
            @change="yearSelectChange"
            v-model="yearValue"
            size="mini"
            v-if="this.yearSelect"
            style="margin-left: 5px"
          >
            <option
              style="text-align-last: center"
              v-for="i in numberofYears"
              :key="i"
              :value="startingYear + (i - 1)"
              :label="
                formatNepali
                  ? getNepaliDateWithYear(startingYear + (i - 1)).substr(0, 4)
                  : startingYear + (i - 1)
              "
            ></option>
          </select>
          <button icon="el-icon-arrow-right" @click="next()"><b>></b></button>
        </div>
        <!-- Week Day Container -->
        <div style="padding: 3px">
          <div class="calendar__weeks">
            <div
              style="font-weight: bold"
              class="calendar__weekday"
              v-for="(weekday, w) in weekdays"
              :key="w"
            >
              {{ weekday }}
            </div>
          </div>
          <!-- Day of Month Container -->
          <div class="calendar__days">
            <div
              class="calendar__day_spacer"
              :style="{ gridColumn: `span ${startweek}` }"
            ></div>
            <div
              :class="[
                'calendar__day',
                { selected: active(day) },
                { today: checkToday(day) },
              ]"
              v-for="(day, d) in days"
              :key="d"
              @click="select(day)"
            >
              {{ formatNepali ? convertToNepali(day).substr(8, 10) : day.day }}
            </div>
          </div>
        </div>
      </div>
      <div class="calendar__footer">
        <button @click="today()">{{ formatedTodayText }}</button>
      </div>
    </div>
  </div>
</template>

<script>
import NepaliDate from "nepali-date/cjs/NepaliDate";
import {
  YEAR_DATES,
  ENGLISH_WEEK,
  NEPALI_WEEK,
  NEPALI_MONTH,
  ENGLISH_NEPALI_MONTH,
} from "./constants.js";
export default {
  name: "VNepalidatepicker", // vue component name
  props: {
    value: { type: String, default: "" },
    format: { type: String, default: "YYYY-MM-DD" },
    calenderType: { type: String, default: "English" },
    yearSelect: { type: Boolean, default: true },
    monthSelect: { type: Boolean, default: true },
    classValue: { type: String, default: "" },
    placeholder: { type: String, default: "" },
  },
  model: {
    event: "change",
  },
  data() {
    return {
      date: this.value == "" ? new NepaliDate() : new NepaliDate(this.value),
      formatedValue: this.value,
      visible: false,
      startingYear: 2001,
      numberofYears: 87,
      formatNepali: this.calenderType == "Nepali" ? true : false,
      endDay: null,
      yearValue:
        this.value == ""
          ? new NepaliDate().getYear()
          : new NepaliDate(this.value).getYear(),
      monthValue:
        this.value == ""
          ? new NepaliDate().getMonth()
          : new NepaliDate(this.value).getMonth(),
      startMonthValue: null,
      currentDateValue: undefined,
    };
  },
  computed: {
    getMonthsList() {
      return this.formatNepali ? NEPALI_MONTH : ENGLISH_NEPALI_MONTH;
    },
    year() {
      return this.date.year;
    },
    weekdays() {
      return this.formatNepali ? NEPALI_WEEK : ENGLISH_WEEK;
    },
    days() {
      YEAR_DATES.forEach((yearData) => {
        if (yearData.year == this.date.year) {
          yearData.value.forEach((data, index) => {
            // compare monthValue selected to index of yearData value
            if (index == this.date.month) {
              this.endDay = data; // eslint-disable-line
            }
          });
        }
      });
      return Array(this.endDay)
        .fill()
        .map((_, idx) => new NepaliDate(this.year, this.date.month, idx + 1));
    },
    startweek() {
      let currentDateValue = new NepaliDate(this.yearValue, this.monthValue, 1);

      ENGLISH_WEEK.forEach((data, index) => {
        if (currentDateValue.format("DDD") == "Sun") {
          this.startMonthValue = 7; // eslint-disable-line
        } else if (currentDateValue.format("DDD") == data) {
          this.startMonthValue = index; // eslint-disable-line
        }
      });
      return this.startMonthValue;
    },
    formatedYearOrMonth() {
      if (this.monthSelect == false && this.yearSelect == false) {
        return this.formatNepali
          ? this.date.format("mmmm yyyy")
          : this.date.format("MMMM YYYY");
      }

      if (this.monthSelect == false) {
        return this.formatNepali
          ? this.date.format("mmmm")
          : this.date.format("MMMM");
      }

      if (this.yearSelect == false) {
        return this.formatNepali
          ? this.date.format("yyyy")
          : this.date.format("YYYY");
      }
      return "";
    },
    formatedYear() {
      return this.formatNepali
        ? this.date.format("yyyy")
        : this.date.format("YYYY");
    },
    formatedDate() {
      return this.formatNepali
        ? this.date.format("dddd, dd mmmm")
        : this.date.format("DDDD, DD MMMM");
    },
    formatedTodayText() {
      return this.formatNepali ? "आज" : "Today";
    },
  },
  methods: {
    convertToNepali(date) {
      return new NepaliDate(date).format("yyyy-mm-d");
    },
    getNepaliDateWithYear(year) {
      return new NepaliDate(year, 0, 1).format("yyyy-mm-d");
    },
    active(date) {
      return this.date.getTime() === date.getTime();
    },
    checkToday(date) {
      let today = new NepaliDate();
      return (
        date.day == today.day &&
        date.year == today.year &&
        date.month == today.month
      );
    },
    next() {
      let _month = this.date.month + 1;
      let _year = this.date.year;
      if (_month > 11) {
        _year++;
        _month = 0;
      }
      this.setMonthAndYear(_month, _year);
      this.date = new NepaliDate(_year, _month, 1);
    },
    prev() {
      let _month = this.date.month - 1;
      let _year = this.date.year;
      if (_month < 0) {
        _year--;
        _month = 11;
      }
      this.setMonthAndYear(_month, _year);
      this.date = new NepaliDate(_year, _month, 1);
    },
    monthSelectChange() {
      this.date.setMonth(this.monthValue);
    },
    yearSelectChange() {
      this.date.setYear(this.yearValue);
    },
    select(date) {
      this.date = date;
      this.formatedValue = this.date.format(this.format);
      this.$emit("change", this.formatedValue);
      this.hide();
    },
    show() {
      this.visible = true;
      setTimeout(() => document.addEventListener("click", this.hide), 200);
    },
    hide() {
      this.visible = false;
      document.removeEventListener("click", this.hide);
    },
    today() {
      this.formatedValue = new NepaliDate().format(this.format);
      this.date = new NepaliDate();
      this.setMonthAndYear(this.date.getMonth(), this.date.getYear());
      this.hide();
    },
    setMonthAndYear(month, year) {
      this.monthValue = month;
      this.yearValue = year;
    },
  },
};
</script>

<style scoped>
/* Importing the Manrope font */
@import url("https://fonts.googleapis.com/css2?family=Manrope:wght@300;400;500;700&display=swap");

* {
  margin: 0;
  box-sizing: border-box;
  font-family: "Manrope", sans-serif;
}
.datepicker {
  position: relative;
  border-radius: 0.4em;
}
.datepicker button {
  outline: none;
  border: 0;
  background: transparent;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}
.calendar {
  z-index: 9;
  position: absolute;
  width: 260px;
  top: 100%;
  -webkit-box-shadow: 0px 0px 16px 8px rgba(0, 0, 0, 0.11);
  -moz-box-shadow: 0px 0px 16px 8px rgba(0, 0, 0, 0.11);
  box-shadow: 0px 0px 16px 8px rgba(0, 0, 0, 0.11);
  background: #fff;
  visibility: hidden;
  opacity: 0;
  border-radius: 0.4em;
  overflow: hidden;
}
.calendar.show {
  visibility: visible;
  opacity: 1;
}
.calendar__header {
  padding: 15px 10px;
  background: #023047;
  color: #fff;
}
.calendar__year {
  opacity: 0.6;
  font-size: 1rem;
  font-weight: 700;
  line-height: 1.2rem;
}
.calendar__date {
  font-size: 1.2rem;
  line-height: 1.5rem;
  font-weight: 500;
}
.calendar__month {
  padding: 5px 3px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.calendar__month select {
  height: 28px;
  width: 100px;
  background-color: #aacfe2;
  border: none;
  border-radius: 0.2em;
  text-align-last: center;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  outline: none;
  cursor: pointer;
}
.calendar__month button {
  width: 25px;
  margin-right: 4px;
  height: 28px;
  margin-left: 4px;
  border-radius: 0.2em;
  color: #023047;
  text-align: center;
  background: #aacfe2;
}
.calendar__month button:hover {
  background: #023047;
  color: #aacfe2;
}

.calendar__weeks,
.calendar__days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}
.calendar__days {
  gap: 4px;
}
.calendar__day,
.calendar__weekday {
  text-align: center;
  font-size: 12px;
  color: #023047;
}
.calendar__weekday {
  opacity: 0.8;
  font-weight: 300;
}
.calendar__day {
  width: 32px;
  height: 32px;
  line-height: 32px;
  font-weight: 300;
  color: #023047;
  font-weight: bold;
  cursor: pointer;
  border-radius: 2px;
  background: #d7f1fe;
}
.calendar__day.selected {
  background: #023047;
  color: #fff;
}
.calendar__day.today {
  background: #fb8500;
  color: #fff;
}
.calendar__day:hover {
  background: #023047;
  color: #fff;
  opacity: 0.8;
}
.calendar__footer {
  text-align: right;
  margin-top: 2px;
}
.calendar__footer button {
  width: 100%;
  padding: 8px 10px;
  text-transform: uppercase;
  font-weight: bold;
  color: #fff;
  background-color: #023047;
  opacity: 0.9;
}
.calendar__footer button:hover {
  opacity: 1;
}
</style>
