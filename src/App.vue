<template>
  <div id="app">
    <div class="header">
      <h1>🗺️ 广州市3D地图</h1>
      <p>各区摄像头分布监控</p>
    </div>

    <div class="chart-container">
      <div id="chart" ref="chartRef"></div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: viewMode === '3d' }" @click="setViewMode('3d')">🌍 3D地形</button>
      <button class="btn" :class="{ active: viewMode === '2d' }" @click="setViewMode('2d')">🗺️ 2D平面</button>
      <button class="btn" @click="clearSelection">❌ 清除选择</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import 'echarts-gl'
import guangzhouJson from './guangzhou.json'

const chartRef = ref(null)
let chart = null
const viewMode = ref('3d')

// 广州各区数据（含摄像头数量）
const districtData = [
  { name: '天河区', value: [113.361, 23.125], cameras: 1258, color: '#3b82f6' },
  { name: '越秀区', value: [113.268, 23.129], cameras: 982, color: '#10b981' },
  { name: '海珠区', value: [113.318, 23.083], cameras: 876, color: '#ef4444' },
  { name: '荔湾区', value: [113.244, 23.112], cameras: 654, color: '#f59e0b' },
  { name: '白云区', value: [113.262, 23.168], cameras: 1102, color: '#8b5cf6' },
  { name: '番禺区', value: [113.364, 22.938], cameras: 945, color: '#ec4899' },
  { name: '黄埔区', value: [113.457, 23.177], cameras: 834, color: '#06b6d4' },
  { name: '花都区', value: [113.211, 23.392], cameras: 567, color: '#14b8a6' },
  { name: '南沙区', value: [113.517, 22.801], cameras: 423, color: '#22c55e' },
  { name: '增城区', value: [113.810, 23.302], cameras: 389, color: '#a855f7' },
  { name: '从化区', value: [113.587, 23.546], cameras: 312, color: '#84cc16' },
]

// 选中的区域
const selectedDistrict = ref(null)

function initChart() {
  echarts.registerMap('guangzhou', guangzhouJson)
  chart = echarts.init(chartRef.value)
  updateMap()
  
  // 点击事件
  chart.on('click', (params) => {
    if (params.name) {
      selectedDistrict.value = params.name
      updateMap()
    }
  })
  
  window.addEventListener('resize', () => chart?.resize())
}

function updateMap() {
  const is3D = viewMode.value === '3d'
  
  // 根据选中状态决定旗帜颜色
  const getFlagColor = (name) => {
    if (selectedDistrict.value === name) return '#3b82f6' // 蓝色
    return '#ef4444' // 红色
  }

  if (is3D) {
    const option = {
      backgroundColor: '#0a0a1a',
      
      geo3D: {
        map: 'guangzhou',
        roam: true,
        center: [113.35, 23.15],
        zoom: 2.5,
        
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
            show: true,
            formatter: (params) => params.name,
            color: '#fff',
            fontSize: 12,
            fontWeight: 'bold',
            textShadowColor: '#000',
            textShadowBlur: 4,
            distance: 5,
          }
        },
        
        label: {
          show: true,
          formatter: (params) => params.name,
          color: '#fff',
          fontSize: 10,
          fontWeight: 'normal',
          textShadowColor: '#000',
          textShadowBlur: 3,
          distance: 5,
        },
        
        regions: districtData.map(d => ({
          name: d.name,
          itemStyle: {
            color: d.color,
            opacity: selectedDistrict.value === d.name ? 0.9 : 0.6,
            borderColor: d.color,
            borderWidth: selectedDistrict.value === d.name ? 3 : 2,
            shadowBlur: selectedDistrict.value === d.name ? 15 : 10,
            shadowColor: d.color,
          },
        })),
        
        realisticMaterial: {
          roughness: 0.3,
          metalness: 0.7,
        },
        
        viewControl: {
          projection: 'perspective',
          distance: 80,
          autoRotate: false,
          autoRotateSpeed: 0.8,
          alpha: 45,
          beta: 15,
          center: [0, 0, 0],
          minDistance: 20,
          maxDistance: 200,
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
        borderWidth: 1,
        textStyle: { color: '#fff' },
        formatter: (params) => {
          const district = districtData.find(d => d.name === params.name)
          if (district) {
            return `<div style="font-size:14px"><b>${params.name}</b></div>
                    <div style="color:#fbbf24">📷 摄像头数量: <b>${district.cameras}</b> 个</div>`
          }
          return `<b>${params.name}</b>`
        }
      },

      series: [
        // 旗帜标记
        {
          name: '旗帜',
          type: 'scatter3D',
          coordinateSystem: 'geo3D',
          data: districtData.map(d => ({
            name: d.name,
            value: [d.value[0], d.value[1], 2],
            cameras: d.cameras,
            itemStyle: {
              color: getFlagColor(d.name),
            },
          })),
          symbol: 'path://M12 2L12 22M8 6L12 2L16 6', // 旗帜路径
          symbolSize: 12,
          symbolKeepAspect: true,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => '🚩',
            fontSize: 14,
            distance: 8,
            textStyle: {
              color: getFlagColor(params.data.name),
              fontSize: 16,
            },
          },
          emphasis: {
            scale: 1.5,
          },
          zlevel: 3,
        },
      ],
    }
    chart.setOption(option, true)
  } else {
    const option = {
      backgroundColor: '#0a0a1a',
      geo: {
        map: 'guangzhou',
        roam: true,
        center: [113.35, 23.15],
        zoom: 2.5,
        itemStyle: {
          areaColor: '#1e3a5f',
          borderColor: '#38bdf8',
          borderWidth: 1,
        },
        emphasis: {
          itemStyle: {
            areaColor: '#2563eb',
          },
          label: {
            show: true,
            formatter: (params) => params.name,
            color: '#fff',
            fontSize: 10,
            fontWeight: 'bold',
          }
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
        borderWidth: 1,
        textStyle: { color: '#fff' },
        formatter: (params) => {
          const district = districtData.find(d => d.name === params.name)
          if (district) {
            return `<div style="font-size:14px"><b>${params.name}</b></div>
                    <div style="color:#fbbf24">📷 摄像头数量: <b>${district.cameras}</b> 个</div>`
          }
          return `<b>${params.name}</b>`
        }
      },
      series: [
        // 旗帜标记
        {
          name: '旗帜',
          type: 'scatter',
          coordinateSystem: 'geo',
          data: districtData.map(d => ({
            name: d.name,
            value: d.value,
            cameras: d.cameras,
            itemStyle: {
              color: getFlagColor(d.name),
              shadowBlur: 10,
              shadowColor: getFlagColor(d.name),
            },
          })),
          symbol: 'path://M12 2L12 22M8 6L12 2L16 6',
          symbolSize: 15,
          symbolKeepAspect: true,
          label: {
            show: true,
            position: 'top',
            formatter: (params) => '🚩',
            fontSize: 16,
            distance: 8,
            color: getFlagColor(params.data.name),
          },
          emphasis: {
            scale: 1.5,
          },
          zlevel: 3,
        },
      ],
    }
    chart.setOption(option, true)
  }
}

function setViewMode(mode) {
  viewMode.value = mode
  updateMap()
}

function clearSelection() {
  selectedDistrict.value = null
  updateMap()
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

.chart-container {
  background: rgba(10, 10, 26, 0.95);
  border-radius: 20px;
  border: 1px solid #1e3a5f;
  overflow: hidden;
  box-shadow: 0 0 60px rgba(56, 189, 248, 0.15);
}

#chart {
  width: 100%;
  height: 650px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  justify-content: center;
}

.btn {
  padding: 10px 24px;
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