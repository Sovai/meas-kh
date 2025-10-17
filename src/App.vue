<template>
  <div
    class="min-h-screen bg-gradient-to-br from-amber-50 via-yellow-50 to-orange-50 p-2 sm:p-4 py-4 sm:py-8"
  >
    <div class="max-w-3xl mx-auto">
      <div class="text-center mb-4 sm:mb-8">
        <div class="flex justify-center items-center mb-4 sm:mb-6 relative">
          <h1 class="text-2xl sm:text-3xl lg:text-4xl font-bold text-amber-900">
            {{ t.app.title }}
          </h1>
          <div class="absolute right-0 flex items-center gap-2">
            <button
              @click="showPricingModal = true"
              class="px-2 sm:px-3 py-1.5 sm:py-2 bg-blue-50 hover:bg-blue-100 border border-blue-200 rounded-lg sm:rounded-xl transition-all duration-200 flex items-center gap-1 sm:gap-2 text-blue-700 font-medium text-sm sm:text-base"
              title="How we calculate prices"
            >
              <svg
                class="w-4 h-4 sm:w-5 sm:h-5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                ></path>
              </svg>
            </button>
            <button
              @click="toggleLanguage"
              class="px-2 sm:px-4 py-1.5 sm:py-2 bg-white/80 backdrop-blur-sm border border-amber-200 rounded-lg sm:rounded-xl hover:bg-white transition-all duration-200 flex items-center gap-1 sm:gap-2 text-amber-800 font-medium text-sm sm:text-base"
              title="Switch Language"
            >
              <svg
                class="w-4 h-4 sm:w-5 sm:h-5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M3 5h12M9 3v2m1.048 9.5A18.022 18.022 0 016.412 9m6.088 9h7M11 21l5-10 5 10M12.751 5C11.783 10.77 8.07 15.61 3 18.129"
                ></path>
              </svg>
              <span class="hidden sm:inline">{{
                currentLanguage === 'km' ? 'English' : 'ខ្មែរ'
              }}</span>
            </button>
          </div>
        </div>
        <p class="text-sm sm:text-base text-amber-700">{{ t.app.subtitle }}</p>
      </div>

      <div
        class="bg-white/80 backdrop-blur-sm rounded-2xl sm:rounded-3xl p-4 sm:p-6 lg:p-8 border border-amber-200 mb-4 sm:mb-6"
      >
        <div v-if="error" class="text-center py-8 sm:py-12">
          <p class="text-red-600 mb-4 text-sm sm:text-base">{{ error }}</p>
          <button
            @click="fetchGoldPrice"
            class="px-4 sm:px-6 py-2 sm:py-3 bg-amber-600 text-white rounded-xl hover:bg-amber-700 transition-colors text-sm sm:text-base"
          >
            {{ t.app.retry }}
          </button>
        </div>

        <div v-else-if="loading && !goldPrice" class="text-center py-8 sm:py-12">
          <svg
            class="w-10 h-10 sm:w-12 sm:h-12 animate-spin text-amber-600 mx-auto mb-4"
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
          <p class="text-amber-700 text-sm sm:text-base">{{ t.app.loading }}</p>
        </div>

        <div v-else>
          <div
            class="flex flex-col sm:flex-row items-start sm:items-center justify-between mb-4 sm:mb-6 gap-3"
          >
            <div class="flex items-center gap-2 sm:gap-3">
              <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-green-500 rounded-full animate-pulse"></div>
              <span class="text-xs sm:text-sm text-amber-600">{{ t.app.liveUpdates }}</span>
            </div>
            <div class="flex items-center gap-2 sm:gap-3 w-full sm:w-auto">
              <div class="text-[10px] sm:text-xs text-amber-600 flex-1 sm:flex-none">
                {{ t.app.lastUpdated }}: {{ formatTime(lastUpdate) }}
              </div>
              <button
                @click="fetchGoldPrice"
                :disabled="loading"
                class="px-2 sm:px-3 py-1 sm:py-1.5 bg-amber-600 text-white text-[10px] sm:text-xs font-medium rounded-lg hover:bg-amber-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-1 sm:gap-1.5 whitespace-nowrap"
                :title="t.app.refresh"
              >
                <svg
                  class="w-3 h-3 sm:w-3.5 sm:h-3.5"
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
            data-chi-card
            class="bg-gradient-to-br from-amber-200 to-orange-200 rounded-2xl sm:rounded-3xl p-4 sm:p-6 lg:p-10 border-2 sm:border-4 border-amber-500 mb-4 relative"
          >
            <button
              @click="downloadChiCardAsImage"
              :disabled="isDownloadingImage"
              data-html2canvas-ignore
              class="absolute top-2 right-2 sm:top-4 sm:right-4 p-1.5 sm:p-2 bg-white/90 hover:bg-white rounded-lg transition-all duration-200 group disabled:opacity-50 disabled:cursor-not-allowed"
              :title="isDownloadingImage ? 'Generating image...' : t.app.downloadImage"
            >
              <svg
                v-if="isDownloadingImage"
                class="w-4 h-4 sm:w-5 sm:h-5 text-amber-700 animate-spin"
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
                class="w-4 h-4 sm:w-5 sm:h-5 text-amber-700 group-hover:text-amber-900"
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

            <div class="text-center mb-4 sm:mb-6 pr-8 sm:pr-0">
              <div class="text-amber-900 text-base sm:text-xl font-bold mb-1 sm:mb-2">
                {{ t.units.perChi }}
              </div>
              <div class="text-[10px] sm:text-[11px] text-amber-700">
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

            <div class="mb-4 sm:mb-6 pb-4 sm:pb-6 border-b-2 border-amber-400/40">
              <div class="text-center text-xs sm:text-sm font-bold text-amber-900 mb-3 sm:mb-4">
                {{ t.chiPricing.goldBar }}
              </div>
              <div class="grid grid-cols-2 gap-3 sm:gap-6">
                <div class="text-center">
                  <div
                    class="text-xs sm:text-sm text-amber-800 uppercase mb-1 sm:mb-2 font-semibold"
                  >
                    {{ t.chiPricing.buy }}
                  </div>
                  <div class="text-xl sm:text-3xl lg:text-5xl font-black text-amber-950 break-all">
                    {{ formatPrice(chiBarBuy) }}
                  </div>
                </div>
                <div class="text-center">
                  <div
                    class="text-xs sm:text-sm text-blue-700 uppercase mb-1 sm:mb-2 font-semibold"
                  >
                    {{ t.chiPricing.sell }}
                  </div>
                  <div class="text-xl sm:text-3xl lg:text-5xl font-black text-blue-900 break-all">
                    {{ formatPrice(chiBarSell) }}
                  </div>
                </div>
              </div>
            </div>

            <div>
              <div class="text-center text-xs sm:text-sm font-bold text-amber-900 mb-3 sm:mb-4">
                {{ t.chiPricing.goldJewel }}
              </div>
              <div class="grid grid-cols-2 gap-3 sm:gap-6">
                <div class="text-center">
                  <div
                    class="text-xs sm:text-sm text-amber-800 uppercase mb-1 sm:mb-2 font-semibold"
                  >
                    {{ t.chiPricing.buy }}
                  </div>
                  <div class="text-xl sm:text-3xl lg:text-5xl font-black text-amber-950 break-all">
                    {{ formatPrice(chiJewelBuy) }}
                  </div>
                </div>
                <div class="text-center">
                  <div
                    class="text-xs sm:text-sm text-blue-700 uppercase mb-1 sm:mb-2 font-semibold"
                  >
                    {{ t.chiPricing.sell }}
                  </div>
                  <div class="text-xl sm:text-3xl lg:text-5xl font-black text-blue-900 break-all">
                    {{ formatPrice(chiJewelSell) }}
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div
            class="bg-white/60 backdrop-blur-sm rounded-xl sm:rounded-2xl p-3 sm:p-4 border border-slate-200 mb-4 sm:mb-8"
          >
            <div
              class="text-[10px] sm:text-xs text-slate-600 font-semibold mb-2 sm:mb-3 text-center uppercase"
            >
              {{ t.units.referencePrices }}
            </div>
            <div class="grid grid-cols-2 gap-2 sm:gap-4">
              <div class="text-center p-2 sm:p-3 bg-amber-50/50 rounded-lg sm:rounded-xl">
                <div class="text-[9px] sm:text-[10px] text-amber-700 mb-1">
                  {{ t.units.troyOunce }}
                </div>
                <div class="text-sm sm:text-xl font-bold text-amber-900 break-all">
                  {{ formatPrice(displayGoldPrice) }}
                </div>
              </div>

              <div class="text-center p-2 sm:p-3 bg-amber-50/50 rounded-lg sm:rounded-xl">
                <div class="text-[9px] sm:text-[10px] text-amber-700 mb-1">
                  {{ t.units.perGram }}
                </div>
                <div class="text-sm sm:text-xl font-bold text-amber-900 break-all">
                  {{ formatPrice(pricePerGram) }}
                </div>
                <div class="text-[9px] sm:text-[10px] text-amber-600 mt-1">
                  per 1.0{{ t.units.gram }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Consolidated Footer Section -->
      <div
        class="mt-4 sm:mt-6 bg-gradient-to-br from-amber-50 to-orange-50 rounded-xl sm:rounded-2xl border border-amber-200 overflow-hidden"
      >
        <!-- Chi Description & Conversions -->
        <div class="p-4 sm:p-5 space-y-3">
          <div class="text-center">
            <p
              class="text-[10px] sm:text-xs text-amber-700 leading-relaxed"
              v-html="t.info.chiDescription"
            ></p>
          </div>
          <div class="text-center text-xs sm:text-sm text-amber-600 font-medium">
            <p>{{ t.info.conversions }}</p>
          </div>
        </div>

        <!-- Disclaimer -->
        <div class="bg-amber-100/60 border-t border-amber-200 p-4 sm:p-5">
          <p class="text-[10px] sm:text-xs text-amber-900 text-center leading-relaxed">
            <span class="block mt-1.5 sm:mt-2">{{ t.info.disclaimer }}</span>
          </p>
        </div>
      </div>
    </div>

    <!-- Pricing Logic Modal -->
    <div
      v-if="showPricingModal"
      class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50"
      @click.self="showPricingModal = false"
    >
      <div
        class="bg-white rounded-2xl sm:rounded-3xl max-w-2xl w-full max-h-[90vh] overflow-y-auto shadow-2xl"
      >
        <div
          class="sticky top-0 bg-gradient-to-r from-amber-500 to-orange-500 p-4 sm:p-6 rounded-t-2xl sm:rounded-t-3xl flex items-center justify-between"
        >
          <h2 class="text-xl sm:text-2xl font-bold text-white flex items-center gap-2">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M9 7h6m0 10v-3m-3 3h.01M9 17h.01M9 14h.01M12 14h.01M15 11h.01M12 11h.01M9 11h.01M7 21h10a2 2 0 002-2V5a2 2 0 00-2-2H7a2 2 0 00-2 2v14a2 2 0 002 2z"
              ></path>
            </svg>
            {{ currentLanguage === 'km' ? 'របៀបគណនាតម្លៃ' : 'How We Calculate Prices' }}
          </h2>
          <button
            @click="showPricingModal = false"
            class="text-white hover:bg-white/20 rounded-lg p-2 transition-colors"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </button>
        </div>

        <div class="p-4 sm:p-6 space-y-4 sm:space-y-6">
          <!-- Base Calculation -->
          <div
            class="bg-gradient-to-br from-blue-50 to-indigo-50 rounded-xl p-4 border border-blue-200"
          >
            <h3 class="text-lg font-bold text-blue-900 mb-3 flex items-center gap-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M13 10V3L4 14h7v7l9-11h-7z"
                ></path>
              </svg>
              {{ currentLanguage === 'km' ? 'គណនាមូលដ្ឋាន' : 'Base Calculation' }}
            </h3>
            <div class="space-y-2 text-sm text-blue-900">
              <div class="bg-white/60 rounded-lg p-3">
                <p class="font-semibold mb-1">
                  1. {{ currentLanguage === 'km' ? 'តម្លៃក្នុងមួយក្រាម' : 'Price Per Gram' }}:
                </p>
                <p class="font-mono text-xs sm:text-sm">
                  {{ currentLanguage === 'km' ? 'តម្លៃមាសអន្តរជាតិ' : 'International Gold Price' }}
                  ÷ 31.1035g = <span class="font-bold">{{ formatPrice(pricePerGram) }}</span
                  >/g
                </p>
              </div>
              <div class="bg-white/60 rounded-lg p-3">
                <p class="font-semibold mb-1">
                  2. {{ currentLanguage === 'km' ? 'តម្លៃក្នុងមួយជី' : 'Price Per Chi' }}:
                </p>
                <p class="font-mono text-xs sm:text-sm">
                  {{ formatPrice(pricePerGram) }}/g × 3.75g =
                  <span class="font-bold text-amber-600">{{ formatPrice(pricePerChi) }}</span
                  >/chi
                </p>
              </div>
            </div>
          </div>

          <!-- Gold Bar Pricing -->
          <div
            class="bg-gradient-to-br from-amber-50 to-yellow-50 rounded-xl p-4 border border-amber-300"
          >
            <h3 class="text-lg font-bold text-amber-900 mb-3 flex items-center gap-2">
              <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path d="M4 3a2 2 0 100 4h12a2 2 0 100-4H4z"></path>
                <path
                  fill-rule="evenodd"
                  d="M3 8h14v7a2 2 0 01-2 2H5a2 2 0 01-2-2V8zm5 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z"
                  clip-rule="evenodd"
                ></path>
              </svg>
              {{ t.chiPricing.goldBar }} ({{
                currentLanguage === 'km' ? 'មាសសុទ្ធ 24K' : 'Pure 24K Gold'
              }})
            </h3>
            <div class="space-y-3">
              <div class="bg-white/60 rounded-lg p-3">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-semibold text-amber-800"
                    >{{ t.chiPricing.buy }} ({{
                      currentLanguage === 'km' ? 'ហាងទិញពីអតិថិជន' : 'Shop Buys from Customer'
                    }}):</span
                  >
                  <span class="font-bold text-lg text-amber-900">{{ formatPrice(chiBarBuy) }}</span>
                </div>
                <p class="text-xs text-amber-700">
                  {{
                    currentLanguage === 'km'
                      ? 'ហាងទិញត្រឡប់ - បញ្ចុះតម្លៃ $5'
                      : 'Shop buys back - discount $5'
                  }}
                </p>
                <p class="font-mono text-xs mt-1 bg-amber-100/50 p-2 rounded">
                  {{ formatPrice(pricePerChi) }} - $5 = {{ formatPrice(chiBarBuy) }}
                </p>
              </div>
              <div class="bg-white/60 rounded-lg p-3">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-semibold text-blue-800"
                    >{{ t.chiPricing.sell }} ({{
                      currentLanguage === 'km' ? 'ហាងលក់ទៅអតិថិជន' : 'Shop Sells to Customer'
                    }}):</span
                  >
                  <span class="font-bold text-lg text-blue-900">{{ formatPrice(chiBarSell) }}</span>
                </div>
                <p class="text-xs text-blue-700">
                  {{
                    currentLanguage === 'km'
                      ? 'តម្លៃមូលដ្ឋាន (មាសសុទ្ធ 24K)'
                      : 'Base price (pure 24K gold)'
                  }}
                </p>
                <p class="font-mono text-xs mt-1 bg-blue-100/50 p-2 rounded">
                  {{ formatPrice(pricePerChi) }} + $0 = {{ formatPrice(chiBarSell) }}
                </p>
              </div>
            </div>
          </div>

          <!-- Gold Jewelry Pricing -->
          <div
            class="bg-gradient-to-br from-orange-50 to-red-50 rounded-xl p-4 border border-orange-300"
          >
            <h3 class="text-lg font-bold text-orange-900 mb-3 flex items-center gap-2">
              <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path
                  d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                ></path>
              </svg>
              {{ t.chiPricing.goldJewel }} ({{
                currentLanguage === 'km' ? 'សុទ្ធភាពទាប' : 'Lower Purity'
              }})
            </h3>
            <div class="space-y-3">
              <div class="bg-white/60 rounded-lg p-3">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-semibold text-orange-800"
                    >{{ t.chiPricing.buy }} ({{
                      currentLanguage === 'km' ? 'ហាងទិញពីអតិថិជន' : 'Shop Buys from Customer'
                    }}):</span
                  >
                  <span class="font-bold text-lg text-orange-900">{{
                    formatPrice(chiJewelBuy)
                  }}</span>
                </div>
                <p class="text-xs text-orange-700">
                  {{
                    currentLanguage === 'km'
                      ? 'បាត់បង់តម្លៃសិប្បកម្ម និងសុទ្ធភាពទាប - បញ្ចុះតម្លៃ $10'
                      : 'Loss of craftsmanship + lower purity - discount $10'
                  }}
                </p>
                <p class="font-mono text-xs mt-1 bg-orange-100/50 p-2 rounded">
                  {{ formatPrice(pricePerChi) }} - $10 = {{ formatPrice(chiJewelBuy) }}
                </p>
              </div>
              <div class="bg-white/60 rounded-lg p-3">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-semibold text-blue-800"
                    >{{ t.chiPricing.sell }} ({{
                      currentLanguage === 'km' ? 'ហាងលក់ទៅអតិថិជន' : 'Shop Sells to Customer'
                    }}):</span
                  >
                  <span class="font-bold text-lg text-blue-900">{{
                    formatPrice(chiJewelSell)
                  }}</span>
                </div>
                <p class="text-xs text-blue-700">
                  {{
                    currentLanguage === 'km'
                      ? 'សុទ្ធភាពទាប (22K ឬតិចជាងនេះ) - បញ្ចុះតម្លៃ $5'
                      : 'Lower purity (22K or less) - discount $5'
                  }}
                </p>
                <p class="font-mono text-xs mt-1 bg-blue-100/50 p-2 rounded">
                  {{ formatPrice(pricePerChi) }} - $5 = {{ formatPrice(chiJewelSell) }}
                </p>
              </div>
            </div>
          </div>

          <!-- Why Different Prices -->
          <div
            class="bg-gradient-to-br from-purple-50 to-pink-50 rounded-xl p-4 border border-purple-300"
          >
            <h3 class="text-lg font-bold text-purple-900 mb-3 flex items-center gap-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M8.228 9c.549-1.165 2.03-2 3.772-2 2.21 0 4 1.343 4 3 0 1.4-1.278 2.575-3.006 2.907-.542.104-.994.54-.994 1.093m0 3h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                ></path>
              </svg>
              {{ currentLanguage === 'km' ? 'ហេតុអ្វីតម្លៃខុសគ្នា?' : 'Why Different Prices?' }}
            </h3>
            <ul class="space-y-2 text-sm text-purple-900">
              <li class="flex items-start gap-2">
                <span class="text-amber-500 font-bold mt-0.5">→</span>
                <span
                  ><strong>{{ currentLanguage === 'km' ? 'មាសដុំ' : 'Gold Bars' }}:</strong>
                  {{
                    currentLanguage === 'km'
                      ? 'មាសសុទ្ធ 24K រក្សាតម្លៃបានល្អ'
                      : 'Pure 24K gold, maintains value better'
                  }}</span
                >
              </li>
              <li class="flex items-start gap-2">
                <span class="text-orange-500 font-bold mt-0.5">→</span>
                <span
                  ><strong>{{ currentLanguage === 'km' ? 'មាសគ្រឿង' : 'Gold Jewelry' }}:</strong>
                  {{
                    currentLanguage === 'km'
                      ? 'សុទ្ធភាពទាប (22K, 18K) តម្លៃថោកជាង'
                      : 'Lower purity (22K, 18K), less expensive'
                  }}</span
                >
              </li>
              <li class="flex items-start gap-2">
                <span class="text-blue-500 font-bold mt-0.5">→</span>
                <span
                  ><strong
                    >{{
                      currentLanguage === 'km' ? 'ហាងលក់ទៅអតិថិជន' : 'Shop Sells to Customer'
                    }}:</strong
                  >
                  {{
                    currentLanguage === 'km'
                      ? 'ហាងបន្ថែមប្រាក់ចំណេញ និងមាសគ្រឿងបាត់បង់តម្លៃសិប្បកម្ម'
                      : 'Shop adds profit margin and jewelry has lost craftsmanship value'
                  }}</span
                >
              </li>
            </ul>
          </div>

          <div class="bg-amber-50 border border-amber-300 rounded-lg p-3">
            <p class="text-xs text-amber-800 text-center">
              <strong>{{ currentLanguage === 'km' ? 'សម្គាល់' : 'Note' }}:</strong>
              {{
                currentLanguage === 'km'
                  ? 'តម្លៃពិតប្រាកដអាចខុសគ្នារវាងហាងនីមួយៗ សូមពិនិត្យជាមួយហាងមាសក្នុងតំបន់របស់អ្នក'
                  : 'Actual prices may vary between shops. Always verify with your local gold dealer.'
              }}
            </p>
          </div>
        </div>
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
const showPricingModal = ref(false)

const CHI_TO_GRAMS = 3.75
const GRAMS_PER_OUNCE = 31.1035
const CHI_JEWEL_DISCOUNT = 5 // Jewelry is cheaper (lower purity than bars)
const CHI_BAR_SELL_DISCOUNT = 5 // Shop buys back bars at a discount
const CHI_JEWEL_SELL_DISCOUNT = 10 // Shop buys back jewelry at a larger discount (lower purity)

const pricePerGram = computed(() => {
  return displayGoldPrice.value ? displayGoldPrice.value / GRAMS_PER_OUNCE : 0
})

const pricePerChi = computed(() => {
  return pricePerGram.value * CHI_TO_GRAMS
})

// Gold Bar Pricing (Higher - Pure 24K gold)
const chiBarBuy = computed(() => {
  // Shop BUYS from customer (shop pays customer this price)
  return pricePerChi.value - CHI_BAR_SELL_DISCOUNT
})

const chiBarSell = computed(() => {
  // Shop SELLS to customer (customer pays shop this price)
  return pricePerChi.value
})

// Gold Jewelry Pricing (Lower - Usually 22K or less)
const chiJewelBuy = computed(() => {
  // Shop BUYS from customer (shop pays customer this price - lower purity + lost craftsmanship)
  return pricePerChi.value - CHI_JEWEL_SELL_DISCOUNT
})

const chiJewelSell = computed(() => {
  // Shop SELLS to customer (customer pays shop this price - lower purity than bars)
  return pricePerChi.value - CHI_JEWEL_DISCOUNT
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
    const canvas = await html2canvas(chiCardRef.value, {
      backgroundColor: null,
      scale: 2,
      logging: false,
      useCORS: true,
      width: 600, // Fixed width for consistent output
      windowWidth: 600, // Set window width to match
      onclone: (clonedDoc) => {
        // Show the date stamp in the cloned document
        const dateStamp = clonedDoc.querySelector('.date-stamp')
        if (dateStamp) {
          dateStamp.classList.remove('hidden')
        }

        // Set fixed width on the chi card for consistent sizing
        const chiCard = clonedDoc.querySelector('[data-chi-card]')
        if (chiCard) {
          chiCard.style.width = '600px'
          chiCard.style.padding = '40px'

          // Add disclaimer to the cloned document
          const disclaimer = clonedDoc.createElement('div')
          disclaimer.className = 'mt-4 pt-4 border-t-2 border-amber-400/40'
          disclaimer.innerHTML = `
            <p style="font-size: 10px; line-height: 1.4; color: #78350f; text-align: center; margin: 0;">
              <span style="display: block; margin-top: 4px;">${t.value.info.disclaimer}</span>
            </p>
          `
          chiCard.appendChild(disclaimer)
        }
      },
    })

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
