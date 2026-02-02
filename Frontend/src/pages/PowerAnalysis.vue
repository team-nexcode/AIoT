<script setup lang="ts">
import { computed } from 'vue'
import {
  Activity,
  AlertTriangle,
  BarChart3,
  FileText,
  PlugZap,
  Sparkles,
  TrendingUp,
  CheckCircle2 // 👈 누락되었던 아이콘 추가
} from 'lucide-vue-next'

/**
 * 🔹 타입 정의
 */
type AnalysisReport = {
  hours: number
  waste: { standby_wh: number }
  anomalies: { count: number }
  state_now: { state: string }
}

// 🔹 Props 설정 (기본값 포함)
const props = defineProps<{
  report?: AnalysisReport
}>()

// 🔹 데이터 설정 (중복 선언 제거됨)
const hourlyUsage = [
  { hour: '0', value: 1.2 },
  { hour: '3', value: 0.8 },
  { hour: '6', value: 1.5 },
  { hour: '9', value: 3.2 },
  { hour: '12', value: 4.1 },
  { hour: '15', value: 3.6 },
  { hour: '18', value: 5.4 },
  { hour: '21', value: 4.8 },
]

const topDevices = [
  { name: '온풍기', usage: 38 },
  { name: '전기히터', usage: 22 },
  { name: 'TV', usage: 11 },
]

// 🔹 계산된 속성 (Computed)
const report = computed<AnalysisReport>(() => 
  props.report ?? {
    hours: 6,
    waste: { standby_wh: 58.32 },
    anomalies: { count: 4 },
    state_now: { state: 'ON' },
  }
)

const maxUsage = Math.max(...hourlyUsage.map((h) => h.value), 0.1) // 0으로 나누기 방지

const standbyHigh = computed(() => report.value.waste.standby_wh >= 50)
const anomaliesHigh = computed(() => report.value.anomalies.count >= 3)
const isRisky = computed(() => standbyHigh.value || anomaliesHigh.value)

const statusBadge = computed(() => {
  return isRisky.value
    ? { text: '주의', cls: 'bg-amber-500/10 text-amber-200 border-amber-500/20' }
    : { text: '정상', cls: 'bg-emerald-500/10 text-emerald-200 border-emerald-500/20' }
})

const summary = computed(() => {
  const { hours, waste, anomalies, state_now } = report.value
  let s = `최근 ${hours}시간 기준 standby 추정 ${waste.standby_wh.toFixed(2)}Wh, ` +
          `이상치 ${anomalies.count}건, 현재 상태 ${state_now.state}.`

  if (waste.standby_wh >= 50) s += ' standby 낭비가 커서 차단을 권장합니다.'
  if (anomalies.count >= 3) s += ' 기기 점검이 필요해 보입니다.'
  return s
})

const recommendations = computed(() => {
  const list: { tone: 'warn' | 'ok'; title: string; desc: string }[] = []

  if (standbyHigh.value) {
    list.push({ tone: 'warn', title: '미사용 시간대 차단 권장', desc: '대기전력 누적이 커서 스케줄 차단을 추천합니다.' })
  } else {
    list.push({ tone: 'ok', title: 'Standby 상태 양호', desc: '대기전력 수준이 안정적입니다.' })
  }

  if (anomaliesHigh.value) {
    list.push({ tone: 'warn', title: '이상치 점검 필요', desc: '센서나 부하 상태를 확인해 주세요.' })
  } else {
    list.push({ tone: 'ok', title: '이상치 없음', desc: '측정값이 매우 안정적입니다.' })
  }

  return list
})
</script>

