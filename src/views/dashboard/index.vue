<template>
  <div class="dashboard-page">
    <!-- 欢迎横幅 -->
    <div class="welcome-banner">
      <div class="welcome-left">
        <div class="welcome-title">{{ greeting }}，{{ userName }}</div>
        <div class="welcome-sub">
          今日共有 {{ data.todayAppointments ?? 0 }} 场体检预约、{{ data.todayActivityRegistrations ?? 0 }}
          人次活动报名，社区整体运营平稳。
        </div>
      </div>
      <div class="welcome-right">
        <el-icon :size="22"><Sunrise /></el-icon>
        <span>AI 智能养老社区 · 让守护更有温度</span>
      </div>
    </div>

    <!-- 数据统计卡片 -->
    <el-row :gutter="16" class="stat-row">
      <el-col v-for="card in statCards" :key="card.title" class="stat-col">
        <StatisticCard v-bind="card" />
      </el-col>
    </el-row>

    <!-- 健康数据分析 -->
    <el-row :gutter="16" class="chart-row">
      <el-col :xs="24" :md="8">
        <el-card shadow="never">
          <template #header>
            <ChartHeader title="老人健康状态分布" desc="基于最近一次健康巡检" />
          </template>
          <HealthChart :option="pieOption" height="264px" />
        </el-card>
      </el-col>
      <el-col :xs="24" :md="8">
        <el-card shadow="never">
          <template #header>
            <ChartHeader title="血压异常趋势" desc="近 7 天异常记录（人次）" />
          </template>
          <HealthChart :option="lineOption" height="264px" />
        </el-card>
      </el-col>
      <el-col :xs="24" :md="8">
        <el-card shadow="never">
          <template #header>
            <ChartHeader title="体检完成率" desc="本月预约套餐完成情况" />
          </template>
          <HealthChart :option="gaugeOption" height="264px" />
        </el-card>
      </el-col>
    </el-row>

    <!-- 待办事项 + 最近动态 -->
    <el-row :gutter="16" class="bottom-row">
      <el-col :xs="24" :md="10">
        <el-card shadow="never" class="fill-card">
          <template #header>
            <ChartHeader title="待办事项" desc="需要您尽快处理" />
          </template>
          <TodoList :items="todoItems" />
        </el-card>
      </el-col>
      <el-col :xs="24" :md="14">
        <el-card shadow="never" class="fill-card">
          <template #header>
            <ChartHeader title="最近动态" desc="社区实时动态" />
          </template>
          <RecentActivity :items="recentActivities" />
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, h } from 'vue'
import { Sunrise } from '@element-plus/icons-vue'
import { getDashboard } from '../../api/dashboard'
import { useAuthStore } from '../../store/auth'
import StatisticCard from '../../components/dashboard/StatisticCard.vue'
import HealthChart from '../../components/dashboard/HealthChart.vue'
import TodoList from '../../components/dashboard/TodoList.vue'
import RecentActivity from '../../components/dashboard/RecentActivity.vue'
import {
  healthDistribution,
  bpTrend,
  checkupCompletion,
  healthAlertCount,
  unreadMessageCount,
  recentActivities
} from './mock'

const auth = useAuthStore()
const data = ref({})

onMounted(async () => {
  data.value = await getDashboard()
})

// ===== 欢迎区 =====
const userName = computed(() => auth.userInfo?.realName || auth.userInfo?.phone || '管理员')
const greeting = computed(() => {
  const h = new Date().getHours()
  if (h < 6) return '凌晨好'
  if (h < 9) return '早上好'
  if (h < 12) return '上午好'
  if (h < 14) return '中午好'
  if (h < 18) return '下午好'
  return '晚上好'
})

// ===== 统计卡片（真实接口数据）=====
const statCards = computed(() => [
  {
    title: '会员总数',
    value: data.value.totalMembers ?? 0,
    icon: 'User',
    theme: 'primary',
    trend: 3.2,
    description: '较上月'
  },
  {
    title: '今日新增会员',
    value: data.value.todayNewMembers ?? 0,
    icon: 'UserFilled',
    theme: 'green',
    trend: 12.5,
    description: '较昨日'
  },
  {
    title: '今日预约',
    value: data.value.todayAppointments ?? 0,
    icon: 'Calendar',
    theme: 'cyan',
    trend: 8.1,
    description: '体检预约单量'
  },
  {
    title: '活动报名',
    value: data.value.todayActivityRegistrations ?? 0,
    icon: 'Flag',
    theme: 'warning',
    trend: -2.4,
    description: '今日报名人次'
  },
  {
    title: '待处理事项',
    value: data.value.pendingAppointments ?? 0,
    icon: 'Bell',
    theme: 'purple',
    trend: null,
    description: '待确认预约'
  }
])

// ===== 待办事项（预约数为真实数据，其余待接后端）=====
const todoItems = computed(() => [
  {
    title: '待审核预约',
    description: '会员提交的体检预约等待确认',
    icon: 'Calendar',
    color: '#2e7cf6',
    colorBg: '#eaf2fe',
    count: data.value.pendingAppointments ?? 0,
    route: '/appointment'
  },
  {
    title: '健康异常提醒',
    description: 'AI 巡检发现指标异常，需人工复核',
    icon: 'Warning',
    color: '#f56c6c',
    colorBg: '#fdeaea',
    count: healthAlertCount,
    route: '/health'
  },
  {
    title: '未处理消息',
    description: '会员留言与系统通知待回复',
    icon: 'ChatDotRound',
    color: '#f5a623',
    colorBg: '#fdf3e3',
    count: unreadMessageCount,
    route: '/message'
  }
])

