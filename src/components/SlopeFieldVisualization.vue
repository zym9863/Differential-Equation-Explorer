<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from 'vue'
import { evaluate } from 'mathjs'
import * as d3 from 'd3'

const equation = ref("dy/dx = x + y")
const svgRef = ref<SVGElement>()
const canvasWidth = 800
const canvasHeight = 600
const xRange = ref([-4, 4])
const yRange = ref([-3, 3])
const gridDensity = ref(20)
const solutionCurves = ref<Array<{id: number, points: number[][], color: string}>>([])
const showGrid = ref(true)
const arrowLength = ref(15)

// 预设方程
const presetEquations = [
  { name: "线性方程", eq: "dy/dx = x + y" },
  { name: "分离变量", eq: "dy/dx = x*y" },
  { name: "简单振荡", eq: "dy/dx = -x" },
  { name: "Logistic增长", eq: "dy/dx = y*(1-y)" },
  { name: "圆形轨迹", eq: "dy/dx = -x/y" },
  { name: "双曲线", eq: "dy/dx = y/x" }
]

let curveIdCounter = 0

// 解析微分方程
const parseEquation = (eqStr: string): (x: number, y: number) => number => {
  try {
    let processedEq = eqStr.replace(/dy\/dx\s*=\s*/, '')
    processedEq = processedEq.replace(/\^/g, '**')
    
    return (x: number, y: number) => {
      if (Math.abs(y) < 1e-10 && processedEq.includes('/y')) {
        return 0
      }
      if (Math.abs(x) < 1e-10 && processedEq.includes('/x')) {
        return 0
      }
      const scope = { x, y }
      try {
        const result = evaluate(processedEq, scope)
        return isFinite(result) ? result : 0
      } catch {
        return 0
      }
    }
  } catch (error) {
    return () => 0
  }
}

// 使用Runge-Kutta方法生成解曲线
const generateSolutionCurve = (f: (x: number, y: number) => number, x0: number, y0: number, direction: number = 1): number[][] => {
  const points: number[][] = [[x0, y0]]
  const h = 0.05 * direction
  const maxSteps = 200
  let x = x0
  let y = y0
  
  for (let i = 0; i < maxSteps; i++) {
    if (x < xRange.value[0] || x > xRange.value[1] || y < yRange.value[0] || y > yRange.value[1]) {
      break
    }
    
    const k1 = h * f(x, y)
    const k2 = h * f(x + h/2, y + k1/2)
    const k3 = h * f(x + h/2, y + k2/2)
    const k4 = h * f(x + h, y + k3)
    
    if (!isFinite(k1) || !isFinite(k2) || !isFinite(k3) || !isFinite(k4)) {
      break
    }
    
    y = y + (k1 + 2*k2 + 2*k3 + k4) / 6
    x = x + h
    
    if (!isFinite(x) || !isFinite(y)) {
      break
    }
    
    points.push([x, y])
  }
  
  return points
}

