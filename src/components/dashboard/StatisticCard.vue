<template>
  <div class="stat-card" :style="{ '--theme-color': themeColor, '--theme-bg': themeBg }">
    <div class="stat-icon">
      <el-icon :size="24"><component :is="icon" /></el-icon>
    </div>
    <div class="stat-body">
      <div class="stat-title">{{ title }}</div>
      <div class="stat-value num-anim">{{ displayValue }}</div>
      <div class="stat-foot">
        <span v-if="trend !== null" class="trend" :class="trend >= 0 ? 'up' : 'down'">
          <el-icon :size="12">
            <component :is="trend >= 0 ? 'CaretTop' : 'CaretBottom'" />
          </el-icon>
          {{ Math.abs(trend) }}%
        </span>
        <span class="stat-desc">{{ description }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  title: { type: String, required: true },
  value: { type: Number, default: 0 },
  icon: { type: String, default: 'DataAnalysis' },
  /** 主题色：primary 医疗蓝 / green 养老绿 / warning 暖橙 / purple 紫 / cyan 青 */
  theme: { type: String, default: 'primary' },
  /** 环比趋势百分比，null 表示不展示 */
  trend: { type: Number, default: null },
  description: { type: String, default: '' }
})

const THEME_COLORS = {
  primary: ['#2e7cf6', '#eaf2fe'],
  green: ['#34b382', '#e8f7f1'],
  warning: ['#f5a623', '#fdf3e3'],
  purple: ['#8b7cf6', '#f0eeff'],
  cyan: ['#22b8cf', '#e6f7fa']
}
const [themeColor, themeBg] = THEME_COLORS[props.theme] || THEME_COLORS.primary

// ===== 数字滚动动画（easeOutCubic，800ms）=====
const displayValue = ref(0)
let rafId = null

function animate(target) {
  cancelAnimationFrame(rafId)
  const duration = 800
  const start = performance.now()
  const from = displayValue.value
  const step = (now) => {
    const p = Math.min((now - start) / duration, 1)
    const eased = 1 - Math.pow(1 - p, 3)
    displayValue.value = Math.round(from + (target - from) * eased)
    if (p < 1) rafId = requestAnimationFrame(step)
  }
  rafId = requestAnimationFrame(step)
}

watch(
  () => props.value,
  (v) => animate(v ?? 0)
)

onMounted(() => animate(props.value ?? 0))
onBeforeUnmount(() => cancelAnimationFrame(rafId))
</script>

<style scoped>
.stat-card {
  display: flex;
  align-items: center;
  gap: 16px;
  height: 100%;
  padding: 20px;
  background: #fff;
  border: 1px solid var(--ec-border);
  border-radius: var(--ec-radius-lg);
  box-shadow: var(--ec-shadow-card);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}
.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--ec-shadow-hover);
}
.stat-icon {
  flex: none;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 52px;
  height: 52px;
  border-radius: var(--ec-radius-md);
  color: var(--theme-color);
  background: var(--theme-bg);
}
.stat-body {
  min-width: 0;
}
.stat-title {
  font-size: 13px;
  color: var(--ec-text-secondary);
}
.stat-value {
  margin: 4px 0 6px;
  font-size: 28px;
  font-weight: 700;
  line-height: 1;
  color: var(--ec-text-primary);
}
.stat-foot {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
}
.trend {
  display: inline-flex;
  align-items: center;
  gap: 1px;
  font-weight: 600;
}
.trend.up {
  color: var(--ec-green);
}
.trend.down {
  color: var(--ec-danger);
}
.stat-desc {
  color: var(--ec-text-secondary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
