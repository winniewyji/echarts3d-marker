<template>
  <div id="app">
    <div class="header">
      <h1>🌍 全球AI市场分布</h1>
      <p>3D地形+国家标记+市场份额分析</p>
    </div>

    <div class="main-container">
      <!-- 左侧3D地图 -->
      <div class="map-section">
        <div id="chart" ref="chartRef"></div>
      </div>
      
      <!-- 右侧柱状图 -->
      <div class="bar-section">
        <div id="barChart" ref="barChartRef"></div>
      </div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D地形</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D平面</button>
      <button class="btn" :class="{ active: showLines }" @click="showLines = !showLines; updateMap()">✈️ 连接线</button>
      <button class="btn" :class="{ active: autoRotate }" @click="autoRotate = !autoRotate; updateMap()">🔄 {{ autoRotate ? '停止旋转' : '自动旋转' }}</button>
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

// 主要国家数据（使用首都/重要城市坐标）
const countryData = [
  { name: '中国', value: [116.4, 39.9], flag: '🇨🇳', color: '#3b82f6' },
  { name: '美国', value: [-77.0, 38.9], flag: '🇺🇸', color: '#10b981' },
  { name: '日本', value: [139.7, 35.7], flag: '🇯🇵', color: '#ef4444' },
  { name: '德国', value: [13.4, 52.5], flag: '🇩🇪', color: '#f59e0b' },
  { name: '英国', value: [-0.1, 51.5], flag: '🇬🇧', color: '#8b5cf6' },
  { name: '法国', value: [2.4, 48.8], flag: '🇫🇷', color: '#ec4899' },
  { name: '韩国', value: [126.9, 37.5], flag: '🇰🇷', color: '#06b6d4' },
  { name: '印度', value: [77.1, 28.6], flag: '🇮🇳', color: '#14b8a6' },
]

// AI市场份额
const marketData = [
  { name: 'MiniMax', value: 35, color: '#38bdf8' },
  { name: 'OpenAI', value: 28, color: '#10b981' },
  { name: 'Google', value: 18, color: '#f472b6' },
  { name: 'Anthropic', value: 12, color: '#fbbf24' },
  { name: 'Others', value: 7, color: '#94a3b8' },
]

// 连接线（中国连接其他国家）
const connectLines = countryData.slice(1).map(d => ({
  from: countryData[0].value,
  to: d.value,
}))

function initChart() {
  echarts.registerMap('world', worldJson)
  chart = echarts.init(chartRef.value)
  barChart = echarts.init(barChartRef.value)
  updateMap()
  updateBarChart()
  window.addEventListener('resize', () => {
    chart?.resize()
    barChart?.resize()
  })
}

