<script setup>
import HeroCarousel from '@/components/HeroCarousel.vue'
import photoRock from '@/assets/images/photo-rock.jpg'
import photoShoes from '@/assets/images/photo-shoes.jpg'

// 메인 캐러셀 화면 목록
// - type: 'image' → src 사진이 화면 전체에 깔려요
// - type: 'brand' → 사노리페어 버건디 + 등고선 무늬
// 사진을 바꾸려면 src/assets/images 에 파일을 넣고 위 import 경로만 바꾸면 돼요.
const slides = [
  { type: 'image', src: photoRock, label: '1번 화면' },
  { type: 'image', src: photoShoes, label: '2번 화면' },
  { type: 'brand', label: '3번 화면' }
]
</script>

<template>
  <HeroCarousel :slides="slides" :duration="10000" :move="1100" />
</template>
