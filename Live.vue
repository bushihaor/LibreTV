<template>
  <div class="p-4">
    <h1 class="text-xl font-bold mb-4">📺 IPTV 直播频道</h1>

    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
      <button
        v-for="(item, index) in channels"
        :key="index"
        @click="currentUrl = item.url"
        class="bg-gray-800 hover:bg-pink-600 text-white text-sm py-2 px-3 rounded"
      >
        {{ item.name }}
      </button>
    </div>

    <div v-if="currentUrl" class="mt-6">
      <h2 class="text-lg font-semibold mb-2">正在播放：{{ currentChannelName }}</h2>
      <video :src="currentUrl" controls autoplay class="w-full h-[400px] rounded shadow-md" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const channels = ref([])
const currentUrl = ref('')
const currentChannelName = ref('')

// 示例 M3U 文件地址（你可以换成自己托管的）
const M3U_URL = 'https://raw.githubusercontent.com/maokefan/iptvtest/main/cn.m3u'

const parseM3U = async () => {
  const res = await fetch(M3U_URL)
  const text = await res.text()
  const lines = text.split('\n')

  const list = []
  for (let i = 0; i < lines.length; i++) {
    if (lines[i].startsWith('#EXTINF')) {
      const nameMatch = lines[i].match(/,(.*)$/)
      const name = nameMatch ? nameMatch[1] : `频道 ${i}`
      const url = lines[i + 1]?.trim()
      if (url && url.startsWith('http')) {
        list.push({ name, url })
      }
    }
  }

  channels.value = list
}

onMounted(() => {
  parseM3U()
})

watchEffect(() => {
  const current = channels.value.find((c) => c.url === currentUrl.value)
  currentChannelName.value = current?.name || ''
})
</script>

<style scoped>
video {
  background: black;
}
</style>
