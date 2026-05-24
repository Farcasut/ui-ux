<template>
  <aside class="sidebar overflow-hidden flex flex-col" @click.self="$emit('clear-selection')">

    <!-- ── TASK DETAIL (any view + task selected) ── -->
    <template v-if="selectedTask">
      <div class="px-5 pt-5 pb-2 shrink-0 flex items-center justify-between">
        <p class="section-label">Detalii task selectat</p>
        <button
          class="w-6 h-6 flex items-center justify-center rounded-full hover:bg-black/10 cursor-pointer border-0 bg-transparent text-gray-400 hover:text-black transition-colors text-lg leading-none"
          title="Inchide"
          @click="$emit('clear-selection')"
        >×</button>
      </div>

      <div class="flex-1 overflow-y-auto px-5 pb-4 flex flex-col gap-3">
        <div class="detail-field">
          <label class="detail-label">Titlu</label>
          <div class="detail-value">{{ selectedTask.title }}</div>
        </div>
        <div class="detail-field">
          <label class="detail-label">Lista</label>
          <div class="detail-value">{{ selectedTask.list }}</div>
        </div>
        <div class="detail-field">
          <label class="detail-label">Prioritate</label>
          <div class="detail-value">{{ selectedTask.priority }}{{ selectedTask.priority === 'P1' ? ' – urgent' : selectedTask.priority === 'P2' ? ' – normal' : ' – low' }}</div>
        </div>
        <div class="detail-field">
          <label class="detail-label">Deadline</label>
          <div class="detail-value">{{ formatDeadline(selectedTask.deadline, selectedTask.time) }}</div>
        </div>
        <div class="detail-field">
          <label class="detail-label">Reminder</label>
          <div class="detail-value">{{ selectedTask.reminder || '—' }}</div>
        </div>
        <div class="detail-field">
          <label class="detail-label">Status</label>
          <div class="detail-value">{{ selectedTask.done ? 'Finalizat' : 'In lucru' }}</div>
        </div>
        <div v-if="selectedTask.notes" class="detail-field">
          <label class="detail-label">Note</label>
          <div class="detail-value text-xs text-gray-500">{{ selectedTask.notes }}</div>
        </div>
      </div>

      <div class="px-5 pb-5 flex flex-col gap-2 shrink-0">
        <div class="flex gap-2">
          <button
            class="flex-1 py-2.5 rounded-xl border border-black text-sm font-semibold bg-transparent cursor-pointer hover:bg-black/5 transition-colors"
            @click="$emit('open-edit-modal', selectedTask)"
          >Edit</button>
          <button
            class="flex-1 py-2.5 rounded-xl text-sm font-semibold cursor-pointer border-0 transition-colors"
            :class="selectedTask.done ? 'bg-gray-200 text-black hover:bg-gray-300' : 'bg-black text-white hover:bg-gray-800'"
            @click="$emit('update-task', { id: selectedTask.id, done: !selectedTask.done })"
          >{{ selectedTask.done ? 'Undone' : 'Done' }}</button>
        </div>
        <div class="flex gap-2">
          <button
            class="flex-1 py-2.5 rounded-xl border border-black/20 text-sm font-semibold bg-transparent cursor-pointer hover:bg-black/5 transition-colors"
            @click="$emit('open-edit-modal', { ...selectedTask, id: undefined })"
          >Duplica</button>
          <button
            class="flex-1 py-2.5 rounded-xl bg-red-600 text-white text-sm font-semibold cursor-pointer border-0 hover:bg-red-700 transition-colors"
            @click="$emit('delete-task', selectedTask.id)"
          >Sterge</button>
        </div>
      </div>
    </template>

    <!-- ── TODAY PROGRESS (default view) ── -->
    <template v-else>
      <div class="px-5 pt-5 pb-2 shrink-0">
        <p class="section-label">Today Progress</p>
      </div>

      <!-- Donut chart -->
      <div class="flex flex-col items-center py-4 shrink-0">
        <div class="relative">
          <svg width="120" height="120" viewBox="0 0 120 120">
            <circle cx="60" cy="60" r="46" fill="none" stroke="var(--color-surface-selected)" stroke-width="14"/>
            <circle
              cx="60" cy="60" r="46"
              fill="none"
              stroke="#0D0D0D"
              stroke-width="14"
              :stroke-dasharray="`${progressArc} ${circumference - progressArc}`"
              stroke-linecap="round"
              transform="rotate(-90 60 60)"
            />
          </svg>
          <div class="absolute inset-0 flex flex-col items-center justify-center">
            <span class="text-2xl font-black leading-none">{{ progressPercent }}%</span>
          </div>
        </div>
        <p class="text-xs text-gray-500 mt-2">{{ viewCompletedCount }} / {{ viewTasks.length }} tasks</p>
      </div>

      <div class="sidebar-divider mx-0 shrink-0"></div>

      <!-- Deadlines -->
      <div class="flex-1 overflow-y-auto px-5 pt-4 pb-4">
        <p class="section-label mb-3">Deadlines</p>

        <p v-if="deadlineTasks.length === 0" class="text-xs text-gray-400">Totul e in regula!</p>

        <div class="flex flex-col gap-1.5">
          <div
            v-for="task in deadlineTasks"
            :key="task.id"
            class="flex items-center gap-2 py-1.5"
          >
            <div class="flex-1 min-w-0">
              <p class="text-xs font-medium truncate">{{ task.title }}</p>
              <p class="text-[10px] text-gray-400">{{ task.list }}</p>
            </div>
            <span
              class="text-[11px] font-bold shrink-0 px-2 py-1 rounded-lg"
              :class="timeBadgeClass(task.deadline)"
            >{{ task.time || formatDate(task.deadline) }}</span>
          </div>
        </div>
      </div>
    </template>

  </aside>
