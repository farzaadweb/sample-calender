<script setup lang="ts">
import DATA from './data/data.sample.json'
import { ref, onMounted, watch } from 'vue'

const currentDate = ref<Date>(new Date())
const monthCalendar = ref<Date[]>([])
const calenderType = ref<'week' | 'month'>('week')
const periods = [
  '9,10',
  '10,11',
  '11,12',
  '12,13',
  '13,14',
  '14,15',
  '15,16',
  '16,17',
  '17,18',
  '18,19',
  '19,20',
  '20,21'
]
const apiResponse = ref<any>(DATA)

function calculateFirstDayOfMonth(currentDate: Date): Date {
  const firstDayOfMonth = new Date(currentDate)
  firstDayOfMonth.setDate(
    firstDayOfMonth.getDate() -
      (+firstDayOfMonth.toLocaleDateString('fa-IR-u-nu-latn', { day: '2-digit' }) - 1)
  )
  return firstDayOfMonth
}

function calculateDaysInMonth(monthNumber: number) {
  if (monthNumber >= 1 && monthNumber <= 6) {
    return 31
  } else if (monthNumber > 6 && monthNumber <= 11) {
    return 30
  } else if (monthNumber === 12) {
    return 29
  } else {
    alert('An issue occurred while calculating the month number.')
    return undefined
  }
}

function generateMonthCalendar(firstDayOfMonth: Date) {
  const convertedDate = new Date(firstDayOfMonth.toLocaleDateString('fa-IR-u-nu-latn'))
  const convertedMonth = convertedDate.getMonth() + 1
  const daysInMonth = ref<29 | 30 | 31 | undefined>(calculateDaysInMonth(convertedMonth))
  if (!daysInMonth.value) return
  const monthDays = ref<Date[]>([])
  const currentDate = ref<Date>(new Date(firstDayOfMonth))
  for (let i = 0; i < daysInMonth.value; i++) {
    if (i === 0) {
      monthDays.value.push(new Date(currentDate.value))
      continue
    } else {
      currentDate.value.setDate(currentDate.value.getDate() + 1)
      monthDays.value.push(new Date(currentDate.value))
    }
  }

  monthCalendar.value = monthDays.value
}

function divideMonthDays(monthDays: Date[]): (Date | 'Empty')[] {
  const weekday: string[] = ['شنبه', 'یکشنبه', 'دوشنبه', 'سه‌شنبه', 'چهارشنبه', 'پنجشنبه', 'جمعه']
  let output: (Date | 'Empty')[] = []
  let findeWeekDayLocationStatus: boolean = true
  for (const [index, day] of monthDays.entries()) {
    if (findeWeekDayLocationStatus && index === 0) {
      for (const wd of weekday) {
        if (wd === day.toLocaleDateString('fa-IR', { weekday: 'long' })) {
          output.push(day)
          findeWeekDayLocationStatus = false
          break
        } else {
          output.push('Empty')
          continue
        }
      }
    } else if (index === monthDays.length - 1) {
      const minIndex = weekday.findIndex(
        (e) => e === day.toLocaleDateString('fa-IR', { weekday: 'long' })
      )
      if (minIndex === 6) {
        output.push(day)
        continue
      } else {
        output.push(day)
        for (let i = minIndex + 1; i < 7; i++) {
          output.push('Empty')
        }
        continue
      }
    } else {
      output.push(day)
    }
  }
  return output
}

function paginateCalender(
  monthDays: (Date | 'Empty')[],
  target: Date = currentDate.value
): (Date | 'Empty')[] {
  const dividedDays = ref<(Date | 'Empty')[][]>([])
  for (let i = 0; i < monthDays.length; i += 7) {
    dividedDays.value.push(monthDays.slice(i, i + 7))
  }
  return dividedDays.value.filter((d: (Date | 'Empty')[]) => {
    if (
      d.some((date) => {
        return `${date}`.startsWith(`${target}`)
      })
    )
      return d
  })[0]
}

function nextWeek(date: Date = currentDate.value): void {
  const currDate = new Date(date)
  currDate.setDate(currDate.getDate() + 7)
  currentDate.value = new Date(currDate)
}

function goToday(date: Date = currentDate.value): void {
  currentDate.value = new Date()
}

