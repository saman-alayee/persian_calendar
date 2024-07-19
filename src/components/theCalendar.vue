<template>
  <div>

  <div class="persian-calendar">
    <div class="calendar-header">
      <button class="arrow-btn" @click="prevMonth"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16"
          fill="currentColor" class="bi bi-chevron-right" viewBox="0 0 16 16">
          <path fill-rule="evenodd"
            d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708" />
        </svg></button>
      <select class="select-btn" v-model="currentYear">
        <option v-for="year in yearRange" :key="year" :value="year">{{ year }}</option>
      </select>
      <select v-model="currentMonth">
        <option v-for="(month, index) in months" :key="index" :value="index + 1">{{ month }}</option>
      </select>
      <button class="arrow-btn" @click="nextMonth">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
          class="bi bi-chevron-left" viewBox="0 0 16 16">
          <path fill-rule="evenodd"
            d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0" />
        </svg>
        </button>
    </div>
    <div class="calendar-body">
      <div class="day-names">
        <span v-for="day in daysOfWeek" :key="day">{{ day }}</span>
      </div>
      <div class="days">
        <span v-for="(day, index) in paddedDaysInMonth" :key="index"
          :class="{ 'today': isToday(day.shamsiDate), 'disabled': !day.isCurrentMonth, 'selected': isSelected(day.shamsiDate) }"
          @click="selectDate(day)">
          {{ day.date }}
        </span>
      </div>
    </div>
    <div v-if="selectedDate" class="selected-date">
      Selected Date: {{ selectedDateDisplay }}
    </div>
  </div>  </div>

</template>


<script>
import jalaali from 'jalaali-js';

