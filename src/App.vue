<template>
  <div id="app">
    <div class="header">
      <h1>🗺️ 广州市3D地图</h1>
      <p>广州各区3D可视化分布图</p>
    </div>

    <div class="chart-container">
      <div id="chart" ref="chartRef"></div>
    </div>

    <div class="controls">
      <button class="btn" :class="{ active: autoRotate }" @click="autoRotate = !autoRotate; updateChart()">🔄 {{ autoRotate ? '停止旋转' : '自动旋转' }}</button>
      <button class="btn" @click="resetView">🎯 重置视角</button>
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
const autoRotate = ref(true)

function initChart() {
  echarts.registerMap('guangzhou', guangzhouJson)
  chart = echarts.init(chartRef.value)
  updateChart()
  window.addEventListener('resize', () => chart?.resize())
}

function updateChart() {
  const option = {
    backgroundColor: '#0a0a1a',
    
    // ========== 3D 地图 ==========
    geo3D: {
      map: 'guangzhou',
      roam: true,
      center: [113.35, 23.15],
      zoom: 1.2,
      
      // 3D 材质和光照
      itemStyle: {
        color: '#1e3a5f',
        opacity: 1,
        borderColor: '#38bdf8',
        borderWidth: 1.5,
      },
      emphasis: {
        itemStyle: {
          color: '#2563eb',
        },
        label: {
          show: true,
          color: '#fff',
          fontSize: 14,
          fontWeight: 'bold',
          textShadowColor: '#000',
          textShadowBlur: 4,
        }
      },
      
      // 区域标签（每个区名字）
      label: {
        show: true,
        formatter: (params) => params.name,
        color: '#fff',
        fontSize: 11,
        fontWeight: 'normal',
        textShadowColor: '#000',
        textShadowBlur: 3,
        distance: 8,
      },
      
      // 3D 材质
      realisticMaterial: {
        roughness: 0.4,
        metalness: 0.2,
      },
      
      // 深度效果（让区域凸起）
      regions: guangzhouJson.features.map(f => ({
        name: f.properties.name,
        itemStyle: {
          color: getDistrictColor(f.properties.name),
          opacity: 0.95,
          borderColor: '#38bdf8',
          borderWidth: 1,
          shadowBlur: 10,
          shadowColor: 'rgba(56, 189, 248, 0.3)',
        },
        height: 0.5, // 基础高度
      })),
      
      // 视角控制
      viewControl: {
        projection: 'perspective',
        distance: 100,
        autoRotate: autoRotate.value,
        autoRotateSpeed: 1.2,
        alpha: 45,
        beta: 15,
        center: [0, 0, 0],
        minDistance: 30,
        maxDistance: 250,
        animation: true,
        animationDurationUpdate: 1000,
        panMouseButton: 'left',
        rotateMouseButton: 'right',
      },
      
      // 3D 光照
      light: {
        main: {
          intensity: 1.8,
          shadow: true,
          shadowQuality: 'ultra',
          alpha: 60,
          beta: 45,
          direction: [1, 1, 1],
        },
        ambient: {
          intensity: 0.6,
          color: '#38bdf8',
        },
        ambientCubemap: {
          enable: false,
        }
      },
      
      // 后处理效果
      postEffect: {
        enable: true,
        bloom: {
          enable: true,
          intensity: 0.08,
        },
        SSAO: {
          enable: true,
          radius: 2,
          intensity: 1,
        },
        depthOfField: {
          enable: false,
        },
      },
      
      // 超采样（抗锯齿）
      temporalSuperSampling: {
        enable: true,
      },
      
      // 地面
      groundPlane: {
        show: true,
        color: '#0a0a1a',
      },
    },

    // ========== 悬浮提示 ==========
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(15, 23, 42, 0.95)',
      borderColor: '#38bdf8',
      borderWidth: 1,
      textStyle: { color: '#fff' },
      formatter: (params) => {
        if (params.name) {
          return `<b style="font-size:14px">${params.name}</b><br/>AI市场区域`
        }
        return ''
      }
    },
  }

  chart.setOption(option, true)
}

function resetView() {
  updateChart()
}

// 不同区域用不同颜色
function getDistrictColor(name) {
  const colors = {
    '天河区': '#3b82f6',
    '黄埔区': '#10b981',
    '番禺区': '#8b5cf6',
    '白云区': '#f59e0b',
    '海珠区': '#ef4444',
    '越秀区': '#ec4899',
    '荔湾区': '#06b6d4',
    '花都区': '#84cc16',
    '南沙区': '#14b8a6',
    '增城区': '#a855f7',
    '从化区': '#22c55e',
  }
  return colors[name] || '#1e40af'
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
  gap: 12px;
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