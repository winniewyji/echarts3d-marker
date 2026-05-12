<template>
  <div id="app">
    <div class="header">
      <h1>🌍 全球AI模型市场份额分布</h1>
      <p>3D地球可视化 · 主要国家AI布局</p>
    </div>

    <div class="main-container">
      <div class="chart-section">
        <div id="chart" ref="chartRef"></div>
      </div>
      <div class="chart-section bar-chart-section">
        <div id="barChart" ref="barChartRef"></div>
      </div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D 视角</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D 视角</button>
      <button class="btn" :class="{ active: showLines }" @click="showLines = !showLines; updateChart()">✈️ {{ showLines ? '隐藏航线' : '显示航线' }}</button>
      <button class="btn" :class="{ active: autoRotate }" @click="autoRotate = !autoRotate; updateChart()">🔄 {{ autoRotate ? '停止旋转' : '自动旋转' }}</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import 'echarts-gl'
import worldJson from './world.json'

const chartRef = ref(null)
const barChartRef = ref(null)
let chart = null
let barChart = null
const viewMode = ref('3d')
const showLines = ref(true)
const autoRotate = ref(true)

// 8个国家国旗数据（位置调整到和参考图一致）
const flagData = [
  { name: '中国', value: [104.0, 35.0], flag: '🇨🇳', rank: 1 },
  { name: '美国', value: [-98.5, 39.8], flag: '🇺🇸', rank: 2 },
  { name: '俄罗斯', value: [55.0, 62.0], flag: '🇷🇺', rank: 3 },
  { name: '日本', value: [138.0, 36.0], flag: '🇯🇵', rank: 4 },
  { name: '韩国', value: [127.0, 37.0], flag: '🇰🇷', rank: 5 },
  { name: '英国', value: [-1.0, 52.0], flag: '🇬🇧', rank: 6 },
  { name: '法国', value: [2.0, 47.0], flag: '🇫🇷', rank: 7 },
  { name: '德国', value: [10.0, 51.0], flag: '🇩🇪', rank: 8 },
]

// AI模型市场份额数据
const marketShare = [
  { name: 'MiniMax', value: 34.2, color: '#38bdf8' },
  { name: 'OpenAI', value: 27.8, color: '#10b981' },
  { name: 'Google', value: 18.5, color: '#f472b6' },
  { name: 'Others', value: 19.5, color: '#94a3b8' },
]

// 连接线数据（从中国出发连接到其他国家）
const connectLines = [
  { from: [104.0, 35.0], to: [-98.5, 39.8] },   // 中国→美国
  { from: [104.0, 35.0], to: [55.0, 62.0] },    // 中国→俄罗斯
  { from: [104.0, 35.0], to: [138.0, 36.0] },   // 中国→日本
  { from: [104.0, 35.0], to: [127.0, 37.0] },   // 中国→韩国
  { from: [104.0, 35.0], to: [-1.0, 52.0] },    // 中国→英国
  { from: [104.0, 35.0], to: [2.0, 47.0] },     // 中国→法国
  { from: [104.0, 35.0], to: [10.0, 51.0] },    // 中国→德国
]

function initChart() {
  echarts.registerMap('world', worldJson)
  chart = echarts.init(chartRef.value)
  barChart = echarts.init(barChartRef.value)
  updateChart()
  updateBarChart()
  window.addEventListener('resize', () => {
    chart?.resize()
    barChart?.resize()
  })
}