export default {
  data() {
    const today = new Date();
    const shamsiToday = this.toShamsi(today.getFullYear(), today.getMonth() + 1, today.getDate());
    return {
      currentYear: shamsiToday.year,
      currentMonth: shamsiToday.month,
      selectedDate: null,
      selectedDateDisplay: '',
      daysOfWeek: ['شنبه', 'یکشنبه', 'دوشنبه', 'سه شنبه', 'چهاشنبه', 'پنج شنبه', 'جمعه'], // Corrected order: Saturday to Friday
      months: [
        "فروردین", "اردیبهشت", "خرداد", "تیر", "مرداد", "شهریور",
        "مهر", "آبان", "آذر", "دی", "بهمن", "اسفند"
      ],
    };
  },
  computed: {
    daysInMonth() {
      const numberOfDays = this.getDaysInMonth(this.currentYear, this.currentMonth);
      const firstDayOfMonth = this.getFirstDayOfMonth(this.currentYear, this.currentMonth);
      let days = [];

      for (let i = 1; i <= numberOfDays; i++) {
        const shamsiDate = { year: this.currentYear, month: this.currentMonth, day: i };
        days.push({
          date: i,
          shamsiDate: shamsiDate,
          isCurrentMonth: true
        });
      }

      // اضافه کردن روزهای خالی برای روزهای قبل از شروع ماه
      const paddedDays = Array(firstDayOfMonth).fill({ date: '', shamsiDate: null, isCurrentMonth: false }).concat(days);

      // اضافه کردن روزهای خالی برای پر کردن تقویم
      const totalCells = 42; // 6 هفته × 7 روز
      while (paddedDays.length < totalCells) {
        paddedDays.push({ date: '', shamsiDate: null, isCurrentMonth: false });
      }

      return paddedDays;
    },
    paddedDaysInMonth() {
      return this.daysInMonth;
    },
    yearRange() {
      return this.generateYearRange(1398, 1450);
    }
  },
  methods: {
    toShamsi(gy, gm, gd) {
      let g_d_m = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334];
      let jy = (gy <= 1600) ? 0 : 979;
      gy -= (gy <= 1600) ? 621 : 1600;
      let gy2 = (gm > 2) ? (gy + 1) : gy;
      let days = 365 * gy + parseInt((gy2 + 3) / 4) - parseInt((gy2 + 99) / 100) + parseInt((gy2 + 399) / 400) - 80 + gd + g_d_m[gm - 1];
      jy += 33 * parseInt(days / 12053);
      days %= 12053;
      jy += 4 * parseInt(days / 1461);
      days %= 1461;
      if (days > 365) {
        jy += parseInt((days - 1) / 365);
        days = (days - 1) % 365;
      }
      let jm = (days < 186) ? 1 + parseInt(days / 31) : 7 + parseInt((days - 186) / 30);
      let jd = 1 + ((days < 186) ? (days % 31) : ((days - 186) % 30));
      return { year: jy, month: jm, day: jd };
    },
    toGregorian(jy, jm, jd) {
      let gy;
      if (jy > 979) {
        gy = 1600;
        jy -= 979;
      } else {
        gy = 621;
      }

      const j_days_in_month = [31, 31, 31, 31, 31, 31, 30, 30, 30, 30, 30, 29];

      let days = 365 * jy + Math.floor(jy / 33) * 8 + Math.floor((jy % 33 + 3) / 4);
      for (let i = 0; i < jm - 1; ++i) {
        days += j_days_in_month[i];
      }
      days += jd - 1;

      let gy2 = gy + 400 * Math.floor(days / 146097);
      days %= 146097;

      if (days > 36524) {
        gy2 += 100 * Math.floor(--days / 36524);
        days %= 36524;
        if (days >= 365) {
          days++;
        }
      }

      gy2 += 4 * Math.floor(days / 1461);
      days %= 1461;

      if (days > 365) {
        gy2 += Math.floor((days - 1) / 365);
        days = (days - 1) % 365;
      }

      let gd = days + 1;
      const g_d_m = [31, gy2 % 4 == 0 && gy2 % 100 != 0 || gy2 % 400 == 0 ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
      let gm;
      for (gm = 0; gm < 12 && gd > g_d_m[gm]; gm++) {
        gd -= g_d_m[gm];
      }

      return { year: gy2, month: gm + 1, day: gd };
    },
    generateYearRange(start, end) {
      const years = [];
      for (let year = start; year <= end; year++) {
        years.push(year);
      }
      return years;
    },
    getDaysInMonth(year, month) {
      if (month >= 1 && month <= 6) {
        return 31;
      } else if (month >= 7 && month <= 11) {
        return 30;
      } else if (month === 12) {
        // اسفند در سال کبیسه 30 روزه است
        return this.isLeapYear(year) ? 30 : 29;
      }
      return 0; // برای ماه‌های نامعتبر
    },
    isLeapYear(year) {
      // سال شمسی کبیسه است اگر (سال - 474) % 2820 < 682 باشد
      const a = (year + 1 - 474) % 2820;
      return ((((a + 474) * 682) % 2816) < 682);
    },
    getFirstDayOfMonth(year, month) {
      // محاسبه روز هفته برای اولین روز ماه شمسی
      const firstDayOfMonth = jalaali.toGregorian(year + 1, month, 1);
      const dayOfWeek = new Date(firstDayOfMonth.gy, firstDayOfMonth.gm - 1, firstDayOfMonth.gd).getDay();
      // تبدیل روز هفته میلادی به روز هفته شمسی (شنبه = 0، یکشنبه = 1، و ...)
      return (dayOfWeek + 6) % 7; // تبدیل از میلادی به شمسی
    },
    isToday(shamsiDate) {
      if (!shamsiDate) return false;
      const today = new Date();
      const gregorianDate = this.toGregorian(shamsiDate.year, shamsiDate.month, shamsiDate.day);
      return gregorianDate.year === today.getFullYear() &&
        gregorianDate.month === today.getMonth() + 1 &&
        gregorianDate.day === today.getDate();
    },
    isSelected(shamsiDate) {
      if (!shamsiDate || !this.selectedDate) return false;
      const selectedGregorianDate = this.toGregorian(this.selectedDate.getFullYear(), this.selectedDate.getMonth() + 1, this.selectedDate.getDate());
      const dateToCompare = this.toGregorian(shamsiDate.year, shamsiDate.month, shamsiDate.day);
      return selectedGregorianDate.year === dateToCompare.year &&
        selectedGregorianDate.month === dateToCompare.month &&
        selectedGregorianDate.day === dateToCompare.day;
    },
    selectDate(day) {
      if (day.isCurrentMonth) {
        this.selectedDate = new Date(this.toGregorian(day.shamsiDate.year, day.shamsiDate.month, day.shamsiDate.day).year, this.toGregorian(day.shamsiDate.year, day.shamsiDate.month, day.shamsiDate.day).month - 1, this.toGregorian(day.shamsiDate.year, day.shamsiDate.month, day.shamsiDate.day).day);
        this.selectedDateDisplay = `${day.shamsiDate.year}/${day.shamsiDate.month}/${day.shamsiDate.day}`;
      }
    },
    prevMonth() {
      if (this.currentMonth === 1) {
        this.currentMonth = 12;
        this.currentYear--;
      } else {
        this.currentMonth--;
      }
    },
    nextMonth() {
      if (this.currentMonth === 12) {
        this.currentMonth = 1;
        this.currentYear++;
      } else {
        this.currentMonth++;
      }
    },
    checkLeapYear() {
      const yearsToCheck = [1402, 1403, 1404, 1405];
      yearsToCheck.forEach(year => {
        console.log(`${year} is leap year: ${this.isLeapYear(year)}`);
      });
    }
  },
  created() {
    const gregorianDate = this.toGregorian(1403, 4, 1); // تیر ماه به عنوان ماه 4
    console.log(gregorianDate); // بررسی نتیجه

  },
};
</script>




<style scoped>
.persian-calendar {
  font-family: Arial, sans-serif;
  width: 100%;
  max-width: 400px;
  border: 1px solid #ccc;
  border-radius: 10px;
  overflow: hidden;
  direction: rtl;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #f5f5f5;
  color: black;
  padding: 10px;
  border-bottom: 1px solid #ccc;
}
.selected-date {

  padding: 8px;
  color: white;
  background-color: #2196f3
}
.calendar-header select {
  margin: 0 5px;
  padding:8px;
  border: none;
  border-radius: 8px;
}

.calendar-body {
  padding: 10px;
}

.day-names,
.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  font-size: 14px;
}

.day-names span,
.days span {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
}

.days span {
  cursor: pointer;
  transition: background-color 0.3s ease;
  /* Smooth transition for background color change */
}


.days span.disabled {
  color: #ccc;
  pointer-events: none;
}

.days span.selected {
  background-color: #2196f3;
  /* Example color for selected date */
  color: white;
  border-radius: 50%;
}

.arrow-btn {
  padding: 8px;
  background-color: #2196f3;
  border: none;
  border-radius: 8px;
  color: white;
  cursor: pointer;
}

.days span:hover {
  border-radius: 50%;
  background-color: #2196f3;
  /* Example color on hover */
  color: white;
}
</style>
