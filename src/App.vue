<script setup lang="ts">
import { ref, onMounted } from 'vue'
import EquationSolver from './components/EquationSolver.vue'
import SlopeFieldVisualization from './components/SlopeFieldVisualization.vue'

const activeTab = ref('solver')

const switchTab = (tab: string) => {
  activeTab.value = tab
}
</script>

<template>
  <div class="app">
    <header class="header">
      <h1>微分方程探索者</h1>
      <p class="subtitle">Interactive Differential Equation Explorer</p>
    </header>

    <nav class="nav-tabs">
      <button 
        :class="['tab-btn', { active: activeTab === 'solver' }]"
        @click="switchTab('solver')"
      >
        交互式求解器
      </button>
      <button 
        :class="['tab-btn', { active: activeTab === 'visualization' }]"
        @click="switchTab('visualization')"
      >
        斜率场可视化
      </button>
    </nav>

    <main class="main-content">
      <EquationSolver v-if="activeTab === 'solver'" />
      <SlopeFieldVisualization v-if="activeTab === 'visualization'" />
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  position: relative;
  overflow-x: hidden;
}

.app::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 20% 80%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 40% 40%, rgba(120, 219, 255, 0.2) 0%, transparent 50%);
  pointer-events: none;
}

.header {
  text-align: center;
  padding: 3rem 1rem 2rem;
  color: white;
  position: relative;
  z-index: 1;
}

.header h1 {
  margin: 0;
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  background: linear-gradient(135deg, #ffffff 0%, #f0f8ff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 4px 20px rgba(0,0,0,0.3);
  letter-spacing: -0.02em;
  animation: titleGlow 3s ease-in-out infinite alternate;
}

@keyframes titleGlow {
  0% { text-shadow: 0 4px 20px rgba(0,0,0,0.3); }
  100% { text-shadow: 0 4px 30px rgba(255,255,255,0.3); }
}

.subtitle {
  margin: 1rem 0 0 0;
  font-size: 1.2rem;
  opacity: 0.95;
  font-weight: 300;
  letter-spacing: 0.5px;
  text-shadow: 0 2px 10px rgba(0,0,0,0.2);
}

.nav-tabs {
  display: flex;
  justify-content: center;
  gap: 1rem;
  padding: 0 1rem;
  margin-bottom: 3rem;
  position: relative;
  z-index: 1;
}

.tab-btn {
  padding: 1rem 2rem;
  border: none;
  border-radius: 50px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
  min-width: 180px;
}

.tab-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left 0.6s;
}

.tab-btn:hover::before {
  left: 100%;
}

.tab-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  border-color: rgba(255, 255, 255, 0.4);
}

.tab-btn.active {
  background: rgba(255, 255, 255, 0.95);
  color: #667eea;
  box-shadow: 0 15px 35px rgba(0,0,0,0.2);
  transform: translateY(-2px);
  border-color: rgba(255, 255, 255, 0.8);
}

.tab-btn.active::before {
  display: none;
}

.main-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 1rem 3rem;
  position: relative;
  z-index: 1;
}

/* 响应式改进 */
@media (max-width: 768px) {
  .header {
    padding: 2rem 1rem 1.5rem;
  }
  
  .nav-tabs {
    flex-direction: column;
    align-items: center;
    gap: 0.8rem;
  }
  
  .tab-btn {
    min-width: auto;
    width: 100%;
    max-width: 280px;
  }
  
  .main-content {
    padding: 0 0.5rem 2rem;
  }
}

@media (max-width: 480px) {
  .subtitle {
    font-size: 1rem;
  }
}
</style>