function updateChart() {
  const is3D = viewMode.value === '3d'

  const option = {
    backgroundColor: '#0a0a1a',
    
    // ========== 3D地图层 ==========
    geo: {
      map: 'world',
      roam: true,
      scaleLimit: { min: 0.8, max: 8 },
      itemStyle: {
        areaColor: '#1a365d',
        borderColor: '#4299e1',
        borderWidth: 0.5,
        shadowColor: 'rgba(66, 153, 225, 0.5)',
        shadowBlur: 15,
      },
      emphasis: {
        itemStyle: {
          areaColor: '#2c5282',
          borderColor: '#63b3ed',
          borderWidth: 1,
        }
      },
      label: { show: false },
      // 3D透视投影
      viewControl: {
        projection: 'perspective',
        autoRotate: autoRotate.value,
        autoRotateSpeed: 0.5,
        distance: is3D ? 100 : 180,
        alpha: is3D ? 45 : 0,
        beta: is3D ? 15 : 0,
        center: [0, 0, 0],
        minDistance: 30,
        maxDistance: 400,
        animation: true,
        animationDurationUpdate: 1000,
      },
      // 3D光照
      light: is3D ? {
        main: {
          intensity: 1.5,
          shadow: true,
          shadowQuality: 'ultra',
          alpha: 60,
          beta: 40,
        },
        ambient: {
          intensity: 0.5,
        },
        ambientCubemap: {
          enable: true,
          exposure: 0.6,
          diffuseIntensity: 0.4,
        }
      } : undefined,
    },

    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(15, 23, 42, 0.95)',
      borderColor: '#38bdf8',
      borderWidth: 1,
      textStyle: { color: '#fff' },
      formatter: (params) => {
        if (params.data?.flag) {
          return `<div style="font-size:24px">${params.data.flag}</div><b>${params.data.name}</b><br/>AI市场份额第${params.data.rank}位`
        }
        return params.name || ''
      }
    },

    // ========== 国旗+飞线层 ==========
    series: [
      // 国旗散点
      {
        name: '国旗',
        type: 'scatter',
        coordinateSystem: 'geo',
        data: flagData.map(d => ({
          name: d.name,
          value: d.value,
          flag: d.flag,
          rank: d.rank,
        })),
        symbol: 'circle',
        symbolSize: 14,
        label: {
          show: true,
          position: 'top',
          formatter: (params) => params.data.flag,
          fontSize: 18,
          color: '#fff',
          textShadowColor: '#000',
          textShadowBlur: 4,
        },
        itemStyle: {
          color: 'rgba(251, 191, 36, 0.7)',
          shadowBlur: 20,
          shadowColor: 'rgba(251, 191, 36, 0.5)',
        },
        emphasis: {
          scale: 2.5,
          itemStyle: {
            color: '#fbbf24',
            shadowBlur: 30,
            shadowColor: 'rgba(251, 191, 36, 0.8)',
          }
        },
        zlevel: 2,
      },

      // 飞线（带动画点）
      ...(showLines.value ? [{
        name: '航线',
        type: 'lines',
        coordinateSystem: 'geo',
        data: connectLines,
        lineStyle: {
          color: '#38bdf8',
          width: 1.5,
          opacity: 0.6,
          curveness: 0.25,
        },
        effect: {
          show: true,
          period: 2.5,
          trailLength: 0.6,
          color: '#38bdf8',
          symbol: 'circle',
          symbolSize: 4,
        },
        zlevel: 3,
      }] : []),
    ],
  }

  chart.setOption(option, true)
}

function updateBarChart() {
  const option = {
    backgroundColor: 'transparent',
    title: {
      text: 'AI模型市场份额',
      left: 'center',
      top: 10,
      textStyle: {
        color: '#e2e8f0',
        fontSize: 14,
        fontWeight: 600,
      },
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      backgroundColor: 'rgba(15, 23, 42, 0.95)',
      borderColor: '#38bdf8',
      textStyle: { color: '#fff' },
      formatter: (params) => {
        const item = params[0]
        return `${item.name}<br/>份额: <b>${item.value}%</b>`
      },
    },
    grid: {
      left: '15%',
      right: '10%',
      top: 50,
      bottom: 20,
      containLabel: false,
    },
    xAxis: {
      type: 'value',
      max: 40,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: '#94a3b8',
        formatter: '{value}%',
      },
      splitLine: {
        lineStyle: {
          color: '#1e3a5f',
          type: 'dashed',
        },
      },
    },
    yAxis: {
      type: 'category',
      data: marketShare.map(d => d.name),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: '#e2e8f0',
        fontSize: 12,
      },
    },
    series: [
      {
        type: 'bar',
        data: marketShare.map(d => ({
          value: d.value,
          itemStyle: {
            color: d.color,
            borderRadius: [0, 4, 4, 0],
          },
        })),
        barWidth: 16,
        label: {
          show: true,
          position: 'right',
          formatter: '{c}%',
          color: '#e2e8f0',
          fontSize: 12,
        },
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowColor: 'rgba(56, 189, 248, 0.5)',
          }
        },
      },
    ],
  }

  barChart.setOption(option, true)
}

function setViewMode(mode) {
  viewMode.value = mode
  updateChart()
}

onMounted(() => {
  initChart()
})

onUnmounted(() => {
  chart?.dispose()
  barChart?.dispose()
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
  margin-bottom: 20px;
}

.header h1 {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 6px;
  background: linear-gradient(135deg, #38bdf8, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header p {
  color: #94a3b8;
  font-size: 0.85rem;
}

.main-container {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.chart-section {
  background: #0a0a1a;
  border-radius: 16px;
  border: 1px solid #1e3a5f;
  overflow: hidden;
  flex: 1;
  min-width: 300px;
}

#chart {
  width: 100%;
  height: 520px;
}

.bar-chart-section {
  flex: 0 0 280px;
}

#barChart {
  width: 100%;
  height: 520px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  padding: 8px 18px;
  background: #1e3a5f;
  color: #fff;
  border: 1px solid #38bdf8;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: all 0.2s;
}

.btn:hover {
  background: #2d4a7c;
}

.btn.active {
  background: #38bdf8;
  color: #0f172a;
  font-weight: 600;
}
</style>