function pervWeek(date: Date = currentDate.value): void {
  const currDate = new Date(date)
  currDate.setDate(currDate.getDate() - 7)
  currentDate.value = new Date(currDate)
}

function generateKey(date: Date) {
  return `${new Date(date).getFullYear()}${new Date(date).getMonth()}${new Date(date).getDate()}`
}

watch(currentDate, (n: Date) => {
  generateMonthCalendar(calculateFirstDayOfMonth(n))
})

onMounted(() => {
  generateMonthCalendar(calculateFirstDayOfMonth(currentDate.value))
})
</script>

<template>
  <div>
    <div class="bg-neutral-100">
      <div class="container mx-auto flex items-center py-8">
        <button
          class="h-12 px-8 text-sm text-neutral-50 bg-blue-500 hover:bg-blue-600 font-medium transition-all"
          @click="() => pervWeek()"
        >
          قبلی
        </button>
        <button
          class="h-12 px-8 text-sm text-neutral-50 bg-yellow-500 hover:bg-yellow-600 font-medium transition-all"
          @click="() => goToday()"
        >
          امروز
        </button>
        <button
          class="h-12 px-8 text-sm text-neutral-50 bg-blue-500 hover:bg-blue-600 font-medium transition-all"
          @click="() => nextWeek()"
        >
          بعدی
        </button>
        <select v-model="calenderType" class="h-12 px-12 outline-none border-y border-l">
          <option value="week">هفتگی</option>
          <option value="month">ماهانه</option>
        </select>
      </div>
    </div>
    <div v-if="monthCalendar.length" class="w-full container mx-auto py-20">
      <div class="grid grid-cols-7 gap-8" v-if="calenderType === 'month'">
        <div
          v-for="(m, i) in divideMonthDays(monthCalendar)"
          :key="i"
          class="w-full bg-neutral-100 rounded-xl flex items-center justify-center h-24 text-sm"
        >
          <span v-if="m === 'Empty'"></span>
          <span v-else>{{
            m.toLocaleDateString('fa-IR', { day: 'numeric', weekday: 'long', month: 'long' })
          }}</span>
        </div>
      </div>
      <table class="table-fixed w-full rounded-xl overflow-hidden shadow-xl" v-else>
        <th>
          <tr
            class="h-24 bg-neutral-100 border-b border-neutral-200 flex items-center justify-center px-8"
          >
            زمان بندی
          </tr>
          <tr
            v-for="(period, i) in periods"
            :key="i"
            class="h-24 bg-neutral-100 border-b border-neutral-200 flex items-center justify-center px-8"
          >
            {{
              period.split(',')[1]
            }}
            -
            {{
              period.split(',')[0]
            }}
          </tr>
        </th>
        <td v-for="(m, i) in paginateCalender(divideMonthDays(monthCalendar))" :key="i">
          <tr
            class="h-24 bg-neutral-100 border-b border-neutral-200 flex items-center justify-center px-8 text-sm"
          >
            <span v-if="m !== 'Empty'">
              {{ m.toLocaleString('fa-IR', { day: 'numeric', weekday: 'long' }) }}</span
            >
            <span v-else>متعلق به ماه قبل</span>
          </tr>
          <tr
            v-for="(period, ii) in periods"
            :key="ii"
            class="h-24 bg-neutral-100 border-b border-neutral-200 flex items-center justify-center px-8"
          >
            <div v-if="m !== 'Empty'">
              <div
                v-if="apiResponse[generateKey(m)] && apiResponse[generateKey(m)][period]"
                class="text-sm w-full"
              >
                <div>
                  <span
                    v-if="apiResponse[generateKey(m)][period].open_rounds"
                    class="w-full bg-green-200"
                  >
                    {{ apiResponse[generateKey(m)][period].open_rounds }} نوبت خالی</span
                  >
                  <span v-else class="w-full bg-red-200">پر شده</span>
                </div>
                <span
                  v-if="apiResponse[generateKey(m)][period].suspended_round"
                  class="w-full bg-yellow-200"
                  >{{ apiResponse[generateKey(m)][period].suspended_round }} تعویقی</span
                >
              </div>
              <div v-else class="text-xs">تنظیماتی صورت نگرفته</div>
            </div>
            <span v-else>خالی</span>
          </tr>
        </td>
      </table>
    </div>
  </div>
</template>
