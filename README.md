# ECharts 3D 地图标记点解决方案

## 问题
ECharts GL 的 3D 地图不支持直接在 `series` 里打点，必须切到 2D 模式。

## 解决方案
使用 **两层地图叠加**：
- **底层**：3D 地图（视觉展示）
- **顶层**：2D 地图 + scatter 系列（标记点）

通过控制顶层地图透明度，让标记点"悬浮"在 3D 地图上方。

## 技术栈
- Vue 3
- ECharts 5 + echarts-gl
- Vite

## 运行
```bash
npm install
npm run dev
```

## 效果
访问 http://localhost:5173 查看带标记点的 3D 中国地图