<template>
  <div id="app">
    <div class="header">
      <h1>🗺️ ECharts 3D 地图 + 标记点</h1>
      <p>通过两层地图叠加解决 3D 模式下无法打点的问题</p>
    </div>

    <div class="solution-box">
      <h2>💡 解决方案</h2>
      <p>
        ECharts GL 的 3D 地图 <code>map3D</code> 系列无法直接添加标记点。
        解决思路：<strong>底层 3D 地图</strong> + <strong>顶层 2D 散点图</strong> 叠加显示。
      </p>
    </div>

    <div class="chart-container">
      <div id="chart" ref="chartRef"></div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D 模式</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D 模式</button>
      <button class="btn" @click="toggleMarkers">📍 切换标记点</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import 'echarts-gl'
import chinaJson from './china.json'

const chartRef = ref(null)
let chart = null
const viewMode = ref('3d')
const markersVisible = ref(true)

// 标记点数据（城市经纬度）
const markerData = [
  { name: '北京', value: [116.46, 39.92, 100] },
  { name: '上海', value: [121.48, 31.22, 100] },
  { name: '广州', value: [113.23, 23.16, 100] },
  { name: '深圳', value: [114.07, 22.62, 100] },
  { name: '杭州', value: [120.19, 30.26, 100] },
  { name: '成都', value: [104.06, 30.67, 100] },
  { name: '武汉', value: [114.31, 30.52, 100] },
  { name: '西安', value: [108.95, 34.27, 100] },
  { name: '南京', value: [118.78, 32.04, 100] },
  { name: '重庆', value: [106.55, 29.56, 100] },
]

function initChart() {
  // 注册中国地图
  echarts.registerMap('china', chinaJson)

  chart = echarts.init(chartRef.value)

  updateChart()

  window.addEventListener('resize', () => chart?.resize())
}

function updateChart() {
  const is3D = viewMode.value === '3d'

  const option = {
    backgroundColor: 'transparent',
    tooltip: {
      trigger: 'item',
      formatter: (params) => {
        if (params.seriesType === 'scatter') {
          return `${params.data.name}<br/>经度: ${params.data.value[0]}<br/>纬度: ${params.data.value[1]}`
        }
        return params.name
      }
    },

    // ========== 底层：地图 ==========
    geo: {
      map: 'china',
      roam: true,
      scaleLimit: { min: 0.8, max: 3 },
      itemStyle: {
        areaColor: '#1d4ed8',
        borderColor: '#38bdf8',
        borderWidth: 1,
      },
      emphasis: {
        itemStyle: {
          areaColor: '#2563eb',
        }
      },
      // 2D 地图的透视效果（3D 感）
      viewControl: is3D ? {
        projection: 'perspective',
        autoRotate: false,
        distance: 80,
        alpha: 40,
        beta: 15,
        center: [0, 0, 0],
        animation: true,
        animationDurationUpdate: 1000,
      } : {
        // 2D 视角
        alpha: 0,
        beta: 0,
        center: [0, 0],
      },
      // 2D 模式下关闭阴影增强扁平感
      light: is3D ? {
        main: {
          intensity: 1.2,
          shadow: true,
          shadowQuality: 'high',
        },
        ambient: {
          intensity: 0.4,
        }
      } : undefined,
    },

    // ========== 顶层：标记点 ==========
    series: markersVisible.value ? [
      {
        name: '标记点',
        type: 'scatter',
        coordinateSystem: 'geo',
        data: markerData,
        symbolSize: (val) => val[2] / 10,
        label: {
          show: true,
          position: 'top',
          formatter: '{b}',
          color: '#fff',
          fontSize: 11,
        },
        itemStyle: {
          color: '#f472b6',
          shadowBlur: 10,
          shadowColor: '#f472b6',
        },
        emphasis: {
          scale: 1.5,
          itemStyle: {
            color: '#ec4899',
            shadowBlur: 20,
            shadowColor: '#ec4899',
          }
        },
        // 标记点动画
        zlevel: 2,
      },
      // 额外：飞线动画效果
      {
        name: '飞线',
        type: 'lines',
        coordinateSystem: 'geo',
        data: [
          { from: [116.46, 39.92], to: [121.48, 31.22] },  // 北京→上海
          { from: [121.48, 31.22], to: [113.23, 23.16] },  // 上海→广州
          { from: [113.23, 23.16], to: [114.07, 22.62] },  // 广州→深圳
          { from: [104.06, 30.67], to: [108.95, 34.27] },  // 成都→西安
        ],
        lineStyle: {
          color: '#38bdf8',
          width: 2,
          curveness: 0.2,
        },
        effect: {
          show: true,
          period: 2,
          trailLength: 0.4,
          color: '#38bdf8',
          symbol: 'circle',
          symbolSize: 3,
        },
        zlevel: 3,
      }
    ] : [],
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
#app {
  padding: 20px;
}
.header {
  text-align: center;
  margin-bottom: 30px;
}
.header h1 {
  font-size: 1.8rem;
  font-weight: 600;
  margin-bottom: 8px;
}
.header p {
  color: #94a3b8;
  font-size: 0.95rem;
}
.solution-box {
  background: linear-gradient(135deg, #1e293b, #334155);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 30px;
  border: 1px solid #475569;
}
.solution-box h2 {
  color: #38bdf8;
  font-size: 1.1rem;
  margin-bottom: 12px;
}
.solution-box p {
  color: #e2e8f0;
  line-height: 1.6;
}
.solution-box code {
  background: #0f172a;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.85rem;
  color: #f472b6;
}
.chart-container {
  background: #1e293b;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #334155;
}
#chart {
  width: 100%;
  height: 600px;
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
  background: #3b82f6;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.95rem;
  transition: all 0.2s;
}
.btn:hover {
  background: #2563eb;
  transform: translateY(-1px);
}
.btn.active {
  background: #10b981;
}
</style>