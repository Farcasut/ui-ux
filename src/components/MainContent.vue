<template>
  <main class="flex-1 flex flex-col overflow-hidden" style="background: var(--color-app-bg);">

    <!-- ════════════════════════════════════════ TODAY VIEW (WF1) -->
    <template v-if="activeList === 'Azi'">

      <!-- Search bar -->
      <div class="px-6 pt-5 shrink-0">
        <div
          class="flex items-center gap-3 px-4 rounded-full border border-black/30"
          style="height: 48px; background: var(--color-surface);"
        >
          <svg class="w-4 h-4 text-gray-500 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/>
          </svg>
          <input
            type="text"
            :value="searchQuery"
            placeholder="Cauta task..."
            class="flex-1 bg-transparent text-sm outline-none placeholder-gray-400"
            @input="$emit('update:search-query', $event.target.value)"
          />
        </div>
      </div>

      <!-- Heading + Stat cards -->
      <div class="px-6 pt-5 shrink-0">
        <h1 class="text-2xl font-black mb-4">Azi</h1>

        <div class="grid grid-cols-3 gap-3 mb-4">
          <div class="stat-card">
            <p class="section-label">Urgent</p>
            <p class="text-3xl font-black leading-none mt-1">{{ urgentCount }}</p>
            <p class="text-xs text-gray-500 mt-1">{{ urgentCount }} tasks sunt urgente</p>
            <p class="text-[10px] text-gray-400">Prioritate P1</p>
          </div>
          <div class="stat-card">
            <p class="section-label">Planificate</p>
            <p class="text-3xl font-black leading-none mt-1">{{ plannedCount }}</p>
            <p class="text-xs text-gray-500 mt-1">Deadline Azi</p>
          </div>
          <div class="stat-card">
            <p class="section-label">Finalizate</p>
            <p class="text-3xl font-black leading-none mt-1">{{ doneCount }}<span class="text-lg text-gray-400">/{{ tasks.length }}</span></p>
            <p class="text-xs text-gray-500 mt-1">Status Azi</p>
          </div>
        </div>

        <!-- Filter chips -->
        <div class="flex items-center gap-1.5 flex-wrap">
          <button
            v-for="chip in todayFilterChips"
            :key="chip"
            :class="['filter-chip', { 'is-active': activeFilter === chip }]"
            @click="$emit('update:active-filter', chip)"
          >{{ chip }}</button>
        </div>
      </div>

      <!-- Task list -->
      <div class="flex flex-col gap-2 overflow-y-auto flex-1 px-6 pt-3 pb-4">
        <div
          v-for="task in tasks"
          :key="task.id"
          :class="['task-row group', { 'is-selected': selectedTaskId === task.id, 'is-urgent': task.priority === 'P1' }]"
          @click="$emit('select-task', selectedTaskId === task.id ? null : task.id)"
        >
          <button
            :class="['task-checkbox', { 'is-done': task.done }]"
            @click.stop="$emit('toggle-task', task.id)"
          >
            <svg v-if="task.done" viewBox="0 0 10 8" class="w-2.5 h-2.5" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M1 4l3 3 5-6"/>
            </svg>
          </button>

          <div class="flex flex-col flex-1 min-w-0">
            <span :class="['text-sm font-medium truncate', task.done ? 'line-through text-gray-400' : '']">{{ task.title }}</span>
            <span class="text-xs text-gray-400 truncate">{{ task.list }}{{ task.time ? ' · ' + task.time : '' }}</span>
          </div>

          <div class="flex items-center gap-1.5 shrink-0">
            <span v-if="task.assignee && isTaskShared(task)" class="tag-btn text-[10px]" style="height: 24px; min-width: auto; padding: 0 8px;">{{ task.assignee }}</span>
            <span :class="['tag-btn text-[10px] font-bold', priorityClass(task.priority)]" style="height: 24px; min-width: 30px;">{{ task.priority }}</span>
            <button
              class="text-gray-400 hover:text-black cursor-pointer border-0 bg-transparent opacity-0 group-hover:opacity-100 transition-opacity shrink-0 p-0.5"
              title="Duplica task"
              @click.stop="$emit('open-edit-modal', { ...task, id: undefined })"
            >
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                <rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
              </svg>
            </button>
            <button
              class="text-gray-400 hover:text-red-500 cursor-pointer border-0 bg-transparent opacity-0 group-hover:opacity-100 transition-opacity text-xl leading-none shrink-0"
              title="Sterge"
              @click.stop="$emit('delete-task', task.id)"
            >×</button>
          </div>
        </div>

        <!-- Add task button -->
        <button
          class="task-row border border-dashed border-black/25 bg-transparent hover:bg-transparent hover:border-black/50 transition-colors cursor-pointer"
          style="justify-content: flex-start;"
          @click="$emit('open-add-modal', null)"
        >
          <span class="text-gray-400 text-lg leading-none">+</span>
          <span class="text-sm text-gray-400 italic">Adauga task...</span>
        </button>

        <p v-if="tasks.length === 0" class="text-center text-gray-400 text-sm mt-8">Niciun task pentru azi</p>
      </div>
    </template>

    <!-- ════════════════════════════════════════ LIST VIEW (WF2) -->
    <template v-else>

      <!-- Header -->
      <div class="px-6 pt-6 pb-4 shrink-0">
        <div class="flex items-start justify-between gap-4 mb-1">
          <h1 class="text-2xl font-black">Lista: {{ activeList }}</h1>
          <button class="tag-btn px-4 mt-1 shrink-0" style="height: 34px;" @click="$emit('open-share-modal', activeList)">Partajeaza</button>
        </div>
        <p class="text-xs text-gray-500">
          Owner: Mihai &nbsp;·&nbsp; {{ allListTasks.length }} task-uri &nbsp;·&nbsp; {{ doneListCount }} finalizate
        </p>

        <!-- Filters -->
        <div class="flex items-center gap-1.5 mt-3 flex-wrap">
          <button
            v-for="chip in listFilterChips"
            :key="chip"
            :class="['filter-chip', { 'is-active': activeFilter === chip }]"
            @click="$emit('update:active-filter', chip)"
          >{{ chip }}</button>
        </div>
      </div>

      <div class="sidebar-divider mx-6"></div>

      <!-- Add task -->
      <div class="px-6 pt-3 shrink-0">
        <button
          class="w-full flex items-center gap-2 px-4 py-2.5 rounded-xl text-sm text-gray-500 hover:text-black transition-colors bg-transparent cursor-pointer border border-dashed border-black/25 hover:border-black/50"
          @click="$emit('open-add-modal', activeList)"
        >
          <span class="text-base leading-none">+</span>
          Adauga task in lista {{ activeList }}...
        </button>
      </div>

      <!-- Grouped task list -->
      <div class="flex-1 overflow-y-auto px-6 py-4">
        <div v-for="group in taskGroups" :key="group.label" class="mb-6">
          <h2 class="text-xs font-bold uppercase tracking-wider text-gray-500 mb-2 px-1">{{ group.label }}</h2>
          <div class="flex flex-col gap-2">
            <div
              v-for="task in group.tasks"
              :key="task.id"
              :class="['task-row group', { 'is-selected': selectedTaskId === task.id, 'is-urgent': task.priority === 'P1' }]"
              @click="$emit('select-task', selectedTaskId === task.id ? null : task.id)"
            >
              <button
                :class="['task-checkbox', { 'is-done': task.done }]"
                @click.stop="$emit('toggle-task', task.id)"
              >
                <svg v-if="task.done" viewBox="0 0 10 8" class="w-2.5 h-2.5" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M1 4l3 3 5-6"/>
                </svg>
              </button>

              <div class="flex flex-col flex-1 min-w-0">
                <span :class="['text-sm font-medium truncate', task.done ? 'line-through text-gray-400' : '']">{{ task.title }}</span>
                <span class="text-xs text-gray-400 truncate">
                  {{ task.priority }} · {{ formatDate(task.deadline) }}{{ task.time ? ' ' + task.time : '' }}{{ isShared && task.assignee ? ' · ' + task.assignee : '' }}
                </span>
              </div>

              <span :class="['tag-btn text-[10px] font-bold shrink-0', priorityClass(task.priority)]" style="height: 26px; min-width: 32px;">{{ task.priority }}</span>

              <button
                class="text-gray-400 hover:text-black cursor-pointer border-0 bg-transparent opacity-0 group-hover:opacity-100 transition-opacity shrink-0 p-0.5"
                title="Duplica task"
                @click.stop="$emit('open-edit-modal', { ...task, id: undefined })"
              >
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                  <rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
                </svg>
              </button>

              <button
                class="text-gray-400 hover:text-red-500 cursor-pointer border-0 bg-transparent opacity-0 group-hover:opacity-100 transition-opacity text-xl leading-none shrink-0"
                title="Sterge"
                @click.stop="$emit('delete-task', task.id)"
              >×</button>
            </div>
          </div>
        </div>

        <p v-if="tasks.length === 0" class="text-center text-gray-400 text-sm mt-8">Niciun task in aceasta lista</p>
      </div>
    </template>

  </main>
