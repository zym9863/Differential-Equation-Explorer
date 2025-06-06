# Differential Equation Explorer

> English | [中文](README.md)

An interactive differential equation learning and visualization tool built with Vue 3 + TypeScript + Vite.

## 📋 Project Overview

Differential Equation Explorer is a modern web application designed for mathematics learners and educators. It provides two main functional modules:

- **Interactive Solver**: Solves ordinary differential equations using the fourth-order Runge-Kutta numerical method
- **Slope Field Visualization**: Generates dynamic slope field graphics through D3.js, intuitively displaying solutions to differential equations

## ✨ Key Features

### 🧮 Interactive Solver
- Supports multiple ordinary differential equation formats
- Fourth-order Runge-Kutta numerical solving algorithm
- Preset common equation types (linear equations, separation of variables, homogeneous equations, Logistic growth, etc.)
- Adjustable initial conditions and solving parameters
- Detailed solving step explanations
- Numerical solution table display

### 📊 Slope Field Visualization
- Real-time slope field graph generation
- Interactive solution curve drawing (click anywhere to generate solution curves)
- Multiple solution curve management with color differentiation
- Adjustable coordinate range, grid density, arrow length and other parameters
- 6 preset typical differential equations
- Responsive design with mobile device support

### 🎨 Modern UI Design
- Frosted glass effects and gradient backgrounds
- Smooth animation transitions
- Responsive layout for different screen sizes
- Intuitive user interface and operation experience

## 🛠️ Technology Stack

- **Frontend Framework**: Vue 3 (Composition API)
- **Development Language**: TypeScript
- **Build Tool**: Vite
- **Mathematical Computation**: Math.js
- **Data Visualization**: D3.js
- **Styling**: CSS3 (Custom styling system)

## 📦 Project Structure

```
src/
├── App.vue                           # Main application component with tab switching
├── main.ts                           # Application entry file
├── style.css                         # Global styles and CSS variable definitions
├── vite-env.d.ts                     # Vite type declarations
├── assets/                           # Static resources
├── components/
│   ├── EquationSolver.vue           # Differential equation solver component
│   └── SlopeFieldVisualization.vue  # Slope field visualization component
```

## 🚀 Quick Start

### Requirements
- Node.js >= 16.0.0
- npm >= 7.0.0

### Install Dependencies
```bash
npm install
```

### Development Server
```bash
npm run dev
```
Visit http://localhost:5173

### Build for Production
```bash
npm run build
```

### Preview Production Build
```bash
npm run preview
```

## 🎯 Usage Guide

### Interactive Solver
1. Select a preset equation or input a custom differential equation (format: `dy/dx = expression`)
2. Set initial conditions x₀ and y₀
3. Adjust solving parameters (end point and step size)
4. Click the "Solve Equation" button
5. View solving steps and numerical solution table

### Slope Field Visualization
1. Select a preset equation or input a custom equation
2. Adjust coordinate range and display parameters
3. Click anywhere on the right canvas to generate solution curves
4. Use the control panel to manage solution curves (delete individual or clear all)

## 📈 Supported Equation Types

### Preset Equation Examples
- **Linear Equation**: `dy/dx = x + y`
- **Separation of Variables**: `dy/dx = x*y`
- **Homogeneous Equation**: `dy/dx = y/x`
- **Logistic Growth**: `dy/dx = y*(1-y)`
- **Simple Oscillation**: `dy/dx = -x`
- **Circular Trajectory**: `dy/dx = -x/y`

### Custom Equation Format
- Supports basic mathematical operators: `+`, `-`, `*`, `/`, `^`
- Supports mathematical functions: `sin`, `cos`, `tan`, `exp`, `log`, `sqrt`, etc.
- Variable names: `x` (independent variable), `y` (dependent variable)

## 🎨 Interface Preview

- **Modern Design**: Features frosted glass effects and gradient backgrounds
- **Two-Column Layout**: Control panel on the left, visualization area on the right
- **Responsive Adaptation**: Supports both desktop and mobile devices
- **Smooth Animations**: Hover effects and transition animations

## 🔧 Core Algorithms

### Runge-Kutta 4th Order Method
The project uses the classic fourth-order Runge-Kutta method for numerical solving:

```typescript
const rungeKutta4 = (f, x0, y0, h, xEnd) => {
  // k1 = h * f(x, y)
  // k2 = h * f(x + h/2, y + k1/2)
  // k3 = h * f(x + h/2, y + k2/2)
  // k4 = h * f(x + h, y + k3)
  // y_new = y + (k1 + 2*k2 + 2*k3 + k4) / 6
}
```

### Slope Field Generation
Uses D3.js to dynamically generate slope fields, calculating the slope at each point based on grid density:

```typescript
const slope = f(x, y)
const angle = Math.atan(slope)
// Draw direction vectors
```

## 🤝 Contributing

Issues and Pull Requests are welcome!

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Submit a Pull Request

## 📝 Development Notes

### Project Features
- Uses Vue 3 Composition API for clear code organization
- TypeScript provides type safety
- Modular CSS design with rich animation effects
- Math.js handles mathematical expression parsing and computation
- D3.js implements high-performance data visualization

### Performance Optimization
- Uses `nextTick()` to ensure drawing after DOM updates
- Debounced parameter changes to reduce unnecessary redraws
- Proper numerical computation boundary checks to avoid infinite loops

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## 🙏 Acknowledgments

- Vue.js team for the excellent framework
- D3.js community for data visualization solutions
- Math.js team for the mathematical computation library
- All developers who contribute to the open source community

---

**🎓 Educational Value**: This tool is particularly suitable for mathematics education, helping students intuitively understand the behavior and geometric meaning of differential equation solutions.

**🔬 Research Use**: Can be used for quick verification of numerical solutions and visual analysis of differential equations.

**💡 Learning Resource**: Clear code structure, suitable for learning Vue 3, TypeScript, and mathematical visualization techniques.
