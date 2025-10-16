<template>
  <div class="min-h-screen bg-gradient-to-br from-amber-50 via-yellow-50 to-orange-50 p-4 py-8">
    <div class="max-w-6xl mx-auto">
      <div class="text-center mb-8 relative">
        <button
          @click="toggleLanguage"
          class="absolute right-0 top-0 px-4 py-2 bg-white/80 backdrop-blur-sm border border-amber-200 rounded-xl hover:bg-white hover:shadow-md transition-all duration-200 flex items-center gap-2 text-amber-800 font-medium"
          title="Switch Language"
        >
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M3 5h12M9 3v2m1.048 9.5A18.022 18.022 0 016.412 9m6.088 9h7M11 21l5-10 5 10M12.751 5C11.783 10.77 8.07 15.61 3 18.129"
            ></path>
          </svg>
          <span>{{ currentLanguage === 'km' ? 'English' : 'ខ្មែរ' }}</span>
        </button>
        <h1 class="text-4xl font-bold text-amber-900 mb-2">{{ t.app.title }}</h1>
        <p class="text-amber-700">{{ t.app.subtitle }}</p>
      </div>

      <div
        class="bg-white/80 backdrop-blur-sm rounded-3xl shadow-2xl p-8 border border-amber-200 mb-6"
      >
        <div v-if="error" class="text-center py-12">
          <p class="text-red-600 mb-4">{{ error }}</p>
          <button
            @click="fetchGoldPrice"
            class="px-6 py-3 bg-amber-600 text-white rounded-xl hover:bg-amber-700 transition-colors"
          >
            {{ t.app.retry }}
          </button>
        </div>

        <div v-else-if="loading && !goldPrice" class="text-center py-12">
          <svg
            class="w-12 h-12 animate-spin text-amber-600 mx-auto mb-4"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              class="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              stroke-width="4"
            ></circle>
            <path
              class="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            ></path>
          </svg>
          <p class="text-amber-700">{{ t.app.loading }}</p>
        </div>

        <div v-else>
          <div class="flex items-center justify-between mb-6">
            <div class="flex items-center gap-3">
              <div class="w-3 h-3 bg-green-500 rounded-full animate-pulse"></div>
              <span class="text-sm text-amber-600">{{ t.app.liveUpdates }}</span>
            </div>
            <div class="flex items-center gap-3">
              <div class="text-xs text-amber-600">
                {{ t.app.lastUpdated }}: {{ formatTime(lastUpdate) }}
              </div>
              <button
                @click="fetchGoldPrice"
                :disabled="loading"
                class="px-3 py-1.5 bg-amber-600 text-white text-xs font-medium rounded-lg hover:bg-amber-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-1.5"
                :title="t.app.refresh"
              >
                <svg
                  class="w-3.5 h-3.5"
                  :class="{ 'animate-spin': loading }"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                  ></path>
                </svg>
                {{ t.app.refresh }}
              </button>
            </div>
          </div>

          <div
            ref="chiCardRef"
            class="bg-gradient-to-br from-amber-200 to-orange-200 rounded-3xl p-8 lg:p-10 border-4 border-amber-500 shadow-2xl mb-4 relative"
          >
            <button
              @click="downloadChiCardAsImage"
              :disabled="isDownloadingImage"
              data-html2canvas-ignore
              class="absolute top-4 right-4 p-2 bg-white/90 hover:bg-white rounded-lg shadow-md transition-all duration-200 group disabled:opacity-50 disabled:cursor-not-allowed"
              :title="isDownloadingImage ? 'Generating image...' : t.app.downloadImage"
            >
              <svg
                v-if="isDownloadingImage"
                class="w-5 h-5 text-amber-700 animate-spin"
                fill="none"
                viewBox="0 0 24 24"
              >
                <circle
                  class="opacity-25"
                  cx="12"
                  cy="12"
                  r="10"
                  stroke="currentColor"
                  stroke-width="4"
                ></circle>
                <path
                  class="opacity-75"
                  fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                ></path>
              </svg>
              <svg
                v-else
                class="w-5 h-5 text-amber-700 group-hover:text-amber-900"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
                ></path>
              </svg>
            </button>

            <div class="text-center mb-6">
              <div class="text-amber-900 text-xl font-bold mb-2">{{ t.units.perChi }}</div>
              <div class="text-[11px] text-amber-700">
                3.75 {{ t.units.gram }} = 1 {{ t.units.chi }}
              </div>
              <div class="date-stamp text-[10px] text-amber-600 mt-2 hidden">
                {{
                  new Date().toLocaleDateString(currentLanguage === 'km' ? 'km-KH' : 'en-US', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric',
                    hour: '2-digit',
                    minute: '2-digit',
                  })
                }}
              </div>
            </div>

            <div class="mb-6 pb-6 border-b-2 border-amber-400/40">
              <div class="text-center text-sm font-bold text-amber-900 mb-4">
                {{ t.chiPricing.goldBar }}
              </div>
              <div class="grid grid-cols-2 gap-6">
                <div class="text-center">
                  <div class="text-xs text-amber-800 uppercase mb-2 font-semibold">
                    {{ t.chiPricing.buy }}
                  </div>
                  <div class="text-4xl lg:text-5xl font-black text-amber-950">
                    {{ formatPrice(chiBarBuy) }}
                  </div>
                </div>
                <div class="text-center">
                  <div class="text-xs text-blue-700 uppercase mb-2 font-semibold">
                    {{ t.chiPricing.sell }}
                  </div>
                  <div class="text-4xl lg:text-5xl font-black text-blue-900">
                    {{ formatPrice(chiBarSell) }}
                  </div>
                </div>
              </div>
            </div>

            <div>
              <div class="text-center text-sm font-bold text-amber-900 mb-4">
                {{ t.chiPricing.goldJewel }}
              </div>
              <div class="grid grid-cols-2 gap-6">
                <div class="text-center">
                  <div class="text-xs text-amber-800 uppercase mb-2 font-semibold">
                    {{ t.chiPricing.buy }}
                  </div>
                  <div class="text-4xl lg:text-5xl font-black text-amber-950">
                    {{ formatPrice(chiJewelBuy) }}
                  </div>
                </div>
                <div class="text-center">
                  <div class="text-xs text-blue-700 uppercase mb-2 font-semibold">
                    {{ t.chiPricing.sell }}
                  </div>
                  <div class="text-4xl lg:text-5xl font-black text-blue-900">
                    {{ formatPrice(chiJewelSell) }}
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="bg-white/60 backdrop-blur-sm rounded-2xl p-4 border border-slate-200 mb-8">
            <div class="text-xs text-slate-600 font-semibold mb-3 text-center uppercase">
              {{ t.units.referencePrices }}
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div class="text-center p-3 bg-amber-50/50 rounded-xl">
                <div class="text-[10px] text-amber-700 mb-1">{{ t.units.troyOunce }}</div>
                <div class="text-xl font-bold text-amber-900">
                  {{ formatPrice(displayGoldPrice) }}
                </div>
              </div>

              <div class="text-center p-3 bg-amber-50/50 rounded-xl">
                <div class="text-[10px] text-amber-700 mb-1">{{ t.units.perGram }}</div>
                <div class="text-xl font-bold text-amber-900">
                  {{ formatPrice(pricePerGram) }}
                </div>
                <div class="text-[10px] text-amber-600 mt-1">per 1.0{{ t.units.gram }}</div>
              </div>
            </div>
          </div>

          <div class="mt-6 p-4 bg-amber-50 rounded-xl border border-amber-200">
            <p class="text-xs text-amber-700 text-center" v-html="t.info.chiDescription"></p>
          </div>
        </div>
      </div>

      <div class="text-center text-sm text-amber-700">
        <p>{{ t.info.conversions }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import html2canvas from 'html2canvas-pro'
import kmTranslations from './locales/km.json'
import enTranslations from './locales/en.json'

const currentLanguage = ref('km')
const translations = {
  km: kmTranslations,
  en: enTranslations,
}

const t = computed(() => translations[currentLanguage.value])

const toggleLanguage = () => {
  currentLanguage.value = currentLanguage.value === 'km' ? 'en' : 'km'
  localStorage.setItem('language', currentLanguage.value)
}

const goldPrice = ref(null)
const displayGoldPrice = ref(null)
const loading = ref(true)
const lastUpdate = ref(null)
const error = ref(null)
const updateInterval = ref(null)
const animationFrame = ref(null)
const chiCardRef = ref(null)
const isDownloadingImage = ref(false)

const CHI_TO_GRAMS = 3.75
const GRAMS_PER_OUNCE = 31.1035
const CHI_JEWEL_DISCOUNT = 5
const CHI_SELL_PREMIUM = 5

const pricePerGram = computed(() => {
  return displayGoldPrice.value ? displayGoldPrice.value / GRAMS_PER_OUNCE : 0
})

const pricePerChi = computed(() => {
  return pricePerGram.value * CHI_TO_GRAMS
})

const chiBarBuy = computed(() => {
  return pricePerChi.value
})

const chiJewelBuy = computed(() => {
  return pricePerChi.value - CHI_JEWEL_DISCOUNT
})

const chiBarSell = computed(() => {
  return pricePerChi.value + CHI_SELL_PREMIUM
})

const chiJewelSell = computed(() => {
  return pricePerChi.value + CHI_SELL_PREMIUM
})

const animatePrice = (targetPrice) => {
  if (!displayGoldPrice.value) {
    displayGoldPrice.value = targetPrice
    return
  }

  const startPrice = displayGoldPrice.value
  const difference = targetPrice - startPrice
  const duration = 1000
  const startTime = performance.now()

  const animate = (currentTime) => {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    const easeOutQuad = progress * (2 - progress)

    displayGoldPrice.value = startPrice + difference * easeOutQuad

    if (progress < 1) {
      animationFrame.value = requestAnimationFrame(animate)
    } else {
      displayGoldPrice.value = targetPrice
    }
  }

  if (animationFrame.value) {
    cancelAnimationFrame(animationFrame.value)
  }

  animationFrame.value = requestAnimationFrame(animate)
}

const fetchGoldPrice = async () => {
  loading.value = true
  error.value = null

  try {
    const response = await fetch('https://api.gold-api.com/price/XAU')

    if (!response.ok) {
      throw new Error(`API failed with status: ${response.status}`)
    }

    const data = await response.json()

    if (data.price) {
      updateGoldData(data)
    } else {
      throw new Error('Invalid data received from API')
    }
  } catch (err) {
    console.error('API Error:', err)
    error.value = 'Unable to fetch gold price. Please try again later.'
  } finally {
    loading.value = false
  }
}

const updateGoldData = (data) => {
  goldPrice.value = data.price
  animatePrice(data.price)
  lastUpdate.value = new Date()
}

const formatPrice = (price) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(price)
}