</template>

<script>
export default {
  name: 'MainContent',
  emits: [
    'update:search-query', 'update:active-filter',
    'select-task', 'open-add-modal', 'open-edit-modal', 'open-share-modal',
    'toggle-task', 'update-task', 'delete-task',
  ],
  props: {
    tasks:          { type: Array,  default: () => [] },
    allTasks:       { type: Array,  default: () => [] },
    lists:          { type: Array,  default: () => [] },
    activeList:     { type: String, default: 'Azi' },
    selectedTaskId: { type: String, default: null },
    searchQuery:    { type: String, default: '' },
    activeFilter:   { type: String, default: 'ALL' },
    isShared:         { type: Boolean, default: false },
    sharedListNames:  { type: Array,   default: () => [] },
  },
  computed: {
    urgentCount() {
      return this.tasks.filter(t => !t.done && t.priority === 'P1').length
    },
    plannedCount() {
      return this.tasks.filter(t => !t.done).length
    },
    doneCount() {
      return this.tasks.filter(t => t.done).length
    },
    allListTasks() {
      return this.allTasks.filter(t => t.list === this.activeList)
    },
    doneListCount() {
      return this.allListTasks.filter(t => t.done).length
    },
    todayFilterChips() {
      const uniqueLists = [...new Set(this.allTasks.map(t => t.list).filter(Boolean))].slice(0, 3)
      return ['ALL', 'P1', 'P2', 'P3', ...uniqueLists]
    },
    listFilterChips() {
      return ['ALL', 'P1', 'P2', 'P3']
    },
    taskGroups() {
      const todayStr = new Date().toISOString().slice(0, 10)
      const weekEnd  = new Date(); weekEnd.setDate(weekEnd.getDate() + 7)
      const weekStr  = weekEnd.toISOString().slice(0, 10)

      const todayTasks  = this.tasks.filter(t => !t.deadline || t.deadline <= todayStr)
      const weekTasks   = this.tasks.filter(t => t.deadline > todayStr && t.deadline <= weekStr)
      const laterTasks  = this.tasks.filter(t => t.deadline > weekStr)

      const groups = []
      if (todayTasks.length)  groups.push({ label: 'Azi', tasks: todayTasks })
      if (weekTasks.length)   groups.push({ label: 'Saptamana aceasta', tasks: weekTasks })
      if (laterTasks.length)  groups.push({ label: 'Mai tarziu', tasks: laterTasks })
      if (!groups.length && this.tasks.length) groups.push({ label: 'Toate', tasks: this.tasks })
      return groups
    },
  },
  methods: {
    formatDate(iso) {
      if (!iso) return ''
      const d = new Date(iso + 'T00:00:00')
      const today = new Date(); today.setHours(0, 0, 0, 0)
      const diff = Math.round((d - today) / 86400000)
      if (diff === 0) return 'azi'
      if (diff === 1) return 'maine'
      if (diff < 0)  return `${Math.abs(diff)}z intarziere`
      return d.toLocaleDateString('ro-RO', { weekday: 'short', day: 'numeric', month: 'short' })
    },
    priorityClass(p) {
      return { P1: 'is-priority-p1', P2: 'is-priority-p2', P3: 'is-priority-p3' }[p] || ''
    },
    // In the Today view tasks come from multiple lists, so check per-task
    isTaskShared(task) {
      return this.sharedListNames.includes(task.list)
    },
  },
}
</script>

<style scoped>
.task-enter-active, .task-leave-active { transition: all 0.18s ease; }
.task-enter-from,  .task-leave-to      { opacity: 0; transform: translateY(-4px); }
</style>
