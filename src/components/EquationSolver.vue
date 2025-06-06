<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { evaluate, derivative, parse } from 'mathjs'

interface SolutionStep {
  step: string
  description: string
  result?: string
}

const equation = ref("dy/dx = x + y")
const initialX = ref(0)
const initialY = ref(1)
const xEnd = ref(2)
const stepSize = ref(0.1)
const solution = ref<number[][]>([])
const solutionSteps = ref<SolutionStep[]>([])
const isLoading = ref(false)

// 预设方程例子
const presetEquations = [
  { name: "线性方程", eq: "dy/dx = x + y", x0: 0, y0: 1 },
  { name: "分离变量", eq: "dy/dx = x*y", x0: 0, y0: 1 },
  { name: "齐次方程", eq: "dy/dx = y/x", x0: 1, y0: 1 },
  { name: "Logistic增长", eq: "dy/dx = y*(1-y)", x0: 0, y0: 0.1 }
]

// 使用Runge-Kutta方法数值求解
const rungeKutta4 = (f: (x: number, y: number) => number, x0: number, y0: number, h: number, xEnd: number) => {
  const points: number[][] = [[x0, y0]]
  let x = x0
  let y = y0
  
  while (x < xEnd) {
    const k1 = h * f(x, y)
    const k2 = h * f(x + h/2, y + k1/2)
    const k3 = h * f(x + h/2, y + k2/2)
    const k4 = h * f(x + h, y + k3)
    
    y = y + (k1 + 2*k2 + 2*k3 + k4) / 6
    x = x + h
    points.push([x, y])
  }
  
  return points
}

// 解析微分方程字符串
const parseEquation = (eqStr: string): (x: number, y: number) => number => {
  try {
    // 处理常见的微分方程格式
    let processedEq = eqStr.replace(/dy\/dx\s*=\s*/, '')
    processedEq = processedEq.replace(/\^/g, '**') // 转换指数符号
    
    return (x: number, y: number) => {
      const scope = { x, y }
      return evaluate(processedEq, scope)
    }
  } catch (error) {
    throw new Error('方程解析失败，请检查格式')
  }
}

const solveEquation = () => {
  if (!equation.value.trim()) return
  
  isLoading.value = true
  solutionSteps.value = []
  
  try {
    // 添加求解步骤说明
    solutionSteps.value = [
      {
        step: "1",
        description: "解析微分方程",
        result: `原方程: ${equation.value}`
      },
      {
        step: "2", 
        description: "设置初始条件",
        result: `初始条件: y(${initialX.value}) = ${initialY.value}`
      },
      {
        step: "3",
        description: "使用四阶Runge-Kutta数值方法求解",
        result: `步长: h = ${stepSize.value}, 求解区间: [${initialX.value}, ${xEnd.value}]`
      }
    ]
    
    const f = parseEquation(equation.value)
    const points = rungeKutta4(f, initialX.value, initialY.value, stepSize.value, xEnd.value)
    solution.value = points
    
    solutionSteps.value.push({
      step: "4",
      description: "数值解计算完成",
      result: `共计算了 ${points.length} 个数据点`
    })
    
  } catch (error) {
    console.error('求解失败:', error)
    solutionSteps.value = [{
      step: "错误",
      description: "求解失败",
      result: error instanceof Error ? error.message : '未知错误'
    }]
  } finally {
    isLoading.value = false
  }
}

const loadPreset = (preset: typeof presetEquations[0]) => {
  equation.value = preset.eq
  initialX.value = preset.x0
  initialY.value = preset.y0
}

const formatNumber = (num: number): string => {
  return Number(num.toFixed(6)).toString()
}

onMounted(() => {
  solveEquation()
})
</script>

<template>
  <div class="equation-solver">
    <div class="solver-container">
      <!-- 输入面板 -->
      <div class="input-panel">
        <h2>微分方程求解器</h2>
        
        <!-- 预设方程 -->
        <div class="preset-section">
          <h3>预设方程</h3>
          <div class="preset-buttons">
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
        
        <!-- 初始条件 -->
        <div class="initial-conditions">
          <h3>初始条件</h3>
          <div class="input-row">
            <div class="input-group">
              <label>x₀:</label>
              <input v-model.number="initialX" type="number" step="0.1" />
            </div>
            <div class="input-group">
              <label>y₀:</label>
              <input v-model.number="initialY" type="number" step="0.1" />
            </div>
          </div>
        </div>
        
        <!-- 求解参数 -->
        <div class="solve-params">
          <h3>求解参数</h3>
          <div class="input-row">
            <div class="input-group">
              <label>结束点 x:</label>
              <input v-model.number="xEnd" type="number" step="0.1" />
            </div>
            <div class="input-group">
              <label>步长:</label>
              <input v-model.number="stepSize" type="number" step="0.01" min="0.01" max="0.5" />
            </div>
          </div>
        </div>
        
        <button @click="solveEquation" :disabled="isLoading" class="solve-btn">
          {{ isLoading ? '求解中...' : '求解方程' }}
        </button>
      </div>
      
      <!-- 结果显示面板 -->
      <div class="result-panel">
        <!-- 求解步骤 -->
        <div class="solution-steps">
          <h3>求解步骤</h3>
          <div class="steps-container">
            <div 
              v-for="step in solutionSteps" 
              :key="step.step"
              class="step-item"
            >
              <div class="step-number">{{ step.step }}</div>
              <div class="step-content">
                <div class="step-description">{{ step.description }}</div>
                <div v-if="step.result" class="step-result">{{ step.result }}</div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 数值解表格 -->
        <div v-if="solution.length > 0" class="solution-table">
          <h3>数值解 (前20个点)</h3>
          <div class="table-container">
            <table>
              <thead>
                <tr>
                  <th>x</th>
                  <th>y</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(point, index) in solution.slice(0, 20)" :key="index">
                  <td>{{ formatNumber(point[0]) }}</td>
                  <td>{{ formatNumber(point[1]) }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.equation-solver {
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

.equation-solver::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
}

.solver-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2.5rem;
}