const formatTime = (date) => {
  if (!date) return ''
  return new Intl.DateTimeFormat('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
  }).format(date)
}

const downloadChiCardAsImage = async () => {
  if (!chiCardRef.value || isDownloadingImage.value) return

  isDownloadingImage.value = true

  try {
    const dateStamp = chiCardRef.value.querySelector('.date-stamp')
    if (dateStamp) {
      dateStamp.classList.remove('hidden')
    }

    await new Promise((resolve) => setTimeout(resolve, 50))

    const canvas = await html2canvas(chiCardRef.value, {
      backgroundColor: null,
      scale: 2,
      logging: false,
      useCORS: true,
    })

    if (dateStamp) {
      dateStamp.classList.add('hidden')
    }
    return new Promise((resolve, reject) => {
      canvas.toBlob((blob) => {
        if (!blob) {
          reject(new Error('Failed to create image blob'))
          return
        }

        try {
          const url = URL.createObjectURL(blob)
          const link = document.createElement('a')

          const now = new Date()
          let filename = ''

          if (currentLanguage.value === 'km') {
            const dateStr = now.toLocaleDateString('km-KH', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
            })
            filename = `តម្លៃមាស សម្រាប់ថ្ងៃ ${dateStr}.png`
          } else {
            const dateStr = now.toLocaleDateString('en-US', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
            })
            filename = `Gold price for ${dateStr}.png`
          }

          link.download = filename
          link.href = url
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)

          setTimeout(() => URL.revokeObjectURL(url), 100)
          resolve()
        } catch (err) {
          reject(err)
        }
      }, 'image/png')
    })
  } catch (err) {
    console.error('Error downloading image:', err)
    alert('Failed to download image. Please try again.')
  } finally {
    isDownloadingImage.value = false
  }
}

onMounted(async () => {
  const savedLanguage = localStorage.getItem('language')
  if (savedLanguage && (savedLanguage === 'km' || savedLanguage === 'en')) {
    currentLanguage.value = savedLanguage
  }

  await fetchGoldPrice()

  updateInterval.value = setInterval(() => {
    fetchGoldPrice()
  }, 30000)
})

onBeforeUnmount(() => {
  if (updateInterval.value) {
    clearInterval(updateInterval.value)
  }
  if (animationFrame.value) {
    cancelAnimationFrame(animationFrame.value)
  }
})
</script>

<style scoped>
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
</style>
