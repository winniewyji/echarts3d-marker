<template>
  <div id="app">
    <div class="header">
      <h1>🌆 广州AI市场分布</h1>
      <p>3D地形+区域标记+市场份额分析</p>
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
import guangzhouJson from './guangzhou.json'

const chartRef = ref(null)
const barChartRef = ref(null)
let chart = null
let barChart = null
const viewMode = ref('3d')
const showLines = ref(true)
const autoRotate = ref(true)

// 广州各区数据
const districtData = [
  { name: '天河区', value: [113.361, 23.125], icon: '🏢', color: '#3b82f6' },
  { name: '越秀区', value: [113.268, 23.129], icon: '🏛️', color: '#10b981' },
  { name: '海珠区', value: [113.318, 23.083], icon: '🏠', color: '#ef4444' },
  { name: '荔湾区', value: [113.244, 23.112], icon: '🏘️', color: '#f59e0b' },
  { name: '白云区', value: [113.262, 23.168], icon: '🏔️', color: '#8b5cf6' },
  { name: '番禺区', value: [113.364, 22.938], icon: '🏗️', color: '#ec4899' },
  { name: '黄埔区', value: [113.457, 23.177], icon: '🏭', color: '#06b6d4' },
  { name: '花都区', value: [113.211, 23.392], icon: '🌳', color: '#14b8a6' },
  { name: '南沙区', value: [113.517, 22.801], icon: '🌊', color: '#22c55e' },
  { name: '增城区', value: [113.810, 23.302], icon: '🏞️', color: '#a855f7' },
  { name: '从化区', value: [113.587, 23.546], icon: '⛰️', color: '#84cc16' },
]

// 广州塔坐标
const cantonTower = { name: '广州塔', value: [113.3238, 23.1086], icon: '🗼', color: '#fbbf24' }

// 各区AI市场份额
const marketData = [
  { name: '天河', value: 35, color: '#3b82f6' },
  { name: '黄埔', value: 22, color: '#10b981' },
  { name: '番禺', value: 15, color: '#ef4444' },
  { name: '白云', value: 12, color: '#f59e0b' },
  { name: '其他', value: 16, color: '#94a3b8' },
]

// 连接线（从广州塔连接各区）
const connectLines = districtData.slice(0, 6).map(d => ({
  from: cantonTower.value,
  to: d.value,
}))