// 绘制斜率场
const drawSlopeField = () => {
  if (!svgRef.value) return
  
  const svg = d3.select(svgRef.value)
  svg.selectAll("*").remove()
  
  // 设置缩放
  const xScale = d3.scaleLinear()
    .domain(xRange.value)
    .range([0, canvasWidth])
  
  const yScale = d3.scaleLinear()
    .domain(yRange.value)
    .range([canvasHeight, 0])
  
  // 绘制背景
  svg.append("rect")
    .attr("width", canvasWidth)
    .attr("height", canvasHeight)
    .attr("fill", "#fafafa")
    .attr("stroke", "#e0e0e0")
  
  // 绘制网格
  if (showGrid.value) {
    const xAxis = d3.axisBottom(xScale).tickSize(-canvasHeight)
    const yAxis = d3.axisLeft(yScale).tickSize(-canvasWidth)
    
    svg.append("g")
      .attr("class", "grid")
      .attr("transform", `translate(0,${canvasHeight})`)
      .call(xAxis)
      .selectAll("text").remove()
    
    svg.append("g")
      .attr("class", "grid")
      .call(yAxis)
      .selectAll("text").remove()
    
    svg.selectAll(".grid line")
      .attr("stroke", "#e8e8e8")
      .attr("stroke-width", 1)
    
    svg.selectAll(".grid path")
      .attr("stroke", "none")
  }
  
  // 绘制坐标轴
  const xAxisLine = d3.axisBottom(xScale)
  const yAxisLine = d3.axisLeft(yScale)
  
  svg.append("g")
    .attr("class", "x-axis")
    .attr("transform", `translate(0,${yScale(0)})`)
    .call(xAxisLine)
    .selectAll("line, path")
    .attr("stroke", "#333")
    .attr("stroke-width", 2)
  
  svg.append("g")
    .attr("class", "y-axis")
    .attr("transform", `translate(${xScale(0)},0)`)
    .call(yAxisLine)
    .selectAll("line, path")
    .attr("stroke", "#333")
    .attr("stroke-width", 2)
  
  // 解析方程
  const f = parseEquation(equation.value)
  
  // 绘制斜率场
  const stepX = (xRange.value[1] - xRange.value[0]) / gridDensity.value
  const stepY = (yRange.value[1] - yRange.value[0]) / gridDensity.value
  
  for (let x = xRange.value[0]; x <= xRange.value[1]; x += stepX) {
    for (let y = yRange.value[0]; y <= yRange.value[1]; y += stepY) {
      const slope = f(x, y)
      if (!isFinite(slope)) continue
      
      const angle = Math.atan(slope)
      const length = arrowLength.value
      
      const x1 = xScale(x) - (length / 2) * Math.cos(angle)
      const y1 = yScale(y) + (length / 2) * Math.sin(angle)
      const x2 = xScale(x) + (length / 2) * Math.cos(angle)
      const y2 = yScale(y) - (length / 2) * Math.sin(angle)
      
      svg.append("line")
        .attr("x1", x1)
        .attr("y1", y1)
        .attr("x2", x2)
        .attr("y2", y2)
        .attr("stroke", "#3498db")
        .attr("stroke-width", 1.5)
        .attr("opacity", 0.7)
    }
  }
  
  // 绘制现有的解曲线
  drawSolutionCurves()
  
  // 添加点击事件
  svg.on("click", (event) => {
    const [mouseX, mouseY] = d3.pointer(event)
    const x = xScale.invert(mouseX)
    const y = yScale.invert(mouseY)
    
    addSolutionCurve(x, y)
  })
}

// 绘制解曲线
const drawSolutionCurves = () => {
  if (!svgRef.value) return
  
  const svg = d3.select(svgRef.value)
  const xScale = d3.scaleLinear().domain(xRange.value).range([0, canvasWidth])
  const yScale = d3.scaleLinear().domain(yRange.value).range([canvasHeight, 0])
  
  // 移除现有的解曲线
  svg.selectAll(".solution-curve").remove()
  
  // 绘制每条解曲线
  solutionCurves.value.forEach(curve => {
    const line = d3.line<number[]>()
      .x(d => xScale(d[0]))
      .y(d => yScale(d[1]))
      .curve(d3.curveCardinal)
    
    svg.append("path")
      .datum(curve.points)
      .attr("class", "solution-curve")
      .attr("d", line)
      .attr("stroke", curve.color)
      .attr("stroke-width", 3)
      .attr("fill", "none")
      .attr("opacity", 0.8)
    
    // 添加起始点
    if (curve.points.length > 0) {
      const startPoint = curve.points[0]
      svg.append("circle")
        .attr("class", "solution-curve")
        .attr("cx", xScale(startPoint[0]))
        .attr("cy", yScale(startPoint[1]))
        .attr("r", 4)
        .attr("fill", curve.color)
        .attr("stroke", "white")
        .attr("stroke-width", 2)
    }
  })
}

// 添加解曲线
const addSolutionCurve = (x0: number, y0: number) => {
  const f = parseEquation(equation.value)
  
  // 生成双向解曲线
  const forwardPoints = generateSolutionCurve(f, x0, y0, 1)
  const backwardPoints = generateSolutionCurve(f, x0, y0, -1)
  
  // 合并点，去除重复的起始点
  const allPoints = [...backwardPoints.reverse().slice(0, -1), ...forwardPoints]
  
  const colors = ['#e74c3c', '#2ecc71', '#f39c12', '#9b59b6', '#1abc9c', '#f1c40f']
  const color = colors[solutionCurves.value.length % colors.length]
  
  solutionCurves.value.push({
    id: curveIdCounter++,
    points: allPoints,
    color: color
  })
  
  drawSolutionCurves()
}

// 清除所有解曲线
const clearSolutionCurves = () => {
  solutionCurves.value = []
  drawSlopeField()
}

