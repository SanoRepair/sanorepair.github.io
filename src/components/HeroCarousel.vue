<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import logo from '@/assets/images/sano-logo.png'

const props = defineProps({
  slides: { type: Array, required: true },   // [{ type: 'image' | 'brand', src?, label? }]
  duration: { type: Number, default: 10000 }, // 다음 화면까지 시간 (ms)
  move: { type: Number, default: 1100 }       // 화면이 움직이는 시간 (ms)
})

const MOBILE_QUERY = '(max-width: 760px)'
const N = computed(() => props.slides.length)

// 마지막 → 처음으로 자연스럽게 이어지도록 첫 화면을 한 장 더 붙여요
const renderSlides = computed(() => [...props.slides, props.slides[0]])

const pos = ref(0)        // 트랙 위치 (N 이면 복제된 첫 화면)
const cur = ref(0)        // 활성 메뉴 번호
const cycle = ref(0)      // 진행 막대 애니메이션 재시작용
const animate = ref(false)
const isMobile = ref(false)

const trackStyle = computed(() => ({
  transform: isMobile.value
    ? `translateX(${-pos.value * 100}%)`
    : `translateY(${-pos.value * 100}%)`,
  transition: animate.value ? `transform ${props.move}ms cubic-bezier(.65,0,.35,1)` : 'none'
}))

function go(i, withAnim = true) {
  animate.value = withAnim
  if (i === N.value) {          // 3 → 1 : 복제본까지 이동한 뒤 처음으로 순간 이동
    pos.value = N.value
    cur.value = 0
  } else {
    cur.value = (i + N.value) % N.value
    pos.value = cur.value
  }
  cycle.value++
}
const next = () => go(cur.value + 1)
const prev = () => go(cur.value - 1)

function onMoved(e) {
  if (e.propertyName !== 'transform') return
  if (pos.value === N.value) {
    animate.value = false
    pos.value = 0
  }
}

// 모바일 스와이프
let sx = null, sy = null
function onTouchStart(e) { sx = e.touches[0].clientX; sy = e.touches[0].clientY }
function onTouchEnd(e) {
  if (sx === null) return
  const dx = e.changedTouches[0].clientX - sx
  const dy = e.changedTouches[0].clientY - sy
  if (Math.abs(dx) > 40 && Math.abs(dx) > Math.abs(dy)) dx < 0 ? next() : prev()
  sx = null
}

function onKey(e) {
  if (e.key === 'ArrowDown' || e.key === 'ArrowRight') next()
  if (e.key === 'ArrowUp' || e.key === 'ArrowLeft') prev()
}

let mq
function onMq() { isMobile.value = mq.matches; animate.value = false }

onMounted(() => {
  mq = window.matchMedia(MOBILE_QUERY)
  onMq()
  mq.addEventListener('change', onMq)
  window.addEventListener('keydown', onKey)
  // 미리보기용: 주소 끝에 #2 를 붙이면 2번 화면에서 시작
  const start = parseInt(location.hash.slice(1))
  go(start > 0 && start <= N.value ? start - 1 : 0, false)
})
onBeforeUnmount(() => {
  mq?.removeEventListener('change', onMq)
  window.removeEventListener('keydown', onKey)
})
</script>

<template>
  <main
    class="hero"
    aria-roledescription="carousel"
    :style="{ '--dur': duration + 'ms' }"
    @touchstart.passive="onTouchStart"
    @touchend="onTouchEnd"
  >
    <div class="track" :style="trackStyle" @transitionend.self="onMoved">
      <section
        v-for="(s, i) in renderSlides"
        :key="i"
        class="slide"
        :class="{ brand: s.type === 'brand' }"
        :style="s.type === 'image' ? { backgroundImage: `url(${s.src})` } : null"
        :aria-label="s.label"
      />
    </div>

    <!-- 가운데 로고: 고정, 뒤의 사진만 바뀜 -->
    <img class="logo" :src="logo" alt="사노리페어 SANO REPAIR" />

    <!-- 왼쪽 1·2·3 메뉴 (모바일에서는 아래 가로) -->
    <nav class="ind" aria-label="메인 화면 선택">
      <button
        v-for="(s, i) in slides"
        :key="i"
        :class="{ on: i === cur, done: i < cur }"
        :aria-label="`${i + 1}번 화면`"
        @click="go(i)"
      >
        <span class="num">{{ i + 1 }}</span>
        <span class="seg">
          <span
            class="fill"
            :key="i === cur ? `on-${cycle}` : `off-${i}`"
            @animationend="i === cur && next()"
          />
        </span>
      </button>
    </nav>
  </main>
