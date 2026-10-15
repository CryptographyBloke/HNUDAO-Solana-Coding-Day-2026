# 地球Online (Myearth) · 3D 地球交互与地理坐标点亮平台

> 参赛队伍：配合操作
> 项目仓库：https://github.com/fufu2007-coder/Myearth

## 项目简介

**地球Online (Myearth)** 是一个基于 Next.js 16 + React 19 + Three.js 构建的沉浸式 3D 虚拟地球交互与地理坐标点亮平台。

系统提供动态渲染的三维地球模型，支持平滑的鼠标拖拽旋转交互与响应式视口适配。用户在控制台输入现实世界的地点名称（如“杭州西湖”），系统通过后台地理编码接口（Geocoding API）将地址逆向解析为经纬度，并通过球面几何投影算法将目标点动态投射至 3D 地球表面，点亮发光地标并平滑转动聚焦。

## 技术栈与特性

- **前端技术栈**：Next.js (App Router)、React 19、Three.js、Tailwind CSS。
- **球面坐标转换算法**：利用球坐标系投影公式（`latLonToVector3`）将地理经纬度精准映射至三维空间点：
  - $x = -r \cdot \sin(\phi) \cdot \cos(	heta)$
  - $z = r \cdot \sin(\phi) \cdot \sin(	heta)$
  - $y = r \cdot \cos(\phi)$
- **交互与动效**：3D 网格渲染、轨道阻尼拖拽手势、动态标记动画、高斯模糊玻璃拟态标签显示。

## 快速启动

```bash
npm install
npm run dev
```
打开 `http://localhost:3000` 即可开始探索。
