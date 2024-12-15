<template>
  <div class="calendar">
    <div class="calendar__header">
      <button class="calendar__header__button" @click="changeMonth(-1)">
        <ChevronLeft />
      </button>
      <p class="calendar__header__month">{{ formattedMonth }}</p>
      <button class="calendar__header__button" @click="changeMonth(1)">
        <ChevronRight />
      </button>
    </div>

    <div class="calendar__weekdays">
      <p class="calendar__weekdays__day" v-for="day in weekdays" :key="day">
        {{ day }}
      </p>
    </div>

    <div class="calendar__month">
      <p
        class="calendar__month__day"
        :class="{
          'calendar__month__day--disabled': day.isDisabled,
          'calendar__month__day--today': day.isToday && !day.isDisabled,
          'calendar__month__day--selected': isDayInRange(day.date),
          'calendar__month__day--blocked': isBlocked(day.date),
          'calendar__month__day--allowed': isAllowed(day.date),
        }"
        v-for="day in days"
        :key="day.date"
        @click="handleClickDay(day)"
      >
        {{ day.dayNumber }}
      </p>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref, computed } from 'vue'
  import { ChevronRight, ChevronLeft } from 'lucide-vue-next'

  export default defineComponent({
    name: 'SingleCalendar',
    components: {
      ChevronRight,
      ChevronLeft,
    },
    props: {
      blockList: {
        type: Array,
        default: () => [],
      },
      allowedDateList: {
        type: Array,
        default: () => [],
      },
    },
    setup(props) {
      const currentDate = ref(new Date())
      const selectedRange = ref<{ start: Date | null; end: Date | null }>({
        start: null,
        end: null,
      })
      const weekdays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

      const formattedMonth = computed(() => {
        const options = { month: 'long', year: 'numeric' }
        return currentDate.value.toLocaleDateString('en-US', options)
      })

      const daysInMonth = (year, month) =>
        new Date(year, month + 1, 0).getDate()

      const generateDays = () => {
        const year = currentDate.value.getFullYear()
        const month = currentDate.value.getMonth()

        const firstDayOfMonth = new Date(year, month, 1).getDay()
        const totalDaysInCurrentMonth = daysInMonth(year, month)

        const prevMonthDays = daysInMonth(year, month - 1)
        const prevDaysToShow = (firstDayOfMonth + 6) % 7

        const days = []

        for (let i = prevDaysToShow; i > 0; i--) {
          days.push({
            dayNumber: prevMonthDays - i + 1,
            isDisabled: true,
            date: new Date(year, month - 1, prevMonthDays - i + 1),
          })
        }

        for (let i = 1; i <= totalDaysInCurrentMonth; i++) {
          const isToday =
            new Date().toDateString() ===
            new Date(year, month, i).toDateString()
          days.push({
            dayNumber: i,
            isDisabled: false,
            isToday,
            date: new Date(year, month, i),
          })
        }

        const nextDaysToShow = 42 - days.length
        for (let i = 1; i <= nextDaysToShow; i++) {
          days.push({
            dayNumber: i,
            isDisabled: true,
            date: new Date(year, month + 1, i),
            selected: false,
          })
        }

        return days
      }

      const days = computed(() => generateDays())

      const changeMonth = (offset) => {
        currentDate.value = new Date(
          currentDate.value.getFullYear(),
          currentDate.value.getMonth() + offset
        )

        // Adjust selectedRange when the month is changed
        if (selectedRange.value.start && selectedRange.value.end) {
          const newStart = new Date(selectedRange.value.start)
          const newEnd = new Date(selectedRange.value.end)

          // Ensure the start and end dates are within the new month
          if (newStart.getMonth() !== currentDate.value.getMonth()) {
            selectedRange.value.start = null
          }

          if (newEnd.getMonth() !== currentDate.value.getMonth()) {
            selectedRange.value.end = null
          }
        }
      }

      const handleClickDay = (day) => {
        if (selectedRange.value.start && selectedRange.value.end) {
          selectedRange.value.start = null
          selectedRange.value.end = null
        } else if (!selectedRange.value.start) {
          selectedRange.value.start = day.date
        } else if (
          !selectedRange.value.end &&
          day.date > selectedRange.value.start
        ) {
          selectedRange.value.end = day.date
        }
      }

      const isDayInRange = (date) => {
        if (!selectedRange.value.start || !selectedRange.value.end) return false
        return (
          date >= selectedRange.value.start && date <= selectedRange.value.end
        )
      }

      const isBlocked = (date) => {
        return (
          props.blockList.some(
            (blockedDate) => blockedDate.toDateString() === date.toDateString()
          ) || isWeekend(date)
        )
      }

      const isAllowed = (date) => {
        return props.allowedDateList.some(
          (allowedDate) => allowedDate.toDateString() === date.toDateString()
        )
      }

      const isWeekend = (date) => {
        const day = date.getDay()
        return day === 0 || day === 6
      }

      return {
        weekdays,
        formattedMonth,
        days,
        changeMonth,
        handleClickDay,
        selectedRange,
        isDayInRange,
        isBlocked,
        isAllowed,
      }
    },
  })
</script>

<style scoped lang="scss">
  .calendar {
    width: 552px;
    padding: 40px;
    border-radius: 28px;
    background: #fcfcfc;
    box-shadow: 5px 34px 80px 0px rgba(0, 0, 0, 0.15);

    &__header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding-bottom: 28px;
      border-bottom: 1px solid #a3a3a6;

      &__button {
        width: fit-content;
        font-size: 40px;
        background: #ffff;
        border: none;
        outline: none;
        cursor: pointer;
      }

      &__month {
        font-family: Poppins;
        text-align: center;
        font-size: 24px;
        font-weight: 500;
        color: #1b1b1b;
      }
    }

    &__weekdays {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      align-content: center;
      justify-content: center;
      height: 68px;

      &__day {
        text-align: center;
        font-family: Inter;
        font-size: 26px;
        font-weight: 500;
        color: #b3b3b3;
      }
    }

    &__month {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      align-content: center;
      justify-content: center;

      &__day {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 68px;
        height: 68px;
        font-family: Roboto;
        text-align: center;
        font-size: 28px;
        font-weight: 500;

        &--today {
          border-radius: 28px;
          border: 1px solid #657ff5;
          color: #657ff5;
        }

        &--disabled {
          color: #b3b3b3;
        }

        &--selected {
          background-color: #e0e0e0;
        }

        &--blocked {
          color: #b3b3b3;
          background-color: #f0f0f0;
        }

        &--allowed {
          background-color: #def7dd;
        }
      }
    }
  }
</style>
