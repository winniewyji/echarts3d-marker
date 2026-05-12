<template>
  <div id="app">
    <div class="header">
      <h1>🌍 ECharts 3D 世界地图 + 国旗标记</h1>
      <p>通过 geo3D 实现真实的 3D 地球效果，支持国旗 emoji 标记</p>
    </div>

    <div class="solution-box">
      <h2>💡 解决方案</h2>
      <p>
        使用 ECharts GL 的 <code>geo3D</code> 组件实现真正的 3D 地球效果。<br/>
        国旗通过 <code>scatter3D</code> 系列添加到 3D 地图上方。
      </p>
    </div>

    <div class="chart-container">
      <div id="chart" ref="chartRef"></div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D 模式</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D 模式</button>
      <button class="btn" @click="toggleMarkers">🚩 切换国旗</button>
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

// 国旗标记点数据（国家中心点经纬度）
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

  if (is3D) {
    // ========== 3D 模式：使用 geo3D ==========
    const option = {
      backgroundColor: '#0f172a',
      tooltip: {
        trigger: 'item',
        formatter: (params) => {
          if (params.seriesType === 'scatter3D' && params.data.flag) {
            return `${params.data.flag} ${params.data.name}`
          }
          return params.name
        }
      },

      // geo3D 组件 - 真正的 3D 地球
      geo3D: {
        map: 'world',
        roam: true,
        realisticMaterial: {
          roughness: 0.6,
          metalness: 0.1,
        },
        emphasis: {
          itemStyle: {
            color: '#2563eb',
          }
        },
        itemStyle: {
          color: '#1e3a8a',        // 深蓝色底色
          borderColor: '#38bdf8', // 亮蓝色边框
          borderWidth: 0.5,
          shininess: 20,
          opacity: 0.9,
        },
        light: {
          main: {
            intensity: 1.5,
            shadow: true,
            shadowQuality: 'high',
            alpha: 40,
            beta: 30,
          },
          ambient: {
            intensity: 0.4,
          },
          ambientCubemap: {
            exposure: 1,
            diffuseIntensity: 0.5,
          }
        },
        viewControl: {
          projection: 'perspective',
          autoRotate: true,
          autoRotateSpeed: 0.5,
          distance: 120,
          alpha: 40,
          beta: 10,
          center: [0, 0, 0],
          minDistance: 50,
          maxDistance: 300,
          animation: true,
          animationDurationUpdate: 1000,
        },
        groundPlane: {
          show: false,
        },
        postEffect: {
          enable: true,
          bloom: {
            enable: true,
            intensity: 0.1,
          },
          SSAO: {
            enable: true,
            radius: 2,
            intensity: 1,
          },
        },
        temporalSuperSampling: {
          enable: true,
        },
      },

      // 3D 散点标记
      series: markersVisible.value ? [
        {
          name: '国旗',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: flagData.map(d => ({
            name: d.name,
            value: [d.value[0], d.value[1], 1], // [lng, lat, altitude]
            flag: d.flag,
          })),
          symbol: 'circle',
          symbolSize: 8,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.flag || '',
            fontSize: 18,
            distance: 5,
            textStyle: {
              color: '#ffffff',
              fontSize: 18,
              borderWidth: 1,
              borderColor: '#000',
              backgroundColor: 'transparent',
            },
          },
          itemStyle: {
            color: '#f472b6',
            opacity: 1,
          },
          emphasis: {
            scale: 2,
          },
        },
      ] : [],
    }

    chart.setOption(option, true)
  } else {
    // ========== 2D 模式：使用普通 geo ==========
    const option = {
      backgroundColor: '#0f172a',
      tooltip: {
        trigger: 'item',
        formatter: (params) => {
          if (params.seriesType === 'scatter' && params.data.flag) {
            return `${params.data.flag} ${params.data.name}`
          }
          return params.name
        }
      },

      // 普通 2D 地图
      geo: {
        map: 'world',
        roam: true,
        scaleLimit: { min: 0.8, max: 5 },
        itemStyle: {
          areaColor: '#1e3a8a',
          borderColor: '#38bdf8',
          borderWidth: 0.5,
        },
        emphasis: {
          itemStyle: {
            areaColor: '#2563eb',
          }
        },
        label: {
          show: false,
        },
      },

      // 2D 散点标记
      series: markersVisible.value ? [
        {
          name: '国旗',
          type: 'scatter',
          coordinateSystem: 'geo',
          data: flagData.map(d => ({
            name: d.name,
            value: d.value,
            flag: d.flag,
          })),
          symbol: 'circle',
          symbolSize: 14,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => params.data.flag,
            fontSize: 18,
          },
          itemStyle: {
            color: 'rgba(255,255,255,0.1)',
          },
          emphasis: {
            scale: 1.5,
          },
        },
        // 飞线
        {
          name: '飞线',
          type: 'lines',
          coordinateSystem: 'geo',
          data: [
            { from: [-98.5, 39.8], to: [104.0, 35.0] },
            { from: [104.0, 35.0], to: [138.0, 36.0] },
            { from: [10.0, 51.0], to: [2.0, 47.0] },
          ],
          lineStyle: {
            color: '#38bdf8',
            width: 1.5,
            curveness: 0.2,
          },
          effect: {
            show: true,
            period: 4,
            trailLength: 0.3,
            color: '#38bdf8',
            symbol: 'circle',
            symbolSize: 2,
          },
        },
      ] : [],
    }

    chart.setOption(option, true)
  }
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
  background: #0f172a;
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