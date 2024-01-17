<style scoped lang="scss">
.font {
  font-family: Arial, Helvetica, sans-serif, monospace;
}
</style>

<template>
  <div flex="~ col" items-center justify-center class="font">
    <div>
      <img v-show="!isStart" src="@/assets/images/ok.jpg" h-50 />
      <img v-show="isStart" src="@/assets/images/loading.gif" h-50 />
    </div>

    <div mt-2 text="16 primary" font-bold>
      <div v-show="first">点击开始</div>
      <div v-show="!first && !isStart">
        <span>本次用时 </span>
      </div>
      <div v-show="!first">
        <span v-show="time.minutes !== '00'">{{ time.minutes }} 分</span>
        <span ml-3>{{ time.seconds }} 秒</span>
      </div>
    </div>

    <div mt-10>
      <button w-30 h-16 text-6 rounded-full b-none @click="buttonClick">
        {{ isStart ? '回啦！' : '开始' }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import moment from '@/utils/moment'

import type { Moment } from 'moment'

const startTime = ref(moment())

const first = ref(false)
const isStart = ref(false)

const time = reactive({
  minutes: '',
  seconds: ''
})

let timer: NodeJS.Timeout | null = null

let times: {
  start: string
  end: string
}

onMounted(() => {
  times = getTime()

  // 首次开始
  if (!times.start && !times.end) {
    first.value = true
    time.minutes = '00'
    time.seconds = '00'
    return
  }

  if (times.start && !times.end) {
    isStart.value = true
    startTime.value = moment(times.start)

    setTime(startTime.value)
    startTimer()
  }

  if (times.end) {
    setTime(moment(times.start), moment(times.end))
  }
})

const startTimer = () => {
  if (timer !== null) {
    clearInterval(timer)
    time.minutes = '00'
    time.seconds = '00'
  }

  timer = setInterval(() => {
    setTime(startTime.value)
  }, 1000)
}

const setTime = (start: Moment, end?: Moment) => {
  if (!end) {
    end = moment()
  }

  time.minutes = `0${Math.abs(start.diff(end, 'minutes'))}`.slice(-2)
  time.seconds = `0${Math.abs(start.diff(end, 'seconds') % 60)}`.slice(-2)

  console.log('time', time.minutes, time.seconds)
}

const buttonClick = () => {
  first.value = false

  if (isStart.value) {
    clearInterval(timer!)

    const end = moment()

    location.hash = encodeURIComponent(
      `${times.start}~${end.format('YYYY-MM-DD HH:mm:ss')}`
    )

    setTime(moment(times.start), end)

    isStart.value = false
    return
  }

  // 再次开始
  isStart.value = true
  startTime.value = moment()
  times.start = moment().format('YYYY-MM-DD HH:mm:ss')

  location.hash = encodeURIComponent(
    startTime.value.format('YYYY-MM-DD HH:mm:ss')
  )
  startTimer()
}

const getTime = () => {
  const hash = decodeURIComponent(location.hash.replace('#', ''))

  const [start, end] = hash.split('~')

  return {
    start,
    end
  }
}
</script>