function updateMap() {
  const is3D = viewMode.value === '3d'

  if (is3D) {
    const option = {
      backgroundColor: 'transparent',
      
      // 3D地图地形效果
      geo3D: {
        map: 'world',
        roam: true,
        center: [30, 20],
        zoom: 1.5,
        
        itemStyle: {
          color: '#0f172a',
          opacity: 1,
          borderColor: '#38bdf8',
          borderWidth: 0.5,
        },
        emphasis: {
          itemStyle: {
            color: '#1e3a5f',
          },
          label: {
            show: false,
          }
        },
        
        // 模拟地形起伏
        regions: countryData.map(d => ({
          name: d.name,
          itemStyle: {
            color: d.color,
            opacity: 0.6,
            borderColor: d.color,
            borderWidth: 2,
            shadowBlur: 15,
            shadowColor: d.color,
          },
        })),
        
        realisticMaterial: {
          roughness: 0.3,
          metalness: 0.8,
        },
        
        viewControl: {
          projection: 'perspective',
          distance: 150,
          autoRotate: autoRotate.value,
          autoRotateSpeed: 0.8,
          alpha: 35,
          beta: 10,
          center: [0, 0, 0],
          minDistance: 50,
          maxDistance: 400,
          animation: true,
          animationDurationUpdate: 1000,
        },
        
        light: {
          main: {
            intensity: 2,
            shadow: false,
            alpha: 60,
            beta: 45,
            color: '#ffffff',
          },
          ambient: {
            intensity: 0.8,
            color: '#38bdf8',
          },
        },
        
        postEffect: {
          enable: true,
          bloom: {
            enable: true,
            intensity: 0.15,
          },
          SSAO: {
            enable: true,
            radius: 3,
            intensity: 1.5,
          },
        },
        
        temporalSuperSampling: {
          enable: true,
        },
      },

      tooltip: {
        trigger: 'item',
        backgroundColor: 'rgba(15, 23, 42, 0.95)',
        borderColor: '#38bdf8',
        textStyle: { color: '#fff' },
        formatter: (params) => {
          if (params.data?.flag) {
            return `<div style="font-size:28px">${params.data.flag}</div><b>${params.data.name}</b>`
          }
          return params.name || ''
        }
      },

      series: [
        // 3D散点（国家标记）
        {
          name: '国家',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: countryData.map(d => ({
            name: d.name,
            value: [d.value[0], d.value[1], 3],
            flag: d.flag,
            itemStyle: { color: d.color },
          })),
          symbol: 'circle',
          symbolSize: 1.8,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.flag,
            fontSize: 18,
            distance: 8,
            textStyle: {
              color: '#fff',
              fontSize: 18,
              borderWidth: 2,
              borderColor: '#000',
            },
          },
          itemStyle: {
            color: 'transparent',
            opacity: 1,
          },
          emphasis: {
            scale: 2.5,
          },
          zlevel: 2,
        },
        // 3D连接线
        ...(showLines.value ? [{
          name: '连接',
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
            period: 2.5,
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
    const option = {
      backgroundColor: 'transparent',
      geo: {
        map: 'world',
        roam: true,
        itemStyle: {
          areaColor: '#1e3a5f',
          borderColor: '#38bdf8',
          borderWidth: 0.5,
        },
        emphasis: {
          itemStyle: {
            areaColor: '#2563eb',
          },
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
            return `<div style="font-size:28px">${params.data.flag}</div><b>${params.data.name}</b>`
          }
          return params.name || ''
        }
      },
      series: [
        {
          name: '国家',
          type: 'scatter',
          coordinateSystem: 'geo',
          data: countryData.map(d => ({
            name: d.name,
            value: d.value,
            flag: d.flag,
            itemStyle: { color: d.color },
          })),
          symbol: 'circle',
          symbolSize: 15,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.flag,
            fontSize: 18,
            color: '#fff',
            textShadowColor: '#000',
            textShadowBlur: 4,
          },
          zlevel: 2,
        },
        ...(showLines.value ? [{
          name: '连接',
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
            period: 2.5,
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
      data: marketData.map(d => d.name),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: '#e2e8f0',
        fontSize: 11,
        margin: 8,
      },
    },
    yAxis3D: {
      type: 'value',
      max: 40,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: { color: '#94a3b8', formatter: '{value}%' },
    },
    zAxis3D: {
      type: 'value',
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: { show: false },
    },

    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(15, 23, 42, 0.95)',
      borderColor: '#38bdf8',
      textStyle: { color: '#fff' },
      formatter: (params) => `<b>${params.name}</b><br/>份额: <b>${params.value}%</b>`,
    },

    grid3D: {
      boxWidth: 45,
      boxDepth: 35,
      boxHeight: 45,
      viewControl: {
        projection: 'perspective',
        autoRotate: false,
        distance: 100,
        alpha: 35,
        beta: 20,
        center: [0, 0, 0],
      },
      light: {
        main: {
          intensity: 1.5,
          shadow: false,
          alpha: 50,
          beta: 40,
        },
        ambient: {
          intensity: 0.5,
        }
      },
      material: {
        roughness: 0.3,
        metalness: 0.6,
      },
      axisPointer: { show: false },
      axisLine: { show: false },
      splitLine: { show: false },
    },

    series: [
      {
        type: 'bar3D',
        data: marketData.map(d => ({
          value: d.value,
          itemStyle: { color: d.color, opacity: 0.9 },
        })),
        shading: 'realistic',
        barWidth: 8,
        barHeight: 8,
        barDepth: 8,
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
            shadowBlur: 20,
            shadowColor: 'rgba(56, 189, 248, 0.5)',
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
  updateMap()
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
  background: #0a0a1a;
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
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 6px;
  background: linear-gradient(135deg, #38bdf8, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header p {
  color: #94a3b8;
  font-size: 0.9rem;
}

.main-container {
  display: flex;
  gap: 20px;
  height: 580px;
}

.map-section {
  flex: 1;
  background: rgba(10, 10, 26, 0.95);
  border-radius: 16px;
  border: 1px solid #1e3a5f;
  overflow: hidden;
  box-shadow: 0 0 40px rgba(56, 189, 248, 0.1);
}

.bar-section {
  width: 280px;
  background: rgba(10, 10, 26, 0.95);
  border-radius: 16px;
  border: 1px solid #1e3a5f;
  overflow: hidden;
}

#chart {
  width: 100%;
  height: 100%;
}

#barChart {
  width: 100%;
  height: 100%;
}

.controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  padding: 10px 22px;
  background: #1e3a5f;
  color: #fff;
  border: 1px solid #38bdf8;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: all 0.2s;
}

.btn:hover {
  background: #2d4a7c;
  transform: translateY(-1px);
}

.btn.active {
  background: #38bdf8;
  color: #0a0a1a;
  font-weight: 600;
}
</style>