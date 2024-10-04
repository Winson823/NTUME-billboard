<script setup lang="ts">
import 'swiper/css'

import axios from 'axios'
import Swiper from 'swiper'
import { Autoplay } from 'swiper/modules'
import { computed, onMounted } from 'vue'

// 引入 roomdata
import roomData from '@/assets/roomdata.json'

// 初始化應用
import { useInitApp } from './composables/useInitApp'
const { initApp, currentLocale } = useInitApp()

// API 請求
async function initData() {
  const resp = await axios.get('')
  const reserveData = resp.data
}

onMounted(async () => {
  await initData()

  // 初始化 Swiper
  new Swiper('.swiper', {
    modules: [Autoplay],
    loop: true, // 循環輪播
    autoplay: {
      delay: 1000, //
    },
  })
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

// 分頁 roomData，每頁最多 21 個
const paginatedRoomData = computed(() => {
  const chunkSize = 21
  const pages = []

  for (let i = 0; i < roomData.length; i += chunkSize) {
    pages.push(roomData.slice(i, i + chunkSize))
  }
  console.log(pages)
  return pages
})
</script>

<template>
  <div class="h-[1920px]">
    <!-- 公告欄 -->
    <div class="announcement h-[30%] w-full bg-gray-300">123</div>

    <!-- 房間區塊 -->
    <div class="roomInfo h-[70%] w-full bg-black p-4 text-white">
      <!-- 判斷 roomData 是否超過 21，使用 Swiper -->
      <div v-if="paginatedRoomData.length > 1" class="swiper">
        <div class="swiper-wrapper">
          <!-- 使用 v-for 迭代每個分頁 -->
          <div v-for="(page, pageIndex) in paginatedRoomData" :key="pageIndex" class="swiper-slide">
            <div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
              <!-- 使用 v-for 迭代每一頁中的 room -->
              <div
                v-for="(room, index) in page"
                :key="index"
                class="h-[154px] w-[328px] rounded-[15px] border border-white bg-black p-4"
              >
                <!-- 場域名稱 -->
                <div class="location mb-6 text-[24px] font-bold">{{ room.location }}</div>
                <!-- 活動列表 -->
                <div class="events">
                  <div
                    v-for="(event, eIndex) in room.events
                      .filter((event) => event.endTime >= currentTimeString)
                      .slice(0, 2)"
                    :key="eIndex"
                    class="event"
                    :class="{ 'mt-1': eIndex !== 0 }"
                  >
                    <div
                      class="flex items-center justify-between"
                      :class="{
                        'text-[#EBC999]':
                          event.startTime <= currentTimeString &&
                          event.endTime >= currentTimeString,
                      }"
                    >
                      <div class="time text-lg">{{ event.startTime }} - {{ event.endTime }}</div>
                      <div class="name text-lg">
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
          v-for="(room, index) in roomData"
          :key="index"
          class="h-[154px] w-[328px] rounded-[15px] border border-white bg-black p-4"
        >
          <!-- 場域名稱 -->
          <div class="location mb-6 text-[24px] font-bold">{{ room.location }}</div>
          <!-- 活動列表 -->
          <div class="events">
            <div
              v-for="(event, eIndex) in room.events
                .filter((event) => event.endTime >= currentTimeString)
                .slice(0, 2)"
              :key="eIndex"
              class="event"
              :class="{ 'mt-1': eIndex !== 0 }"
            >
              <div
                class="flex items-center justify-between"
                :class="{
                  'text-[#EBC999]':
                    event.startTime <= currentTimeString && event.endTime >= currentTimeString,
                }"
              >
                <div class="time text-lg">{{ event.startTime }} - {{ event.endTime }}</div>
                <div class="name text-lg">
                  {{ event.name.length > 8 ? event.name.slice(0, 8) + '..' : event.name }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
