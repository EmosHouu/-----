<template>

<v-container>
 <v-row style="justify-content: center; margin-top: 10px;">
    <v-col cols="12" md="6" style="text-align: center;">
      <v-progress-circular
      :rotate="360"
      :size="300"
      :width="40"
      :model-value="currentSec"
      color="#AED581"

    >
    <v-row  justify="center">
            <v-col>
              <h1 :style="{ color: '#AED581', fontSize: '26px' }">{{currentText}}</h1>
              <h1 :style="{ color: 'rgb(125, 63, 36)', fontSize: '20px' }">{{currentTime}}</h1>

              <v-btn variant="text" icon="mdi-play" :disabled="status === STATUS.COUNTING || (currentItem.length === 0 && items.length === 0)" @click="startTimer"></v-btn>
      <v-btn variant="text" icon="mdi-pause" :disabled="status !== STATUS.COUNTING" @click="pauseTimer"></v-btn>
      <v-btn variant="text" icon="mdi-skip-next" :disabled="currentItem.length === 0" @click="finishTimer"></v-btn>
            </v-col>
          </v-row>
    </v-progress-circular>

      <iframe style="border-radius:12px" src="https://open.spotify.com/embed/playlist/37i9dQZF1DX32smTEhlmrg?utm_source=generator" width="90%" height="180" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
<!-- 卡片 -->

      <!-- 輪播 -->
      <!-- <v-carousel hide-delimiters>
  <v-carousel-item
    src="https://cdn.vuetifyjs.com/images/cards/docks.jpg"
    cover
  ></v-carousel-item>

  <v-carousel-item
    src="https://cdn.vuetifyjs.com/images/cards/hotel.jpg"
    cover
  ></v-carousel-item>

  <v-carousel-item
    src="https://cdn.vuetifyjs.com/images/cards/sunshine.jpg"
    cover
  ></v-carousel-item>
</v-carousel> -->
      <!-- 輪播 -->

      <!-- <v-card
      width="500"
      title="{{currentText}}"
      subtitle="{{currentTime}}"
      text="This is content"
    ></v-card> -->

    </v-col>
    <!-- <v-col cols="4">
      <v-btn variant="text" icon="mdi-play" :disabled="status === STATUS.COUNTING || (currentItem.length === 0 && items.length === 0)" @click="startTimer"></v-btn>
      <v-btn variant="text" icon="mdi-pause" :disabled="status !== STATUS.COUNTING" @click="pauseTimer"></v-btn>
      <v-btn variant="text" icon="mdi-skip-next" :disabled="currentItem.length === 0"></v-btn>
    </v-col> -->
    <v-col cols="12" md="6" style="text-align: center; margin-top: 30px;">
      <v-date-picker
      color="light-green-lighten-2"
      style="margin: 0 auto;"
    ></v-date-picker>
    </v-col>

  </v-row>
</v-container>

</template>

<style scoped>
.v-progress-circular {
  margin: 1rem;
  animation: changeColor 30s infinite linear;
}

@keyframes changeColor {
  0% {
    color: #AED581;
  }
  50% {
    color: #609425; /* 中途切換的顏色 */
  }
  100% {
    color: #AED581;
  }
}
</style>

<script setup>
import { useListStore } from '@/store/list'
import { useSettingsStore } from '@/store/settings'
import { storeToRefs } from 'pinia'
import { ref, computed } from 'vue'

const list = useListStore()
const { currentItem, items, timeleft } = storeToRefs(list)
const { setCurrentItem, countdown, setFinishedItem } = list

const settings = useSettingsStore()
const { selectedAlarmFile } = storeToRefs(settings)

const STATUS = {
  STOP: 0,
  COUNTING: 1,
  PAUSE: 2
}

const status = ref(STATUS.STOP)
const value = ref(0)
const initialTime = ref(0)
let timer = 0
let pausedValue = 0

const startTimer = () => {
  if (status.value === STATUS.STOP && items.value.length > 0) {
    setCurrentItem()
  }

  status.value = STATUS.COUNTING
  value.value = pausedValue || 0
  initialTime.value = timeleft.value

  // 清除之前的進度條計時器
  clearInterval(timer)

  timer = setInterval(() => {
    countdown()
    if (timeleft.value === 0) {
      finishTimer()
    } else {
      value.value = ((initialTime.value - timeleft.value) * 100) / initialTime.value
    }
  }, 1000)
}

const pauseTimer = () => {
  status.value = STATUS.PAUSE
  clearInterval(timer)
  pausedValue = value.value
}

const finishTimer = () => {
  clearInterval(timer)
  status.value = STATUS.STOP

  const audio = new Audio()
  audio.src = selectedAlarmFile.value
  audio.play()

  setFinishedItem()

  if (items.value.length > 0) {
    startTimer()
  }
}

const currentText = computed(() => {
  if (currentItem.value.length > 0) {
    return currentItem.value
  } else if (items.value.length > 0) {
    return '點擊開始'
  } else {
    return '沒有事項'
  }
})

const currentTime = computed(() => {
  const m = Math.floor(timeleft.value / 60).toString().padStart(2, '0')
  const s = (timeleft.value % 60).toString().padStart(2, '0')
  return m + ':' + s
})

const currentSec = computed(() => {
  let sec = (timeleft.value % 60)
  sec = (sec / 60) * 100
  return sec
})

</script>

<!-- <script>
export default {
  data () {
    return {
      interval: {},
      value: 0
    }
  },
  beforeUnmount () {
    clearInterval(this.interval)
  },
  mounted () {
    this.interval = setInterval(() => {
      if (this.value === 100) {
        return (this.value = 0)
      }
      this.value += 1
    }, 15000)
  }
}
</script> -->