// ===== 图表卡片标题（标题 + 描述 + 操作位）=====
const ChartHeader = {
  props: { title: String, desc: String },
  setup(props) {
    return () =>
      h('div', { class: 'chart-header' }, [
        h('div', { class: 'chart-header-left' }, [
          h('span', { class: 'chart-title' }, props.title),
          h('span', { class: 'chart-desc' }, props.desc)
        ])
      ])
  }
}

// ===== 图表配置 =====
const PIE_COLORS = ['#34b382', '#2e7cf6', '#f5a623', '#8b7cf6', '#f56c6c']

const pieOption = computed(() => ({
  color: PIE_COLORS,
  tooltip: { trigger: 'item', formatter: '{b}：{c} 人（{d}%）' },
  legend: { bottom: 0, icon: 'circle', itemWidth: 8, itemHeight: 8, textStyle: { color: '#8a93a3', fontSize: 12 } },
  series: [
    {
      type: 'pie',
      radius: ['46%', '68%'],
      center: ['50%', '44%'],
      itemStyle: { borderRadius: 6, borderColor: '#fff', borderWidth: 2 },
      label: { show: false },
      emphasis: { label: { show: true, fontWeight: 600, formatter: '{b}\n{c} 人' } },
      data: healthDistribution
    }
  ]
}))

const lineOption = computed(() => ({
  color: ['#2e7cf6', '#34b382'],
  tooltip: { trigger: 'axis' },
  legend: { bottom: 0, icon: 'circle', itemWidth: 8, itemHeight: 8, textStyle: { color: '#8a93a3', fontSize: 12 } },
  grid: { left: 8, right: 12, top: 24, bottom: 36, containLabel: true },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: bpTrend.days,
    axisLine: { lineStyle: { color: '#e8ecf3' } },
    axisLabel: { color: '#8a93a3', fontSize: 11 }
  },
  yAxis: {
    type: 'value',
    splitLine: { lineStyle: { color: '#f0f3f8', type: 'dashed' } },
    axisLabel: { color: '#8a93a3', fontSize: 11 }
  },
  series: [
    {
      name: '收缩压异常',
      type: 'line',
      smooth: true,
      symbolSize: 6,
      data: bpTrend.systolic,
      areaStyle: { opacity: 0.08 }
    },
    {
      name: '舒张压异常',
      type: 'line',
      smooth: true,
      symbolSize: 6,
      data: bpTrend.diastolic,
      areaStyle: { opacity: 0.08 }
    }
  ]
}))

const gaugeOption = computed(() => ({
  series: [
    {
      type: 'gauge',
      center: ['50%', '58%'],
      radius: '92%',
      startAngle: 210,
      endAngle: -30,
      min: 0,
      max: 100,
      progress: {
        show: true,
        width: 16,
        roundCap: true,
        itemStyle: {
          color: {
            type: 'linear',
            x: 0, y: 0, x2: 1, y2: 0,
            colorStops: [
              { offset: 0, color: '#34b382' },
              { offset: 1, color: '#2e7cf6' }
            ]
          }
        }
      },
      axisLine: { lineStyle: { width: 16, color: [[1, '#f0f3f8']] } },
      axisTick: { show: false },
      splitLine: { show: false },
      axisLabel: { show: false },
      pointer: { show: false },
      anchor: { show: false },
      title: { show: true, offsetCenter: [0, '32%'], color: '#8a93a3', fontSize: 13 },
      detail: {
        valueAnimation: true,
        offsetCenter: [0, '-4%'],
        formatter: '{value}%',
        color: '#303845',
        fontSize: 30,
        fontWeight: 700
      },
      data: [{ value: checkupCompletion, name: '本月体检完成率' }]
    }
  ]
}))
</script>

<style scoped>
.dashboard-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 欢迎横幅：医疗蓝 → 养老绿渐变 */
.welcome-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 22px 26px;
  border-radius: var(--ec-radius-lg);
  color: #fff;
  background: linear-gradient(120deg, #2e7cf6 0%, #38a0e0 55%, #34b382 100%);
  box-shadow: 0 6px 20px rgba(46, 124, 246, 0.25);
}
.welcome-title {
  font-size: 19px;
  font-weight: 600;
}
.welcome-sub {
  margin-top: 6px;
  font-size: 13px;
  opacity: 0.92;
}
.welcome-right {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  opacity: 0.92;
  white-space: nowrap;
}

/* 统计卡片行：5 等分 */
.stat-row .stat-col {
  flex: 0 0 20%;
  max-width: 20%;
}
@media (max-width: 1200px) {
  .stat-row .stat-col {
    flex: 0 0 33.33%;
    max-width: 33.33%;
    margin-bottom: 16px;
  }
}
@media (max-width: 768px) {
  .stat-row .stat-col {
    flex: 0 0 50%;
    max-width: 50%;
  }
}

.chart-row :deep(.el-card__header),
.bottom-row :deep(.el-card__header) {
  padding: 16px 20px 12px;
  border-bottom: none;
}
.chart-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.chart-header-left {
  display: flex;
  align-items: baseline;
  gap: 10px;
}
.chart-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--ec-text-primary);
}
.chart-desc {
  font-size: 12px;
  color: var(--ec-text-secondary);
}
.fill-card {
  height: 100%;
}
</style>
