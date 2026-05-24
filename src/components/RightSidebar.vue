<template>
  <aside class="sidebar overflow-y-auto px-4 py-4">
    <h2 class="section-label mb-4">TODAY'S</h2>

    <!-- Donut chart -->
    <div class="flex flex-col items-center mb-4">
      <div class="relative">
        <svg width="120" height="120" viewBox="0 0 120 120">
          <circle cx="60" cy="60" r="46" fill="none" stroke="var(--color-surface-selected)" stroke-width="16"/>
          <circle
            cx="60" cy="60" r="46"
            fill="none"
            stroke="#0D0D0D"
            stroke-width="16"
            :stroke-dasharray="`${progressArc} ${circumference - progressArc}`"
            stroke-dashoffset="57.81"
            stroke-linecap="round"
            transform="rotate(-90 60 60)"
          />
        </svg>
        <div class="absolute inset-0 flex flex-col items-center justify-center">
          <span class="text-xl font-black leading-none">{{ progressPercent }}%</span>
          <span class="text-[9px] text-gray-500 mt-0.5">done</span>
        </div>
      </div>
      <p class="text-xs text-gray-500 mt-1">{{ completedCount }} / {{ tasks.length }} tasks</p>
    </div>

    <div class="sidebar-divider mx-0 mb-4"></div>

    <!-- Upcoming deadlines -->
    <div class="flex-1 overflow-y-auto">
      <p class="section-label mb-3">Upcoming Deadlines</p>

      <p v-if="upcomingTasks.length === 0" class="text-xs text-gray-400">All caught up!</p>

      <div class="flex flex-col gap-2">
        <div
          v-for="task in upcomingTasks"
          :key="task.id"
          class="flex items-start gap-2 py-2 border-b border-black/10 last:border-0"
        >
          <span :class="['w-2 h-2 rounded-full shrink-0 mt-1.5', priorityDot(task.priority)]"></span>
          <div class="flex-1 min-w-0">
            <p :class="['text-xs truncate', task.done ? 'line-through text-gray-400' : '']">{{ task.title }}</p>
            <p class="text-[10px] text-gray-500 mt-0.5">{{ formatDate(task.deadline) }}</p>
          </div>
          <span class="text-[9px] font-bold shrink-0 mt-0.5" :class="urgencyColor(task.deadline)">
            {{ daysUntil(task.deadline) }}
          </span>
        </div>
      </div>
    </div>
  </aside>
</template>

<script>
export default {
  name: 'RightSidebar',
  props: {
    tasks:          { type: Array,  default: () => [] },
    completedCount: { type: Number, default: 0 },
  },
  data() {
    return { circumference: 2 * Math.PI * 46 }
  },
  computed: {
    progressPercent() {
      return this.tasks.length ? Math.round(this.completedCount / this.tasks.length * 100) : 0
    },
    progressArc() {
      return (this.progressPercent / 100) * this.circumference
    },
    upcomingTasks() {
      return this.tasks
        .filter(t => !t.done && t.deadline)
        .sort((a, b) => new Date(a.deadline) - new Date(b.deadline))
        .slice(0, 8)
    },
  },
  methods: {
    formatDate(iso) {
      if (!iso) return ''
      return new Date(iso + 'T00:00:00').toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
    },
    daysUntil(iso) {
      if (!iso) return ''
      const today  = new Date(); today.setHours(0, 0, 0, 0)
      const diff   = Math.round((new Date(iso + 'T00:00:00') - today) / 86400000)
      if (diff < 0)  return 'LATE'
      if (diff === 0) return 'TODAY'
      if (diff === 1) return 'TMR'
      return `${diff}d`
    },
    urgencyColor(iso) {
      if (!iso) return ''
      const today = new Date(); today.setHours(0, 0, 0, 0)
      const diff  = Math.round((new Date(iso + 'T00:00:00') - today) / 86400000)
      if (diff < 0)   return 'text-red-600'
      if (diff === 0) return 'text-red-500'
      if (diff <= 2)  return 'text-orange-500'
      return 'text-gray-400'
    },
    priorityDot(p) {
      return { P1: 'bg-black', P2: 'bg-gray-600', P3: 'bg-gray-400', P4: 'bg-gray-300' }[p] || 'bg-gray-300'
    },
  },
}
</script>