.input-panel, .result-panel {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 
    0 20px 40px rgba(0,0,0,0.08),
    0 1px 2px rgba(0,0,0,0.05),
    inset 0 1px 0 rgba(255,255,255,0.9);
  border: 1px solid rgba(255,255,255,0.2);
  position: relative;
  transition: all 0.3s ease;
}

.input-panel:hover, .result-panel:hover {
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
  font-size: 1.2rem;
  color: #34495e;
  position: relative;
  padding-left: 1rem;
}

h3::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 20px;
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

.preset-buttons {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.8rem;
}

.preset-btn {
  padding: 0.8rem 1.2rem;
  border: 2px solid transparent;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  color: #3498db;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 0.9rem;
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
  font-size: 0.95rem;
  letter-spacing: 0.5px;
}

.equation-input {
  width: 100%;
  padding: 1rem 1.2rem;
  border: 2px solid #e1e8ed;
  border-radius: 12px;
  font-size: 1rem;
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

.initial-conditions, .solve-params {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  border-radius: 16px;
  border: 1px solid #e9ecef;
}

.input-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}

.input-row input {
  width: 100%;
  padding: 0.8rem 1rem;
  border: 2px solid #e1e8ed;
  border-radius: 10px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background: rgba(255, 255, 255, 0.8);
  font-size: 0.95rem;
  text-align: center;
  font-weight: 500;
}

.input-row input:focus {
  outline: none;
  border-color: #3498db;
  background: rgba(255, 255, 255, 1);
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
  transform: translateY(-1px);
}

.solve-btn {
  width: 100%;
  padding: 1.2rem 2rem;
  background: linear-gradient(135deg, #3498db 0%, #2ecc71 100%);
  color: white;
  border: none;
  border-radius: 16px;
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  margin-top: 2rem;
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 1px;
  box-shadow: 0 8px 25px rgba(52, 152, 219, 0.2);
}

.solve-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left 0.6s;
}

.solve-btn:hover:not(:disabled)::before {
  left: 100%;
}

.solve-btn:hover:not(:disabled) {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 15px 35px rgba(52, 152, 219, 0.3);
  background: linear-gradient(135deg, #2ecc71 0%, #3498db 100%);
}

.solve-btn:active:not(:disabled) {
  transform: translateY(-1px) scale(1.01);
}

.solve-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
  background: linear-gradient(135deg, #bdc3c7 0%, #95a5a6 100%);
}

.solution-steps {
  margin-bottom: 2rem;
}

.steps-container {
  max-height: 350px;
  overflow-y: auto;
  padding-right: 0.5rem;
}

.steps-container::-webkit-scrollbar {
  width: 6px;
}

.steps-container::-webkit-scrollbar-track {
  background: #f1f2f6;
  border-radius: 3px;
}

.steps-container::-webkit-scrollbar-thumb {
  background: #ddd;
  border-radius: 3px;
}

.step-item {
  display: flex;
  margin-bottom: 1.2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  border-radius: 12px;
  border: 1px solid #e9ecef;
  transition: all 0.3s ease;
  animation: slideInFromLeft 0.5s ease-out;
}

@keyframes slideInFromLeft {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.step-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0,0,0,0.08);
}

.step-number {
  background: linear-gradient(135deg, #3498db 0%, #2ecc71 100%);
  color: white;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  margin-right: 1.2rem;
  flex-shrink: 0;
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
  font-size: 0.9rem;
}

.step-content {
  flex-grow: 1;
}

.step-description {
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 0.8rem;
  font-size: 1rem;
}

.step-result {
  color: #5a6c7d;
  font-size: 0.9rem;
  font-family: 'JetBrains Mono', 'Fira Code', Consolas, monospace;
  background: rgba(52, 152, 219, 0.05);
  padding: 0.8rem 1rem;
  border-radius: 8px;
  border-left: 3px solid #3498db;
  line-height: 1.5;
}

.solution-table {
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  border-radius: 16px;
  padding: 1.5rem;
  border: 1px solid #e9ecef;
}

.table-container {
  max-height: 450px;
  overflow-y: auto;
  border: 1px solid #e1e8ed;
  border-radius: 12px;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.06);
}

.table-container::-webkit-scrollbar {
  width: 8px;
}

.table-container::-webkit-scrollbar-track {
  background: #f1f2f6;
}

.table-container::-webkit-scrollbar-thumb {
  background: #ddd;
  border-radius: 4px;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-family: 'JetBrains Mono', 'Fira Code', Consolas, monospace;
}

th, td {
  padding: 1rem 1.2rem;
  text-align: center;
  border-bottom: 1px solid #e9ecef;
  font-size: 0.9rem;
}

th {
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
  font-weight: 700;
  color: #2c3e50;
  position: sticky;
  top: 0;
  border-bottom: 2px solid #3498db;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-size: 0.85rem;
}

td {
  font-weight: 500;
  color: #34495e;
}

tr:hover {
  background: rgba(52, 152, 219, 0.05);
}

tr:nth-child(even) {
  background: rgba(248, 249, 250, 0.5);
}

@media (max-width: 768px) {
  .solver-container {
    grid-template-columns: 1fr;
  }
  
  .preset-buttons {
    grid-template-columns: 1fr;
  }
  
  .input-row {
    grid-template-columns: 1fr;
  }
}
</style>
