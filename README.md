# 微分方程探索者 (Differential Equation Explorer)

> [English](README_EN.md) | 中文


一个交互式的微分方程学习和可视化工具，基于 Vue 3 + TypeScript + Vite 构建。

## 📋 项目简介

微分方程探索者是一个现代化的Web应用程序，专为数学学习者和教育工作者设计。它提供了两个主要功能模块：

- **交互式求解器**：使用四阶Runge-Kutta数值方法求解常微分方程
- **斜率场可视化**：通过D3.js生成动态斜率场图形，直观展示微分方程的解

## ✨ 主要特性

### 🧮 交互式求解器
- 支持多种常微分方程格式
- 四阶Runge-Kutta数值求解算法
- 预设常见方程类型（线性方程、分离变量、齐次方程、Logistic增长等）
- 可调节初始条件和求解参数
- 详细的求解步骤说明
- 数值解表格显示

### 📊 斜率场可视化
- 实时生成斜率场图形
- 交互式解曲线绘制（点击任意位置生成解曲线）
- 多条解曲线管理和颜色区分
- 可调节坐标范围、网格密度、箭头长度等参数
- 预设6种典型微分方程
- 响应式设计，支持移动端访问

### 🎨 现代化UI设计
- 毛玻璃效果和渐变背景
- 流畅的动画过渡效果
- 响应式布局，适配不同屏幕尺寸
- 直观的用户界面和操作体验

## 🛠️ 技术栈

- **前端框架**: Vue 3 (Composition API)
- **开发语言**: TypeScript
- **构建工具**: Vite
- **数学计算**: Math.js
- **数据可视化**: D3.js
- **样式**: CSS3 (自定义样式系统)

## 📦 项目结构

```
src/
├── App.vue                           # 主应用组件，包含标签页切换
├── main.ts                           # 应用入口文件
├── style.css                         # 全局样式和CSS变量定义
├── vite-env.d.ts                     # Vite类型声明
├── assets/                           # 静态资源
├── components/
│   ├── EquationSolver.vue           # 微分方程求解器组件
│   └── SlopeFieldVisualization.vue  # 斜率场可视化组件
```

## 🚀 快速开始

### 环境要求
- Node.js >= 16.0.0
- npm >= 7.0.0

### 安装依赖
```bash
npm install
```

### 开发服务器
```bash
npm run dev
```
访问 http://localhost:5173

### 构建生产版本
```bash
npm run build
```

### 预览生产构建
```bash
npm run preview
```

## 🎯 使用方法

### 交互式求解器
1. 选择预设方程或输入自定义微分方程（格式：`dy/dx = 表达式`）
2. 设置初始条件 x₀ 和 y₀
3. 调整求解参数（结束点和步长）
4. 点击"求解方程"按钮
5. 查看求解步骤和数值解表格

### 斜率场可视化
1. 选择预设方程或输入自定义方程
2. 调整坐标范围和显示参数
3. 在右侧画布上点击任意位置生成解曲线
4. 使用控制面板管理解曲线（删除单条或清除全部）

## 📈 支持的方程类型

### 预设方程示例
- **线性方程**: `dy/dx = x + y`
- **分离变量**: `dy/dx = x*y`
- **齐次方程**: `dy/dx = y/x`
- **Logistic增长**: `dy/dx = y*(1-y)`
- **简单振荡**: `dy/dx = -x`
- **圆形轨迹**: `dy/dx = -x/y`

### 自定义方程格式
- 支持基本数学运算符: `+`, `-`, `*`, `/`, `^`
- 支持数学函数: `sin`, `cos`, `tan`, `exp`, `log`, `sqrt` 等
- 变量名: `x` (自变量), `y` (因变量)

## 🎨 界面预览

- **现代化设计**: 采用毛玻璃效果和渐变背景
- **双栏布局**: 左侧控制面板，右侧可视化区域
- **响应式适配**: 支持桌面端和移动端
- **流畅动画**: 悬停效果和过渡动画

## 🔧 核心算法

### Runge-Kutta 4阶方法
项目使用经典的四阶Runge-Kutta方法进行数值求解：

```typescript
const rungeKutta4 = (f, x0, y0, h, xEnd) => {
  // k1 = h * f(x, y)
  // k2 = h * f(x + h/2, y + k1/2)
  // k3 = h * f(x + h/2, y + k2/2)
  // k4 = h * f(x + h, y + k3)
  // y_new = y + (k1 + 2*k2 + 2*k3 + k4) / 6
}
```

### 斜率场生成
使用D3.js动态生成斜率场，根据网格密度计算每个点的斜率：

```typescript
const slope = f(x, y)
const angle = Math.atan(slope)
// 绘制方向向量
```

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支: `git checkout -b feature/amazing-feature`
3. 提交更改: `git commit -m 'Add amazing feature'`
4. 推送分支: `git push origin feature/amazing-feature`
5. 提交 Pull Request

## 📝 开发笔记

### 项目特色
- 使用 Vue 3 Composition API，代码组织清晰
- TypeScript 提供类型安全
- 模块化CSS设计，包含丰富的动画效果
- Math.js 处理数学表达式解析和计算
- D3.js 实现高性能的数据可视化

### 性能优化
- 使用 `nextTick()` 确保DOM更新后再执行绘制
- 防抖处理参数变化，减少不必要的重绘
- 合理的数值计算边界检查，避免无限循环

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

## 🙏 致谢

- Vue.js 团队提供的优秀框架
- D3.js 社区的数据可视化解决方案
- Math.js 团队的数学计算库
- 所有为开源社区做出贡献的开发者们

---

**🎓 教育价值**: 这个工具特别适合数学教育，帮助学生直观理解微分方程的解的行为和几何意义。

**🔬 研究用途**: 可用于快速验证微分方程的数值解和可视化分析。

**💡 学习资源**: 代码结构清晰，适合学习Vue 3、TypeScript和数学可视化技术。