// 加载预设方程
const loadPreset = (preset: typeof presetEquations[0]) => {
  equation.value = preset.eq
  clearSolutionCurves()
  nextTick(() => {
    drawSlopeField()
  })
}

// 删除特定解曲线
const removeCurve = (curveId: number) => {
  solutionCurves.value = solutionCurves.value.filter(curve => curve.id !== curveId)
  drawSlopeField()
}

// 监听参数变化
watch([equation, xRange, yRange, gridDensity, showGrid, arrowLength], () => {
  nextTick(() => {
    drawSlopeField()
  })
}, { deep: true })

onMounted(() => {
  nextTick(() => {
    drawSlopeField()
  })
})
</script>

<template>
  <div class="slope-field-visualization">
    <div class="viz-container">
      <!-- 控制面板 -->
      <div class="control-panel">
        <h2>斜率场可视化</h2>
        
        <!-- 预设方程 -->
        <div class="preset-section">
          <h3>预设方程</h3>
          <div class="preset-grid">
            <button 
              v-for="preset in presetEquations" 
              :key="preset.name"
              @click="loadPreset(preset)"
              class="preset-btn"
            >
              {{ preset.name }}
            </button>
          </div>
        </div>
        
        <!-- 方程输入 -->
        <div class="input-group">
          <label>微分方程：</label>
          <input 
            v-model="equation" 
            type="text" 
            placeholder="例如: dy/dx = x + y"
            class="equation-input"
          />
        </div>
        
        <!-- 坐标范围 -->
        <div class="range-controls">
          <h3>坐标范围</h3>
          <div class="range-row">
            <div class="range-group">
              <label>X范围:</label>
              <div class="range-inputs">
                <input v-model.number="xRange[0]" type="number" step="0.5" />
                <span>到</span>
                <input v-model.number="xRange[1]" type="number" step="0.5" />
              </div>
            </div>
            <div class="range-group">
              <label>Y范围:</label>
              <div class="range-inputs">
                <input v-model.number="yRange[0]" type="number" step="0.5" />
                <span>到</span>
                <input v-model.number="yRange[1]" type="number" step="0.5" />
              </div>
            </div>
          </div>
        </div>
        
        <!-- 显示设置 -->
        <div class="display-controls">
          <h3>显示设置</h3>
          <div class="control-row">
            <div class="input-group">
              <label>网格密度:</label>
              <input v-model.number="gridDensity" type="range" min="10" max="40" />
              <span class="value">{{ gridDensity }}</span>
            </div>
            <div class="input-group">
              <label>箭头长度:</label>
              <input v-model.number="arrowLength" type="range" min="8" max="25" />
              <span class="value">{{ arrowLength }}</span>
            </div>
          </div>
          <div class="checkbox-group">
            <label>
              <input v-model="showGrid" type="checkbox" />
              显示网格
            </label>
          </div>
        </div>
        
        <!-- 解曲线管理 -->
        <div class="curves-management">
          <h3>解曲线 ({{ solutionCurves.length }}条)</h3>
          <button @click="clearSolutionCurves" class="clear-btn">
            清除所有曲线
          </button>
          <div v-if="solutionCurves.length > 0" class="curves-list">
            <div 
              v-for="curve in solutionCurves" 
              :key="curve.id"
              class="curve-item"
            >
              <div class="curve-color" :style="{ backgroundColor: curve.color }"></div>
              <span>曲线 {{ curve.id + 1 }} ({{ curve.points.length }} 点)</span>
              <button @click="removeCurve(curve.id)" class="remove-btn">×</button>
            </div>
          </div>
        </div>
        
        <div class="instructions">
          <h3>使用说明</h3>
          <ul>
            <li>在右侧图形上点击任意位置生成解曲线</li>
            <li>蓝色小线段表示斜率方向</li>
            <li>彩色曲线是通过点击生成的解</li>
            <li>可以调整参数实时更新显示</li>
          </ul>
        </div>
      </div>
      
      <!-- 可视化画布 -->
      <div class="canvas-panel">
        <div class="canvas-container">
          <svg 
            ref="svgRef"
            :width="canvasWidth" 
            :height="canvasHeight"
            class="slope-field-canvas"
          ></svg>
        </div>
        <div class="canvas-info">
          <p>点击画布上任意位置生成通过该点的解曲线</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.slope-field-visualization {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 24px;
  padding: 2.5rem;
  box-shadow: 
    0 32px 64px rgba(0,0,0,0.12),
    inset 0 1px 0 rgba(255,255,255,0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255,255,255,0.1);
  position: relative;
  overflow: hidden;
}