function initChart() {
  echarts.registerMap('guangzhou', guangzhouJson)
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
        map: 'guangzhou',
        roam: true,
        center: [113.35, 23.15],
        zoom: 1.8,
        
        itemStyle: {
          color: '#1e3a5f',
          opacity: 1,
          borderColor: '#38bdf8',
          borderWidth: 1,
        },
        emphasis: {
          itemStyle: {
            color: '#2563eb',
          },
          label: {
            show: false,
          }
        },
        
        // 区域着色
        regions: districtData.map(d => ({
          name: d.name,
          itemStyle: {
            color: d.color,
            opacity: 0.5,
            borderColor: d.color,
            borderWidth: 2,
            shadowBlur: 10,
            shadowColor: d.color,
          },
        })),
        
        realisticMaterial: {
          roughness: 0.3,
          metalness: 0.7,
        },
        
        viewControl: {
          projection: 'perspective',
          distance: 100,
          autoRotate: autoRotate.value,
          autoRotateSpeed: 0.8,
          alpha: 40,
          beta: 15,
          center: [0, 0, 0],
          minDistance: 30,
          maxDistance: 250,
          animation: true,
          animationDurationUpdate: 1000,
        },
        
        light: {
          main: {
            intensity: 1.8,
            shadow: false,
            alpha: 55,
            beta: 40,
            color: '#ffffff',
          },
          ambient: {
            intensity: 0.6,
            color: '#38bdf8',
          },
        },
        
        postEffect: {
          enable: true,
          bloom: {
            enable: true,
            intensity: 0.12,
          },
          SSAO: {
            enable: true,
            radius: 2,
            intensity: 1.2,
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
          if (params.data?.icon) {
            return `<div style="font-size:28px">${params.data.icon}</div><b>${params.data.name}</b>`
          }
          return params.name || ''
        }
      },

      series: [
        // 3D散点（广州塔）
        {
          name: '广州塔',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: [{
            name: cantonTower.name,
            value: [cantonTower.value[0], cantonTower.value[1], 4],
            icon: cantonTower.icon,
          }],
          symbol: 'circle',
          symbolSize: 2.5,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.icon,
            fontSize: 22,
            distance: 10,
            textStyle: {
              color: '#fbbf24',
              fontSize: 22,
            },
          },
          itemStyle: {
            color: '#fbbf24',
            shadowBlur: 30,
            shadowColor: '#fbbf24',
            opacity: 1,
          },
          emphasis: {
            scale: 2,
          },
          zlevel: 3,
        },
        // 3D散点（各区）
        {
          name: '区域',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: districtData.map(d => ({
            name: d.name,
            value: [d.value[0], d.value[1], 2],
            icon: d.icon,
          })),
          symbol: 'circle',
          symbolSize: 1.5,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.icon,
            fontSize: 16,
            distance: 6,
            textStyle: {
              color: '#fff',
              fontSize: 16,
            },
          },
          itemStyle: {
            color: 'transparent',
            opacity: 1,
          },
          emphasis: {
            scale: 2,
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
            width: 1.2,
            opacity: 0.6,
            curveness: 0.15,
          },
          effect: {
            show: true,
            period: 2.5,
            trailLength: 0.4,
            color: '#38bdf8',
            symbol: 'circle',
            symbolSize: 2.5,
          },
          zlevel: 4,
        }] : []),
      ],
    }
    chart.setOption(option, true)
  } else {
    const option = {
      backgroundColor: 'transparent',
      geo: {
        map: 'guangzhou',
        roam: true,
        center: [113.35, 23.15],
        zoom: 1.8,
        itemStyle: {
          areaColor: '#1e3a5f',
          borderColor: '#38bdf8',
          borderWidth: 1,
        },
        emphasis: {
          itemStyle: {
            areaColor: '#2563eb',
          },
        },
        label: {
          show: true,
          formatter: (params) => params.name,
          color: '#fff',
          fontSize: 9,
        },
      },
      tooltip: {
        trigger: 'item',
        backgroundColor: 'rgba(15, 23, 42, 0.95)',
        borderColor: '#38bdf8',
        textStyle: { color: '#fff' },
        formatter: (params) => {
          if (params.data?.icon) {
            return `<div style="font-size:28px">${params.data.icon}</div><b>${params.data.name}</b>`
          }
          return params.name || ''
        }
      },
      series: [
        // 广州塔
        {
          name: '广州塔',
          type: 'scatter',
          coordinateSystem: 'geo',
          data: [{
            name: cantonTower.name,
            value: cantonTower.value,
            icon: cantonTower.icon,
          }],
          symbol: 'circle',
          symbolSize: 20,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.icon,
            fontSize: 22,
            color: '#fbbf24',
          },
          itemStyle: {
            color: '#fbbf24',
            shadowBlur: 30,
            shadowColor: '#fbbf24',
          },
          zlevel: 4,
        },
        // 各区
        {
          name: '区域',
          type: 'scatter',
          coordinateSystem: 'geo',
          data: districtData.map(d => ({
            name: d.name,
            value: d.value,
            icon: d.icon,
          })),
          symbol: 'circle',
          symbolSize: 12,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.icon,
            fontSize: 14,
            color: '#fff',
            textShadowColor: '#000',
            textShadowBlur: 3,
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
            width: 1.2,
            opacity: 0.6,
            curveness: 0.15,
          },
          effect: {
            show: true,
            period: 2.5,
            trailLength: 0.4,
            color: '#38bdf8',
            symbol: 'circle',
            symbolSize: 2.5,
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