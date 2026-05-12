<template>
  <div id="app">
    <div class="header">
      <h1>🌍 ECharts 3D 世界地图 + 国旗标记</h1>
      <p>通过 geo + viewControl.projection 实现 3D 透视效果，国旗通过 scatter 系列打点</p>
    </div>

    <div class="chart-container">
      <div id="chart" ref="chartRef"></div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D 模式</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D 模式</button>
      <button class="btn" :class="{ active: markersVisible }" @click="toggleMarkers">🚩 {{ markersVisible ? '隐藏国旗' : '显示国旗' }}</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import 'echarts-gl'
import worldJson from './world.json'

const chartRef = ref(null)
let chart = null
const viewMode = ref('3d')
const markersVisible = ref(true)

// 国旗标记点数据
const flagData = [
  { name: '中国', value: [104.0, 35.0], flag: '🇨🇳' },
  { name: '美国', value: [-98.5, 39.8], flag: '🇺🇸' },
  { name: '日本', value: [138.0, 36.0], flag: '🇯🇵' },
  { name: '德国', value: [10.0, 51.0], flag: '🇩🇪' },
  { name: '法国', value: [2.0, 47.0], flag: '🇫🇷' },
  { name: '英国', value: [-1.0, 52.0], flag: '🇬🇧' },
  { name: '韩国', value: [127.0, 37.0], flag: '🇰🇷' },
  { name: '印度', value: [78.0, 22.0], flag: '🇮🇳' },
  { name: '俄罗斯', value: [55.0, 62.0], flag: '🇷🇺' },
  { name: '巴西', value: [-52.0, -14.0], flag: '🇧🇷' },
  { name: '澳大利亚', value: [133.0, -27.0], flag: '🇦🇺' },
  { name: '加拿大', value: [-106.0, 56.0], flag: '🇨🇦' },
  { name: '意大利', value: [12.0, 42.0], flag: '🇮🇹' },
  { name: '西班牙', value: [-4.0, 40.0], flag: '🇪🇸' },
  { name: '墨西哥', value: [-102.0, 23.0], flag: '🇲🇽' },
  { name: '阿根廷', value: [-64.0, -34.0], flag: '🇦🇷' },
]

function initChart() {
  echarts.registerMap('world', worldJson)
  chart = echarts.init(chartRef.value)
  updateChart()
  window.addEventListener('resize', () => chart?.resize())
}

function updateChart() {
  const is3D = viewMode.value === '3d'

  const option = {
    backgroundColor: '#0a0a1a',
    
    // ========== 地图层（geo） ==========
    geo: {
      map: 'world',
      roam: true,
      scaleLimit: { min: 0.8, max: 8 },
      itemStyle: {
        areaColor: '#1a365d',
        borderColor: '#4299e1',
        borderWidth: 0.5,
        shadowColor: 'rgba(66, 153, 225, 0.5)',
        shadowBlur: 10,
      },
      emphasis: {
        itemStyle: {
          areaColor: '#2c5282',
          borderColor: '#63b3ed',
          borderWidth: 1,
        }
      },
      label: {
        show: false,
      },
      // 3D 透视投影
      viewControl: {
        projection: 'perspective',
        autoRotate: is3D,
        autoRotateSpeed: 0.3,
        distance: is3D ? 100 : 180,
        alpha: is3D ? 40 : 0,
        beta: is3D ? 15 : 0,
        center: [0, 0, 0],
        minDistance: 30,
        maxDistance: 500,
        animation: true,
        animationDurationUpdate: 800,
      },
      // 3D 光照效果
      light: is3D ? {
        main: {
          intensity: 1.2,
          shadow: true,
          shadowQuality: 'ultra',
          alpha: 50,
          beta: 30,
        },
        ambient: {
          intensity: 0.5,
        },
        ambientCubemap: {
          enable: true,
          exposure: 0.5,
          diffuseIntensity: 0.3,
        }
      } : undefined,
      // 区域渐变效果
      regions: flagData.map(d => ({
        name: d.name,
        itemStyle: {
          areaColor: '#2d4a7c',
        }
      })),
    },

    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(15, 23, 42, 0.9)',
      borderColor: '#38bdf8',
      textStyle: { color: '#fff' },
      formatter: (params) => {
        if (params.data?.flag) {
          return `${params.data.flag} ${params.data.name}`
        }
        return params.name || ''
      }
    },

    // ========== 散点层（国旗） ==========
    series: [
      // 国旗散点
      ...(markersVisible.value ? [{
        name: '国旗',
        type: 'scatter',
        coordinateSystem: 'geo',
        data: flagData.map(d => ({
          name: d.name,
          value: d.value,
          flag: d.flag,
        })),
        symbol: 'circle',
        symbolSize: 12,
        label: {
          show: true,
          position: 'top',
          formatter: (params) => params.data.flag,
          fontSize: 16,
          color: '#fff',
          textShadowColor: '#000',
          textShadowBlur: 3,
        },
        itemStyle: {
          color: 'rgba(251, 191, 36, 0.8)',
          shadowBlur: 15,
          shadowColor: 'rgba(251, 191, 36, 0.6)',
        },
        emphasis: {
          scale: 2,
          itemStyle: {
            color: '#fbbf24',
            shadowBlur: 25,
            shadowColor: 'rgba(251, 191, 36, 0.8)',
          }
        },
        zlevel: 2,
      }] : []),

      // 飞线
      ...(markersVisible.value ? [{
        name: '飞线',
        type: 'lines',
        coordinateSystem: 'geo',
        data: [
          { from: [-98.5, 39.8], to: [104.0, 35.0] },
          { from: [104.0, 35.0], to: [138.0, 36.0] },
          { from: [10.0, 51.0], to: [2.0, 47.0] },
          { from: [-1.0, 52.0], to: [127.0, 37.0] },
          { from: [78.0, 22.0], to: [55.0, 62.0] },
        ],
        lineStyle: {
          color: '#38bdf8',
          width: 1.5,
          opacity: 0.6,
          curveness: 0.2,
        },
        effect: {
          show: true,
          period: 3,
          trailLength: 0.5,
          color: '#38bdf8',
          symbol: 'circle',
          symbolSize: 3,
        },
        zlevel: 3,
      }] : []),
    ],
  }

  chart.setOption(option, true)
}

function setViewMode(mode) {
  viewMode.value = mode
  updateChart()
}

function toggleMarkers() {
  markersVisible.value = !markersVisible.value
  updateChart()
}

onMounted(() => {
  initChart()
})

onUnmounted(() => {
  chart?.dispose()
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #0f172a;
  color: #fff;
  min-height: 100vh;
}

#app {
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 24px;
}

.header h1 {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #38bdf8, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header p {
  color: #94a3b8;
  font-size: 0.9rem;
}

.chart-container {
  background: #0a0a1a;
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid #1e3a5f;
  box-shadow: 0 0 40px rgba(56, 189, 248, 0.1);
}

#chart {
  width: 100%;
  height: 560px;
}

.controls {
  display: flex;
  gap: 12px;
  margin-top: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  padding: 10px 24px;
  background: #1e3a5f;
  color: #fff;
  border: 1px solid #38bdf8;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.btn:hover {
  background: #2d4a7c;
  transform: translateY(-1px);
}

.btn.active {
  background: #38bdf8;
  color: #0f172a;
  font-weight: 600;
}
</style>