</template>

<style scoped>
.hero { position: relative; height: 100vh; height: 100dvh; overflow: hidden; background: #111; }

.track { position: absolute; inset: 0; display: flex; flex-direction: column; will-change: transform; }
.slide { flex: 0 0 100%; width: 100%; height: 100%; background-size: cover; background-position: center; position: relative; }
.slide::after { content: ""; position: absolute; inset: 0; background: linear-gradient(90deg, rgba(0,0,0,.28), rgba(0,0,0,0) 22%); }
.slide.brand { background: var(--maroon); }
.slide.brand::before {
  content: ""; position: absolute; inset: 0; background: var(--sand); opacity: .12;
  -webkit-mask: url('../assets/images/topo.svg') center / cover;
  mask: url('../assets/images/topo.svg') center / cover;
}

.logo {
  position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%);
  height: min(24vh, 30vw); z-index: 3; pointer-events: none;
  filter: drop-shadow(0 6px 18px rgba(0,0,0,.25));
}

/* 왼쪽 세로 메뉴 */
.ind { position: absolute; z-index: 4; left: clamp(22px, 3vw, 46px); top: 9%; bottom: 6%; width: 56px; display: flex; flex-direction: column; }
.ind button { flex: 1; position: relative; border: 0; background: none; color: #fff; cursor: pointer; font: inherit; }
.ind .num { position: absolute; left: 2px; top: 50%; transform: translateY(-50%); font-size: 13px; font-weight: 600; opacity: .6; transition: opacity .4s; }
.ind .seg { position: absolute; left: 30px; top: 0; bottom: 0; width: 1.5px; background: rgba(255,255,255,.45); overflow: hidden; }
.ind .fill { position: absolute; left: 0; top: 0; width: 100%; height: 0; background: #fff; }
.ind button + button::before { content: ""; position: absolute; top: 0; left: 0; width: 31px; height: 1.5px; background: rgba(255,255,255,.7); }
.ind button.on .num { opacity: 1; }
.ind button.on .fill { animation: fillY var(--dur) linear forwards; }
.ind button.done .fill { height: 100%; }
@keyframes fillY { from { height: 0 } to { height: 100% } }
@keyframes fillX { from { width: 0 } to { width: 100% } }

/* 모바일: 가로로 넘김 */
@media (max-width: 760px) {
  .track { flex-direction: row; }
  .slide::after { background: linear-gradient(0deg, rgba(0,0,0,.35), rgba(0,0,0,0) 30%); }
  .logo { height: min(22vh, 46vw); }
  /* 아래 가운데 동그란 숫자 버튼 1·2·3 */
  .ind {
    left: 50%; right: auto; top: auto; transform: translateX(-50%);
    bottom: calc(28px + env(safe-area-inset-bottom));
    width: auto; height: auto; flex-direction: row; gap: 14px;
  }
  .ind button {
    flex: 0 0 auto; width: 44px; height: 44px; border-radius: 50%;
    border: 1.5px solid rgba(255,255,255,.7); background: rgba(0,0,0,.18);
    -webkit-tap-highlight-color: transparent; touch-action: manipulation;
    transition: background .3s, border-color .3s, color .3s;
  }
  .ind .num { position: static; transform: none; font-size: 15px; opacity: .85; }
  .ind button.on { background: var(--sand, #F4C478); border-color: var(--sand, #F4C478); color: var(--maroon, #50181A); }
  .ind button.on .num { opacity: 1; }
  /* 게이지 선·눈금은 숨김. 자동 넘김 타이머로는 계속 써요(보이지만 않게) */
  .ind button + button::before { display: none; }
  .ind .seg { left: 0; right: 0; top: auto; bottom: 0; width: auto; height: 1px; background: transparent; opacity: 0; pointer-events: none; }
  .ind .fill { height: 100%; width: 0; }
  .ind button.on .fill { animation-name: fillX; }
  .ind button.done .fill { width: 100%; }
}
@media (prefers-reduced-motion: reduce) {
  .track { transition-duration: .01s !important; }
}
</style>
