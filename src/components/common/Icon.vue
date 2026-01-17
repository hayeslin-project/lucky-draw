<template>
  <svg
    :class="['icon', sizeClass]"
    :viewBox="viewBox"
    fill="none"
    stroke="currentColor"
    stroke-width="2"
    stroke-linecap="round"
    stroke-linejoin="round"
  >
    <component :is="iconComponent" />
  </svg>
</template>

<script setup lang="ts">
import { computed, h, type FunctionalComponent } from 'vue'

const props = defineProps<{
  name: string
  size?: 'sm' | 'md' | 'lg' | 'xl' | '2xl'
}>()

const viewBox = '0 0 24 24'

const sizeClass = computed(() => {
  const sizes = {
    sm: 'w-4 h-4',
    md: 'w-5 h-5',
    lg: 'w-6 h-6',
    xl: 'w-8 h-8',
    '2xl': 'w-10 h-10',
  }
  return sizes[props.size || 'md']
})

// 图标路径定义
const icons: Record<string, FunctionalComponent> = {
  // 导航图标
  'clipboard-list': () => h('g', [
    h('path', { d: 'M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2' }),
    h('rect', { x: '9', y: '3', width: '6', height: '4', rx: '2' }),
    h('line', { x1: '9', y1: '12', x2: '15', y2: '12' }),
    h('line', { x1: '9', y1: '16', x2: '15', y2: '16' }),
  ]),
  'settings': () => h('g', [
    h('circle', { cx: '12', cy: '12', r: '3' }),
    h('path', { d: 'M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 010 2.83 2 2 0 01-2.83 0l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 01-2 2 2 2 0 01-2-2v-.09A1.65 1.65 0 009 19.4a1.65 1.65 0 00-1.82.33l-.06.06a2 2 0 01-2.83 0 2 2 0 010-2.83l.06-.06a1.65 1.65 0 00.33-1.82 1.65 1.65 0 00-1.51-1H3a2 2 0 01-2-2 2 2 0 012-2h.09A1.65 1.65 0 004.6 9a1.65 1.65 0 00-.33-1.82l-.06-.06a2 2 0 010-2.83 2 2 0 012.83 0l.06.06a1.65 1.65 0 001.82.33H9a1.65 1.65 0 001-1.51V3a2 2 0 012-2 2 2 0 012 2v.09a1.65 1.65 0 001 1.51 1.65 1.65 0 001.82-.33l.06-.06a2 2 0 012.83 0 2 2 0 010 2.83l-.06.06a1.65 1.65 0 00-.33 1.82V9a1.65 1.65 0 001.51 1H21a2 2 0 012 2 2 2 0 01-2 2h-.09a1.65 1.65 0 00-1.51 1z' }),
  ]),
  'dice': () => h('g', [
    h('rect', { x: '2', y: '2', width: '20', height: '20', rx: '2.18', ry: '2.18' }),
    h('circle', { cx: '8', cy: '8', r: '1.5', fill: 'currentColor', stroke: 'none' }),
    h('circle', { cx: '16', cy: '8', r: '1.5', fill: 'currentColor', stroke: 'none' }),
    h('circle', { cx: '8', cy: '16', r: '1.5', fill: 'currentColor', stroke: 'none' }),
    h('circle', { cx: '16', cy: '16', r: '1.5', fill: 'currentColor', stroke: 'none' }),
    h('circle', { cx: '12', cy: '12', r: '1.5', fill: 'currentColor', stroke: 'none' }),
  ]),
  'trophy': () => h('g', [
    h('path', { d: 'M6 9H4.5a2.5 2.5 0 010-5H6' }),
    h('path', { d: 'M18 9h1.5a2.5 2.5 0 000-5H18' }),
    h('path', { d: 'M4 22h16' }),
    h('path', { d: 'M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22' }),
    h('path', { d: 'M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22' }),
    h('path', { d: 'M18 2H6v7a6 6 0 0012 0V2z' }),
  ]),
  // 操作图标
  'play': () => h('polygon', { points: '5 3 19 12 5 21 5 3', fill: 'currentColor', stroke: 'none' }),
  'pause': () => h('g', [
    h('rect', { x: '6', y: '4', width: '4', height: '16', fill: 'currentColor', stroke: 'none' }),
    h('rect', { x: '14', y: '4', width: '4', height: '16', fill: 'currentColor', stroke: 'none' }),
  ]),
  'check': () => h('polyline', { points: '20 6 9 17 4 12' }),
  'x': () => h('g', [
    h('line', { x1: '18', y1: '6', x2: '6', y2: '18' }),
    h('line', { x1: '6', y1: '6', x2: '18', y2: '18' }),
  ]),
  'plus': () => h('g', [
    h('line', { x1: '12', y1: '5', x2: '12', y2: '19' }),
    h('line', { x1: '5', y1: '12', x2: '19', y2: '12' }),
  ]),
  'minus': () => h('line', { x1: '5', y1: '12', x2: '19', y2: '12' }),
  'trash': () => h('g', [
    h('polyline', { points: '3 6 5 6 21 6' }),
    h('path', { d: 'M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a2 2 0 012-2h4a2 2 0 012 2v2' }),
  ]),
  // 文件图标
  'folder': () => h('path', { d: 'M22 19a2 2 0 01-2 2H4a2 2 0 01-2-2V5a2 2 0 012-2h5l2 3h9a2 2 0 012 2z' }),
  'file-text': () => h('g', [
    h('path', { d: 'M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z' }),
    h('polyline', { points: '14 2 14 8 20 8' }),
    h('line', { x1: '16', y1: '13', x2: '8', y2: '13' }),
    h('line', { x1: '16', y1: '17', x2: '8', y2: '17' }),
    h('polyline', { points: '10 9 9 9 8 9' }),
  ]),
  'download': () => h('g', [
    h('path', { d: 'M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4' }),
    h('polyline', { points: '7 10 12 15 17 10' }),
    h('line', { x1: '12', y1: '15', x2: '12', y2: '3' }),
  ]),
  'upload': () => h('g', [
    h('path', { d: 'M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4' }),
    h('polyline', { points: '17 8 12 3 7 8' }),
    h('line', { x1: '12', y1: '3', x2: '12', y2: '15' }),
  ]),
  'table': () => h('g', [
    h('path', { d: 'M9 3H5a2 2 0 00-2 2v4m6-6h10a2 2 0 012 2v4M9 3v18m0 0h10a2 2 0 002-2V9M9 21H5a2 2 0 01-2-2V9m0 0h18' }),
  ]),
  // 用户图标
  'users': () => h('g', [
    h('path', { d: 'M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2' }),
    h('circle', { cx: '9', cy: '7', r: '4' }),
    h('path', { d: 'M23 21v-2a4 4 0 00-3-3.87' }),
    h('path', { d: 'M16 3.13a4 4 0 010 7.75' }),
  ]),
  'user': () => h('g', [
    h('path', { d: 'M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2' }),
    h('circle', { cx: '12', cy: '7', r: '4' }),
  ]),
  // 状态图标
  'party': () => h('g', [
    h('path', { d: 'M5.8 11.3L2 22l10.7-3.79' }),
    h('path', { d: 'M4 3h.01' }),
    h('path', { d: 'M22 8h.01' }),
    h('path', { d: 'M15 2h.01' }),
    h('path', { d: 'M22 20h.01' }),
    h('path', { d: 'M22 2l-2.24.75a2.9 2.9 0 00-1.96 3.12v0c.1.86-.57 1.63-1.45 1.63h-.38c-.86 0-1.6.6-1.76 1.44L14 10' }),
    h('path', { d: 'M22 13l-2.24-.75a2.9 2.9 0 01-1.96-3.12v0c-.1-.86.57-1.63 1.45-1.63h.38c.86 0 1.6-.6 1.76-1.44L22 5' }),
  ]),
  'sparkles': () => h('g', [
    h('path', { d: 'M12 3l1.912 5.813a2 2 0 001.275 1.275L21 12l-5.813 1.912a2 2 0 00-1.275 1.275L12 21l-1.912-5.813a2 2 0 00-1.275-1.275L3 12l5.813-1.912a2 2 0 001.275-1.275L12 3z' }),
    h('path', { d: 'M5 3v4' }),
    h('path', { d: 'M3 5h4' }),
    h('path', { d: 'M19 17v4' }),
    h('path', { d: 'M17 19h4' }),
  ]),
  'scale': () => h('g', [
    h('path', { d: 'M16 16l3-8 3 8c-.87.65-1.92 1-3 1s-2.13-.35-3-1z' }),
    h('path', { d: 'M2 16l3-8 3 8c-.87.65-1.92 1-3 1s-2.13-.35-3-1z' }),
    h('path', { d: 'M7 21h10' }),
    h('path', { d: 'M12 3v18' }),
    h('path', { d: 'M3 7h2c2 0 5-1 7-2 2 1 5 2 7 2h2' }),
  ]),
  'clock': () => h('g', [
    h('circle', { cx: '12', cy: '12', r: '10' }),
    h('polyline', { points: '12 6 12 12 16 14' }),
  ]),
  'phone': () => h('path', { d: 'M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07 19.5 19.5 0 01-6-6 19.79 19.79 0 01-3.07-8.67A2 2 0 014.11 2h3a2 2 0 012 1.72 12.84 12.84 0 00.7 2.81 2 2 0 01-.45 2.11L8.09 9.91a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45 12.84 12.84 0 002.81.7A2 2 0 0122 16.92z' }),
  'building': () => h('g', [
    h('rect', { x: '4', y: '2', width: '16', height: '20', rx: '2', ry: '2' }),
    h('path', { d: 'M9 22v-4h6v4' }),
    h('path', { d: 'M8 6h.01' }),
    h('path', { d: 'M16 6h.01' }),
    h('path', { d: 'M12 6h.01' }),
    h('path', { d: 'M12 10h.01' }),
    h('path', { d: 'M12 14h.01' }),
    h('path', { d: 'M16 10h.01' }),
    h('path', { d: 'M16 14h.01' }),
    h('path', { d: 'M8 10h.01' }),
    h('path', { d: 'M8 14h.01' }),
  ]),
  'id-card': () => h('g', [
    h('rect', { x: '2', y: '5', width: '20', height: '14', rx: '2' }),
    h('line', { x1: '2', y1: '10', x2: '22', y2: '10' }),
  ]),
  'search': () => h('g', [
    h('circle', { cx: '11', cy: '11', r: '8' }),
    h('line', { x1: '21', y1: '21', x2: '16.65', y2: '16.65' }),
  ]),
  'save': () => h('g', [
    h('path', { d: 'M19 21H5a2 2 0 01-2-2V5a2 2 0 012-2h11l5 5v11a2 2 0 01-2 2z' }),
    h('polyline', { points: '17 21 17 13 7 13 7 21' }),
    h('polyline', { points: '7 3 7 8 15 8' }),
  ]),
  'history': () => h('g', [
    h('path', { d: 'M3 3v5h5' }),
    h('path', { d: 'M3.05 13A9 9 0 106 5.3L3 8' }),
    h('path', { d: 'M12 7v5l4 2' }),
  ]),
  'eye': () => h('g', [
    h('path', { d: 'M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z' }),
    h('circle', { cx: '12', cy: '12', r: '3' }),
  ]),
  'help-circle': () => h('g', [
    h('circle', { cx: '12', cy: '12', r: '10' }),
    h('path', { d: 'M9.09 9a3 3 0 015.83 1c0 2-3 3-3 3' }),
    h('line', { x1: '12', y1: '17', x2: '12.01', y2: '17' }),
  ]),
  'inbox': () => h('g', [
    h('polyline', { points: '22 12 16 12 14 15 10 15 8 12 2 12' }),
    h('path', { d: 'M5.45 5.11L2 12v6a2 2 0 002 2h16a2 2 0 002-2v-6l-3.45-6.89A2 2 0 0016.76 4H7.24a2 2 0 00-1.79 1.11z' }),
  ]),
  'volume-2': () => h('g', [
    h('polygon', { points: '11 5 6 9 2 9 2 15 6 15 11 19 11 5' }),
    h('path', { d: 'M19.07 4.93a10 10 0 010 14.14' }),
    h('path', { d: 'M15.54 8.46a5 5 0 010 7.07' }),
  ]),
  'question': () => h('g', [
    h('circle', { cx: '12', cy: '12', r: '1', fill: 'currentColor', stroke: 'none' }),
    h('path', { d: 'M12 2C6.5 2 2 6.5 2 12s4.5 10 10 10 10-4.5 10-10S17.5 2 12 2zm0 16c-.6 0-1-.4-1-1s.4-1 1-1 1 .4 1 1-.4 1-1 1zm1-4.8V14h-2v-2c0-.6.4-1 1-1 1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2H8c0-2.2 1.8-4 4-4s4 1.8 4 4c0 1.9-1.3 3.4-3 3.8v.2z' }),
  ]),
}

const iconComponent = computed(() => {
  return icons[props.name] || icons['help-circle']
})
</script>

<style scoped>
.icon {
  display: inline-block;
  flex-shrink: 0;
}

.w-4 { width: 1rem; }
.h-4 { height: 1rem; }
.w-5 { width: 1.25rem; }
.h-5 { height: 1.25rem; }
.w-6 { width: 1.5rem; }
.h-6 { height: 1.5rem; }
.w-8 { width: 2rem; }
.h-8 { height: 2rem; }
.w-10 { width: 2.5rem; }
.h-10 { height: 2.5rem; }
</style>
