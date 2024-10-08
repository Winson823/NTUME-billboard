<script setup lang="ts">
import 'swiper/css'

import Swiper from 'swiper'
import { Autoplay } from 'swiper/modules'
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

import RoomBlock from '@/pages/index/component/RoomBlock.vue'
import { Order, Room } from '@/pages/index/composables/type'
const dataTimeInterval = ref(1800000) //30mins

const roomData = ref<Room[]>([])
let intervalID = null as number | null

import { Method, useFetchData } from '@/composables/useFetchData'

// 初始化應用
import { useInitApp } from './composables/useInitApp'
const { initApp } = useInitApp()

//亂數
const generateRandomString = () => {
  return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
}

//API 請求
async function initData() {
  try {
    await getSpaceData()
    return true //防止沒拿到資料卡住
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}
const getSpaceData = async () => {
  const {
    result: [res],
  } = await useFetchData({
    url: './ntume/get_ntu_board_info',
    method: Method.GET,
  })
  const rawRoomData = Array.isArray(res?.data) ? res.data : [] //檢查 res?.data 是否是一個數組。true: res.data / False []
  // 添加或更新一個唯一的亂碼 ID
  // const updatedRoomData = rawRoomData.map((room) => ({
  //   ...room,
  //   field: `${room.field}-${generateRandomString()}`,
  // }))
  // roomData.value = updatedRoomData
  roomData.value = rawRoomData
  console.log(roomData.value)
}

onMounted(async () => {
  await initData()

  // 初始化 Swiper
  new Swiper('.swiper', {
    modules: [Autoplay],
    loop: true, // 循環輪播
    autoplay: {
      delay: 2000, // 自動輪播延遲
    },
  })

  //setInterval在window下執行其值為number
  intervalID = window.setInterval(() => {
    getSpaceData()
  }, dataTimeInterval.value)
})

// 清除定時器，避免內存洩漏
onBeforeUnmount(() => {
  if (intervalID) {
    clearInterval(intervalID)
  }
})

// 取得當前時間字串
const getCurrentTimeString = () => {
  const currentTime = new Date()
  const hours = String(currentTime.getHours()).padStart(2, '0')
  const minutes = String(currentTime.getMinutes()).padStart(2, '0')
  return `${hours}:${minutes}`
}

const currentTimeString = getCurrentTimeString()
initApp()

const transformedRoomData = computed(() =>
  roomData.value.map((room) => ({
    field: room.field,
    order: room.order.map((event) => ({
      time: event.time,
      name: event.name,
    })),
  })),
)

// 分頁 transformedRoomData，每頁最多 21 個
const paginatedRoomData = computed(() => {
  const chunkSize = 21
  const pages = []

  for (let i = 0; i < transformedRoomData.value.length; i += chunkSize) {
    pages.push(transformedRoomData.value.slice(i, i + chunkSize))
  }

  return pages
})
</script>

<template>
  <div class="h-[full]">
    <!-- 房間區塊 -->
    <div class="roomInfo flex w-full justify-center bg-[url('@/assets/boardBg.png')] bg-cover p-4">
      <!-- 判斷 roomData 是否超過 21，使用 Swiper -->
      <div v-if="paginatedRoomData.length > 1" class="swiper">
        <div class="swiper-wrapper">
          <!-- 使用 v-for 迭代每個分頁 -->
          <div v-for="(page, pageIndex) in paginatedRoomData" :key="pageIndex" class="swiper-slide">
            <div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
              <!-- 使用 v-for 迭代每一頁中的 room -->
              <div
                v-for="(room, index) in page.filter(
                  (room) =>
                    room.order.filter((event) => event.time.split('-')[1] >= currentTimeString)
                      .length > 0,
                )"
                :key="index"
                class="h-[154px] w-[328px] rounded-[15px] border-[3px] border-[rgb(204,204,204,0.7)] bg-[rgb(2,4,20,0.6)] p-4"
                :class="{
                  '!border-[#03B0EC]':
                    room.order[0]?.time.split('-')[0] <= currentTimeString &&
                    room.order[0]?.time.split('-')[1] >= currentTimeString,
                }"
              >
                <!-- 場域名稱 -->
                <div class="location mb-6 text-[20px]">
                  {{ room.field }}
                </div>
                <!-- 活動列表 -->
                <div class="events">
                  <div
                    v-for="(event, eIndex) in room.order
                      .filter((event) => event.time.split('-')[1] >= currentTimeString)
                      .slice(0, 2)"
                    :key="eIndex"
                    class="event"
                    :class="{ 'mt-1': eIndex !== 0 }"
                  >
                    <div
                      class="flex items-center justify-between"
                      :class="{
                        'text-[#EBC999]':
                          event.time.split('-')[0] <= currentTimeString &&
                          event.time.split('-')[1] >= currentTimeString,
                      }"
                    >
                      <div class="time text-lg">{{ event.time }}</div>
                      <div class="name truncate-text text-lg">
                        {{ event.name.length > 8 ? event.name.slice(0, 8) + '..' : event.name }}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 當 roomData 少於 21 時，正常顯示 -->
      <div v-else class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
        <div
          v-for="(room, index) in transformedRoomData"
          :key="index"
          class="h-[154px] w-[328px] rounded-[15px] border-[3px] border-[rgb(204,204,204,0.7)] bg-[rgb(2,4,20,0.6)] p-4"
          :class="{
            '!border-[#03B0EC] !shadow-[0_0_4px_3px_rgba(3,176,236,0.5)]':
              room.order[0]?.time.split('-')[0] <= currentTimeString &&
              room.order[0]?.time.split('-')[1] >= currentTimeString,
          }"
        >
          <!-- 場域名稱 -->
          <div class="location mb-6 text-[20px] text-white">{{ room.field }}</div>
          <!-- 活動列表 -->
          <div class="events">
            <div
              v-for="(event, eIndex) in room.order
                .filter((event) => event.time.split('-')[1] >= currentTimeString)
                .slice(0, 2)"
              :key="eIndex"
              class="event"
              :class="{ 'mt-1': eIndex !== 0 }"
            >
              <div
                class="flex items-center justify-between"
                :class="{
                  'text-[#EBC999]':
                    event.time.split('-')[0] <= currentTimeString &&
                    event.time.split('-')[1] >= currentTimeString,
                }"
              >
                <div class="time text-lg">{{ event.time }}</div>
                <div class="name truncate-text text-lg">
                  {{ event.name }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.truncate-text {
  width: 150px; /* 限制文本寬度 */
  white-space: nowrap; /* 強制單行顯示 */
  overflow: hidden; /* 隱藏溢出的部分 */
  text-overflow: ellipsis; /* 超出部分用省略號表示 */
  text-align: right;
}
</style>
