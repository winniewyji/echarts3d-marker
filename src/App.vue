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

const marketShare = [
  { name: 'MiniMax', value: 34.2, color: '#38bdf8' },
  { name: 'OpenAI', value: 27.8, color: '#10b981' },
  { name: 'Google', value: 18.5, color: '#f472b6' },
  { name: 'Others', value: 19.5, color: '#94a3b8' },
]

const connectLines = [
  { from: [104.0, 35.0], to: [-98.5, 39.8] },
  { from: [104.0, 35.0], to: [55.0, 62.0] },
  { from: [104.0, 35.0], to: [138.0, 36.0] },
  { from: [104.0, 35.0], to: [127.0, 37.0] },
  { from: [104.0, 35.0], to: [-1.0, 52.0] },
  { from: [104.0, 35.0], to: [2.0, 47.0] },
  { from: [104.0, 35.0], to: [10.0, 51.0] },
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

  if (is3D) {
    // ========== 真正的 3D 地球模式 ==========
    const option = {
      backgroundColor: '#0a0a1a',
      
      // 真正的 3D 地球组件
      geo3D: {
        map: 'world',
        roam: true,
        center: [0, 0],
        itemStyle: {
          color: '#1a365d',
          opacity: 1,
          borderColor: '#4299e1',
          borderWidth: 0.5,
        },
        emphasis: {
          itemStyle: {
            color: '#2c5282',
          }
        },
        label: {
          show: false,
        },
        realisticMaterial: {
          roughness: 0.6,
          metalness: 0.1,
        },
        viewControl: {
          distance: 120,
          autoRotate: autoRotate.value,
          autoRotateSpeed: 0.8,
          alpha: 40,
          beta: 10,
          minDistance: 50,
          maxDistance: 250,
          animation: true,
          animationDurationUpdate: 1000,
        },
        light: {
          main: {
            intensity: 1.5,
            shadow: false,
            alpha: 50,
            beta: 30,
          },
          ambient: {
            intensity: 0.4,
          },
          ambientCubemap: {
            enable: false,
          }
        },
        postEffect: {
          enable: true,
          bloom: {
            enable: true,
            intensity: 0.05,
          },
          SSAO: {
            enable: true,
            radius: 1,
            intensity: 0.5,
          },
        },
        temporalSuperSampling: {
          enable: false,
        },
      },

      tooltip: {
        trigger: 'item',
        backgroundColor: 'rgba(15, 23, 42, 0.95)',
        borderColor: '#38bdf8',
        textStyle: { color: '#fff' },
        formatter: (params) => {
          if (params.data?.flag) {
            return `<div style="font-size:24px">${params.data.flag}</div><b>${params.data.name}</b><br/>AI市场份额第${params.data.rank}位`
          }
          return params.name || ''
        }
      },

      series: [
        // 3D 散点（国旗）
        {
          name: '国旗',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: flagData.map(d => ({
            name: d.name,
            value: [d.value[0], d.value[1], 2],
            flag: d.flag,
            rank: d.rank,
          })),
          symbol: 'circle',
          symbolSize: 1.5,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.flag,
            fontSize: 16,
            distance: 5,
            textStyle: {
              color: '#fff',
              fontSize: 16,
              backgroundColor: 'transparent',
            },
          },
          itemStyle: {
            color: '#fbbf24',
            opacity: 1,
          },
          emphasis: {
            scale: 2,
          },
          zlevel: 2,
        },
        // 3D 飞线
        ...(showLines.value ? [{
          name: '航线',
          type: 'lines3D',
          coordinateSystem: 'geo3D',
          data: connectLines.map(line => ({
            coords: [line.from, line.to],
          })),
          lineStyle: {
            color: '#38bdf8',
            width: 1.5,
            opacity: 0.7,
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
  } else {
    // ========== 2D 地图模式 ==========
    const option = {
      backgroundColor: '#0a0a1a',
      geo: {
        map: 'world',
        roam: true,
        scaleLimit: { min: 0.8, max: 8 },
        itemStyle: {
          areaColor: '#1a365d',
          borderColor: '#4299e1',
          borderWidth: 0.5,
        },
        emphasis: {
          itemStyle: {
            areaColor: '#2c5282',
            borderColor: '#63b3ed',
          }
        },
        label: { show: false },
      },
      tooltip: {
        trigger: 'item',
        backgroundColor: 'rgba(15, 23, 42, 0.95)',
        borderColor: '#38bdf8',
        textStyle: { color: '#fff' },
        formatter: (params) => {
          if (params.data?.flag) {
            return `<div style="font-size:24px">${params.data.flag}</div><b>${params.data.name}</b>`
          }
          return params.name || ''
        }
      },
      series: [
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
            color: 'rgba(251, 191, 36, 0.8)',
          },
          zlevel: 2,
        },
        ...(showLines.value ? [{
          name: '航线',
          type: 'lines',
          coordinateSystem: 'geo',
          data: connectLines,
          lineStyle: {
            color: '#38bdf8',
            width: 1.5,
            opacity: 0.7,
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
}

function updateBarChart() {
  const option = {
    backgroundColor: 'transparent',
    xAxis3D: {
      type: 'category',
      data: marketShare.map(d => d.name),
      axisLine: { lineStyle: { color: '#38bdf8' } },
      axisTick: { show: false },
      axisLabel: {
        color: '#e2e8f0',
        fontSize: 11,
        margin: 6,
      },
    },
    yAxis3D: {
      type: 'value',
      max: 40,
      axisLine: { lineStyle: { color: '#38bdf8' } },
      axisTick: { show: false },
      axisLabel: { color: '#94a3b8', formatter: '{value}%' },
    },
    zAxis3D: {
      type: 'value',
      axisLine: { lineStyle: { color: '#38bdf8' } },
      axisTick: { show: false },
      axisLabel: { show: false },
    },
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(15, 23, 42, 0.95)',
      borderColor: '#38bdf8',
      textStyle: { color: '#fff' },
      formatter: (params) => `<b>${params.name}</b><br/>市场份额: <b>${params.value}%</b>`,
    },
    grid3D: {
      boxWidth: 50,
      boxDepth: 40,
      boxHeight: 50,
      viewControl: {
        projection: 'perspective',
        autoRotate: false,
        distance: 120,
        alpha: 35,
        beta: 20,
        center: [0, 0, 0],
      },
      light: {
        main: {
          intensity: 1.2,
          shadow: false,
          alpha: 45,
          beta: 35,
        },
        ambient: {
          intensity: 0.5,
        }
      },
      material: {
        roughness: 0.5,
        metalness: 0.05,
      },
      axisPointer: { show: false },
      axisLine: { lineStyle: { color: '#1e3a5f' } },
      splitLine: { show: false },
    },
    series: [
      {
        type: 'bar3D',
        data: marketShare.map(d => ({
          value: d.value,
          itemStyle: { color: d.color, opacity: 0.9 },
        })),
        shading: 'realistic',
        barWidth: 10,
        barHeight: 10,
        barDepth: 10,
        label: {
          show: true,
          position: 'top',
          formatter: (params) => `${params.name}\n${params.value}%`,
          color: '#fff',
          fontSize: 10,
          textShadowColor: '#000',
          textShadowBlur: 2,
          distance: 5,
        },
        emphasis: {
          itemStyle: {
            opacity: 1,
            shadowBlur: 15,
            shadowColor: 'rgba(56, 189, 248, 0.4)',
          }
        },
        animationDurationUpdate: 1500,
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
  background: rgba(10, 10, 26, 0.9);
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
  flex: 0 0 260px;
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