.slope-field-visualization::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
}

.viz-container {
  display: grid;
  grid-template-columns: 380px 1fr;
  gap: 2.5rem;
}

.control-panel {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 
    0 20px 40px rgba(0,0,0,0.08),
    0 1px 2px rgba(0,0,0,0.05),
    inset 0 1px 0 rgba(255,255,255,0.9);
  border: 1px solid rgba(255,255,255,0.2);
  height: fit-content;
  transition: all 0.3s ease;
}

.control-panel:hover {
  transform: translateY(-2px);
  box-shadow: 
    0 28px 50px rgba(0,0,0,0.12),
    0 2px 4px rgba(0,0,0,0.08),
    inset 0 1px 0 rgba(255,255,255,0.9);
}

.canvas-panel {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 
    0 20px 40px rgba(0,0,0,0.08),
    0 1px 2px rgba(0,0,0,0.05),
    inset 0 1px 0 rgba(255,255,255,0.9);
  border: 1px solid rgba(255,255,255,0.2);
  transition: all 0.3s ease;
}

.canvas-panel:hover {
  transform: translateY(-2px);
  box-shadow: 
    0 28px 50px rgba(0,0,0,0.12),
    0 2px 4px rgba(0,0,0,0.08),
    inset 0 1px 0 rgba(255,255,255,0.9);
}

h2, h3 {
  color: #2c3e50;
  margin-bottom: 1.5rem;
  font-weight: 700;
  position: relative;
}

h2 {
  font-size: 1.6rem;
  background: linear-gradient(135deg, #2c3e50 0%, #3498db 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 2rem;
}

h3 {
  font-size: 1.1rem;
  color: #34495e;
  position: relative;
  padding-left: 1rem;
  margin-bottom: 1.2rem;
}

h3::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 18px;
  background: linear-gradient(135deg, #3498db, #2ecc71);
  border-radius: 2px;
}

.preset-section {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.preset-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.8rem;
}

.preset-btn {
  padding: 0.8rem 1rem;
  border: 2px solid transparent;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  color: #3498db;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 0.85rem;
  font-weight: 600;
  position: relative;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.1);
}

.preset-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(52, 152, 219, 0.1), transparent);
  transition: left 0.6s;
}

.preset-btn:hover::before {
  left: 100%;
}

.preset-btn:hover {
  background: linear-gradient(135deg, #3498db 0%, #2ecc71 100%);
  color: white;
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(52, 152, 219, 0.3);
  border-color: rgba(255,255,255,0.3);
}

.input-group {
  margin-bottom: 1.5rem;
}

.input-group label {
  display: block;
  margin-bottom: 0.8rem;
  font-weight: 600;
  color: #2c3e50;
  font-size: 0.9rem;
  letter-spacing: 0.5px;
}

.equation-input {
  width: 100%;
  padding: 0.8rem 1rem;
  border: 2px solid #e1e8ed;
  border-radius: 10px;
  font-size: 0.9rem;
  font-family: 'JetBrains Mono', 'Fira Code', Consolas, monospace;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background: rgba(255, 255, 255, 0.8);
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.06);
}

.equation-input:focus {
  outline: none;
  border-color: #3498db;
  background: rgba(255, 255, 255, 1);
  box-shadow: 
    inset 0 2px 4px rgba(0,0,0,0.06),
    0 0 0 3px rgba(52, 152, 219, 0.1);
  transform: translateY(-1px);
}