<template>
  <div class="p-6 space-y-10 bg-black min-h-screen">
    <div>
      <h2 class="text-2xl font-bold text-white tracking-tight">전력 분석</h2>
      <p class="text-sm text-gray-400 mt-1">에너지 사용 패턴과 AI 진단 결과를 확인하세요</p>
    </div>

    <div class="bg-gray-900/50 border border-gray-800 rounded-2xl p-6">
      <div class="flex items-center justify-between mb-8">
        <h3 class="text-white font-semibold flex items-center gap-2">
          <BarChart3 class="w-5 h-5 text-blue-400" />
          시간대별 평균 전력 사용량
        </h3>
        <span class="text-xs text-blue-400 flex items-center gap-1">
          <TrendingUp class="w-4 h-4" /> kWh
        </span>
      </div>

      <div class="flex items-end gap-3 h-44">
        <div v-for="item in hourlyUsage" :key="item.hour" class="flex-1 flex flex-col items-center group">
          <div
            class="w-full rounded-lg bg-gradient-to-t from-blue-600 to-blue-400 transition-all duration-500"
            :style="{ height: (item.value / maxUsage * 100) + '%' }"
          />
          <span class="text-[11px] text-gray-500 mt-2">{{ item.hour }}시</span>
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div class="lg:col-span-2 rounded-3xl border border-white/10 bg-white/5 backdrop-blur-xl shadow-2xl overflow-hidden">
        <div class="p-6">
          <div class="flex items-start justify-between mb-6">
            <div>
              <h3 class="text-white font-semibold flex items-center gap-2 text-lg">
                <Sparkles class="w-5 h-5 text-purple-400" />
                AI 분석 리포트
              </h3>
              <p class="text-[13px] text-white/50 mt-1">최근 {{ report.hours }}시간 데이터 기반</p>
            </div>
            <div class="flex gap-2">
              <span class="text-[11px] px-2.5 py-1 rounded-full border" :class="statusBadge.cls">
                {{ statusBadge.text }}
              </span>
            </div>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-6">
            <div class="bg-white/5 border border-white/10 rounded-2xl p-4">
              <div class="text-[12px] text-white/50 flex items-center gap-2 mb-1">
                <Activity class="w-4 h-4" /> Standby
              </div>
              <div class="text-2xl font-bold text-white">{{ report.waste.standby_wh.toFixed(1) }} <span class="text-sm font-normal opacity-50">Wh</span></div>
            </div>
            <div class="bg-white/5 border border-white/10 rounded-2xl p-4">
              <div class="text-[12px] text-white/50 flex items-center gap-2 mb-1">
                <AlertTriangle class="w-4 h-4" /> 이상치
              </div>
              <div class="text-2xl font-bold text-white">{{ report.anomalies.count }} <span class="text-sm font-normal opacity-50">건</span></div>
            </div>
            <div class="bg-white/5 border border-white/10 rounded-2xl p-4">
              <div class="text-[12px] text-white/50 flex items-center gap-2 mb-1">
                <FileText class="w-4 h-4" /> 현재 상태
              </div>
              <div class="text-2xl font-bold text-white">{{ report.state_now.state }}</div>
            </div>
          </div>

          <div class="bg-white/5 border border-white/10 rounded-2xl p-5 mb-6">
            <p class="text-sm leading-relaxed text-white/80">{{ summary }}</p>
          </div>

          <div class="space-y-3">
            <div v-for="(it, idx) in recommendations" :key="idx" 
              class="flex items-center gap-4 p-4 rounded-2xl bg-black/20 border border-white/5">
              <div class="w-10 h-10 rounded-xl bg-white/5 flex items-center justify-center flex-shrink-0">
                <CheckCircle2 v-if="it.tone === 'ok'" class="w-5 h-5 text-emerald-400" />
                <AlertTriangle v-else class="w-5 h-5 text-amber-400" />
              </div>
              <div>
                <h4 class="text-sm font-semibold text-white">{{ it.title }}</h4>
                <p class="text-xs text-white/50 mt-0.5">{{ it.desc }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="bg-gray-900/50 border border-gray-800 rounded-3xl p-6">
        <h3 class="text-white font-semibold mb-6 flex items-center gap-2">
          <PlugZap class="w-5 h-5 text-yellow-400" />
          소비 상위 디바이스
        </h3>
        <div class="space-y-6">
          <div v-for="(device, idx) in topDevices" :key="device.name">
            <div class="flex justify-between text-sm mb-2">
              <span class="text-white/70">{{ device.name }}</span>
              <span class="text-white font-medium">{{ device.usage }}%</span>
            </div>
            <div class="h-1.5 w-full bg-gray-800 rounded-full overflow-hidden">
              <div class="h-full bg-blue-500 rounded-full" :style="{ width: device.usage + '%' }" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>