</template>

<script>
export default {
  name: 'RightSidebar',
  emits: ['update-task', 'open-edit-modal', 'clear-selection', 'delete-task'],
  props: {
    tasks:          { type: Array,  default: () => [] },
    viewTasks:      { type: Array,  default: () => [] },
    completedCount: { type: Number, default: 0 },
    activeList:     { type: String, default: 'Azi' },
    selectedTask:   { type: Object, default: null },
    isShared:       { type: Boolean, default: false },
  },
  data() {
    return { circumference: 2 * Math.PI * 46 }
  },
  computed: {
    progressPercent() {
      return this.viewTasks.length ? Math.round(this.viewCompletedCount / this.viewTasks.length * 100) : 0
    },
    viewCompletedCount() {
      return this.viewTasks.filter(t => t.done).length
    },
    progressArc() {
      return (this.progressPercent / 100) * this.circumference
    },
    deadlineTasks() {
      const today = new Date().toISOString().slice(0, 10)
      return this.tasks
        .filter(t => !t.done && t.deadline && t.deadline <= today)
        .sort((a, b) => (a.time || '99:99').localeCompare(b.time || '99:99'))
        .slice(0, 6)
    },
  },
  methods: {
    formatDate(iso) {
      if (!iso) return ''
      return new Date(iso + 'T00:00:00').toLocaleDateString('ro-RO', { month: 'short', day: 'numeric' })
    },
    formatDeadline(iso, time) {
      if (!iso) return '—'
      const d = new Date(iso + 'T00:00:00')
      const today = new Date(); today.setHours(0, 0, 0, 0)
      const diff = Math.round((d - today) / 86400000)
      const dateStr = diff === 0 ? 'Azi' : diff === 1 ? 'Maine' : d.toLocaleDateString('ro-RO', { day: 'numeric', month: 'short' })
      return time ? `${dateStr}, ${time}` : dateStr
    },
    timeBadgeClass(iso) {
      if (!iso) return 'bg-gray-200 text-gray-600'
      const today = new Date(); today.setHours(0, 0, 0, 0)
      const diff = Math.round((new Date(iso + 'T00:00:00') - today) / 86400000)
      if (diff < 0)   return 'bg-black text-white'
      if (diff === 0) return 'bg-black text-white'
      return 'bg-gray-200 text-gray-700'
    },
  },
}
</script>