.range-controls {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.range-row {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
}

.range-group {
  flex: 1;
}

.range-group label {
  font-size: 0.85rem;
  margin-bottom: 0.6rem;
  color: #34495e;
}

.range-inputs {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.range-inputs input {
  width: 70px;
  padding: 0.6rem 0.8rem;
  border: 2px solid #e1e8ed;
  border-radius: 8px;
  font-size: 0.85rem;
  text-align: center;
  font-weight: 500;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.8);
}

.range-inputs input:focus {
  outline: none;
  border-color: #3498db;
  background: rgba(255, 255, 255, 1);
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.range-inputs span {
  font-size: 0.8rem;
  color: #7f8c8d;
  font-weight: 500;
}

.display-controls {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.control-row {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
}

.input-group input[type="range"] {
  width: 100%;
  margin: 0.8rem 0;
  height: 6px;
  background: #e1e8ed;
  border-radius: 3px;
  outline: none;
  -webkit-appearance: none;
}

.input-group input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  background: linear-gradient(135deg, #3498db, #2ecc71);
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(52, 152, 219, 0.3);
  transition: all 0.3s ease;
}

.input-group input[type="range"]::-webkit-slider-thumb:hover {
  transform: scale(1.2);
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.4);
}

.value {
  font-weight: bold;
  color: #3498db;
  margin-left: 0.8rem;
  font-size: 0.9rem;
  padding: 0.3rem 0.6rem;
  background: rgba(52, 152, 219, 0.1);
  border-radius: 6px;
  min-width: 30px;
  text-align: center;
}

.checkbox-group {
  margin-top: 1.2rem;
  padding: 1rem;
  background: rgba(52, 152, 219, 0.05);
  border-radius: 10px;
  border: 1px solid rgba(52, 152, 219, 0.1);
}

.checkbox-group label {
  display: flex;
  align-items: center;
  font-size: 0.9rem;
  cursor: pointer;
  font-weight: 600;
  color: #2c3e50;
}

.checkbox-group input {
  margin-right: 0.8rem;
  width: 16px;
  height: 16px;
  accent-color: #3498db;
}

.curves-management {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.clear-btn {
  width: 100%;
  padding: 0.8rem 1rem;
  background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  margin-bottom: 1.2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-size: 0.85rem;
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.2);
}

.clear-btn:hover {
  background: linear-gradient(135deg, #c0392b 0%, #a93226 100%);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(231, 76, 60, 0.3);
}

.curves-list {
  max-height: 180px;
  overflow-y: auto;
  padding-right: 0.5rem;
}

.curves-list::-webkit-scrollbar {
  width: 4px;
}

.curves-list::-webkit-scrollbar-track {
  background: #f1f2f6;
  border-radius: 2px;
}

.curves-list::-webkit-scrollbar-thumb {
  background: #ddd;
  border-radius: 2px;
}

.curve-item {
  display: flex;
  align-items: center;
  padding: 0.8rem 1rem;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  border-radius: 10px;
  margin-bottom: 0.8rem;
  border: 1px solid #e9ecef;
  transition: all 0.3s ease;
  animation: slideInFromRight 0.3s ease-out;
}

@keyframes slideInFromRight {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.curve-item:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

.curve-color {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  margin-right: 0.8rem;
  border: 2px solid white;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.curve-item span {
  flex-grow: 1;
  font-size: 0.85rem;
  font-weight: 500;
  color: #2c3e50;
}

.remove-btn {
  background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
  color: white;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: bold;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 6px rgba(231, 76, 60, 0.2);
}

.remove-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3);
}

.instructions {
  background: linear-gradient(135deg, #e8f4fd 0%, #f8fcff 100%);
  padding: 1.5rem;
  border-radius: 12px;
  border: 1px solid rgba(52, 152, 219, 0.2);
  border-left: 4px solid #3498db;
}

.instructions h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
  font-size: 1rem;
}

.instructions ul {
  margin: 0;
  padding-left: 1.2rem;
}

.instructions li {
  font-size: 0.85rem;
  margin-bottom: 0.6rem;
  color: #34495e;
  line-height: 1.4;
  position: relative;
}

.instructions li::marker {
  color: #3498db;
}

.canvas-container {
  display: flex;
  justify-content: center;
  margin-bottom: 1.5rem;
  padding: 1rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.slope-field-canvas {
  border: 2px solid #e1e8ed;
  border-radius: 12px;
  cursor: crosshair;
  box-shadow: 0 8px 25px rgba(0,0,0,0.08);
  transition: all 0.3s ease;
}

.slope-field-canvas:hover {
  border-color: #3498db;
  box-shadow: 0 12px 35px rgba(0,0,0,0.12);
}

.canvas-info {
  text-align: center;
  color: #5a6c7d;
  font-size: 0.9rem;
  font-weight: 500;
  padding: 1rem;
  background: rgba(52, 152, 219, 0.05);
  border-radius: 10px;
  border: 1px solid rgba(52, 152, 219, 0.1);
}

@media (max-width: 1200px) {
  .viz-container {
    grid-template-columns: 1fr;
  }
  
  .canvas-container {
    overflow-x: auto;
  }
  
  .slope-field-canvas {
    min-width: 800px;
  }
}
</style>
