<template>
  <main class="flex-1 flex flex-col overflow-hidden py-3.5 px-8" style="background: var(--color-app-bg);">
    <!-- Top search / filter bar -->
    <div
      class="flex items-center gap-3 px-4 rounded-[24.5px] border border-black mb-5 shrink-0"
      style="height: 59px; background: var(--color-app-bg);"
    >
      <svg class="w-4 h-4 text-gray-500 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
        <circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/>
      </svg>
      <input
        type="text"
        :value="searchQuery"
        placeholder="Search tasks..."
        class="flex-1 bg-transparent text-sm outline-none placeholder-gray-500"
        @input="$emit('update:search-query', $event.target.value)"
      />
      <div class="flex items-center gap-1.5">
        <button
          v-for="c in quickChips"
          :key="c"
          :class="['tag-btn', { 'is-active': activeFilter === c }]"
          style="height: 28px;"
          @click="$emit('update:active-filter', c)"
        >{{ c }}</button>
      </div>
    </div>

    <!-- Stat cards -->
    <div class="grid grid-cols-3 gap-4 mb-5 shrink-0">
      <div v-for="card in statCards" :key="card.title" class="stat-card">
        <p class="section-label">{{ card.title }}</p>
        <p class="text-3xl font-black leading-none">{{ card.value }}</p>
        <p class="text-xs text-gray-500">{{ card.sub }}</p>
      </div>
    </div>

    <!-- Filter chips row -->
    <div class="flex items-center gap-1.5 mb-3 flex-wrap shrink-0">
      <button
        v-for="chip in priorityChips"
        :key="chip"
        :class="['filter-chip', { 'is-active': activeFilter === chip }]"
        @click="$emit('update:active-filter', chip)"
      >{{ chip }}</button>
      <span class="ml-1 text-gray-400 text-xs">{{ tasks.length }} task{{ tasks.length !== 1 ? 's' : '' }}</span>
    </div>

    <!-- Task add row -->
    <div class="task-row mb-2 shrink-0" style="cursor: default;">
      <div class="task-checkbox"></div>
      <input
        v-model="newTitle"
        type="text"
        :placeholder="addPlaceholder"
        class="flex-1 bg-transparent text-sm italic text-gray-500 placeholder-gray-400 outline-none"
        @keydown.enter.prevent="submitNewTask"
      />
      <button
        v-if="newTitle.trim()"
        class="text-xs px-3 py-1 rounded-full bg-black text-white cursor-pointer border-0 shrink-0"
        @click="submitNewTask"
      >Add</button>
    </div>

    <!-- Task list -->
    <div class="flex flex-col gap-[7px] overflow-y-auto flex-1 pb-4">
      <transition-group name="task">
        <div
          v-for="task in tasks"
          :key="task.id"
          :class="['task-row group relative', { 'is-selected': selectedId === task.id }]"
          @click="selectedId = selectedId === task.id ? null : task.id"
        >
          <!-- Checkbox -->
          <button
            :class="['task-checkbox', { 'is-done': task.done }]"
            @click.stop="$emit('toggle-task', task.id)"
          >
            <svg v-if="task.done" viewBox="0 0 10 8" class="w-3 h-3" fill="none" stroke="white" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
              <path d="M1 4l3 3 5-6"/>
            </svg>
          </button>

          <!-- Title -->
          <span
            v-if="editingId !== task.id"
            :class="['flex-1 text-sm italic truncate', task.done ? 'line-through text-gray-400' : '']"
            @dblclick.stop="startEdit(task)"
          >{{ task.title }}</span>
          <input
            v-else
            :id="'edit-' + task.id"
            type="text"
            :value="editingTitle"
            class="flex-1 text-sm italic bg-transparent outline-none border-b border-black"
            @input="editingTitle = $event.target.value"
            @blur="saveEdit(task)"
            @keydown.enter.prevent="saveEdit(task)"
            @keydown.escape.prevent="cancelEdit"
            @click.stop
          />

          <!-- Date picker -->
          <input
            type="date"
            :value="task.deadline"
            class="text-xs text-gray-500 bg-transparent outline-none cursor-pointer shrink-0 w-28"
            @change.stop="$emit('update-task', { id: task.id, deadline: $event.target.value })"
            @click.stop
          />

          <!-- Priority button -->
          <button
            :class="['tag-btn', { 'is-priority': task.priority === 'P1' }]"
            style="font-weight: 600;"
            @click.stop="cyclePriority(task)"
            title="Click to change priority"
          >{{ task.priority }}</button>

          <!-- List tag -->
          <div class="relative shrink-0" @click.stop>
            <button
              class="tag-btn"
              :title="task.list || 'No list'"
              @click="listDropdownId = listDropdownId === task.id ? null : task.id"
            >{{ task.list ? task.list.slice(0, 7) : '—' }}</button>

            <div
              v-if="listDropdownId === task.id"
              class="absolute right-0 top-full mt-1 bg-white border border-black rounded-xl shadow-lg z-20 py-1 overflow-hidden"
              style="min-width: 120px;"
            >
              <button
                v-for="l in allLists"
                :key="l.value"
                :class="['w-full text-left text-xs px-3 py-1.5 cursor-pointer border-0 bg-transparent', task.list === l.value ? 'font-bold bg-gray-100' : 'hover:bg-gray-50']"
                @click="changeList(task, l.value)"
              >{{ l.label }}</button>
            </div>
          </div>

          <!-- Delete -->
          <button
            class="text-gray-400 hover:text-black cursor-pointer border-0 bg-transparent opacity-0 group-hover:opacity-100 transition-opacity shrink-0 text-xl leading-none"
            @click.stop="$emit('delete-task', task.id)"
            title="Delete"
          >×</button>
        </div>
      </transition-group>

      <p v-if="tasks.length === 0" class="text-center text-gray-400 text-sm mt-8">No tasks found</p>
    </div>

    <div v-if="listDropdownId" class="fixed inset-0 z-10" @click="listDropdownId = null"></div>
  </main>
</template>

<script>
const PRIORITIES = ['P1', 'P2', 'P3', 'P4']

export default {
  name: 'MainContent',
  emits: ['update:search-query', 'update:active-filter', 'add-task', 'toggle-task', 'update-task', 'delete-task'],
  props: {
    tasks:        { type: Array,  default: () => [] },
    searchQuery:  { type: String, default: '' },
    activeFilter: { type: String, default: 'ALL' },
    lists:        { type: Array,  default: () => [] },
    activeList:   { type: String, default: 'Today' },
  },
  data() {
    return {
      newTitle:       '',
      editingId:      null,
      editingTitle:   '',
      selectedId:     null,
      listDropdownId: null,
      priorityChips:  ['ALL', 'P1', 'P2', 'P3', 'P4'],
      quickChips:     ['ALL', 'P1', 'P2', 'P3'],
    }
  },
  computed: {
    allLists() {
      return [
        { label: '— No list', value: '' },
        { label: 'Inbox',     value: 'Inbox' },
        ...this.lists.filter(l => l !== 'Inbox').map(l => ({ label: l, value: l })),
      ]
    },
    addPlaceholder() {
      return this.activeList === 'Today' ? 'Title it...' : `Add to ${this.activeList}...`
    },
    statCards() {
      const total = this.tasks.length
      const done  = this.tasks.filter(t => t.done).length
      const p1    = this.tasks.filter(t => t.priority === 'P1' && !t.done).length
      return [
        { title: 'Total Tasks',    value: String(total), sub: 'in this view' },
        { title: 'Completed',      value: String(done),  sub: `${total ? Math.round(done / total * 100) : 0}% done` },
        { title: 'High Priority',  value: String(p1),    sub: 'P1 remaining' },
      ]
    },
  },
  methods: {
    submitNewTask() {
      const title = this.newTitle.trim()
      if (!title) return
      this.$emit('add-task', title)
      this.newTitle = ''
    },
    startEdit(task) {
      this.editingId    = task.id
      this.editingTitle = task.title
      this.$nextTick(() => document.getElementById('edit-' + task.id)?.focus())
    },
    saveEdit(task) {
      const title = this.editingTitle.trim()
      if (title && title !== task.title) this.$emit('update-task', { id: task.id, title })
      this.editingId = null
    },
    cancelEdit() { this.editingId = null },
    cyclePriority(task) {
      const next = PRIORITIES[(PRIORITIES.indexOf(task.priority) + 1) % PRIORITIES.length]
      this.$emit('update-task', { id: task.id, priority: next })
    },
    changeList(task, list) {
      this.$emit('update-task', { id: task.id, list })
      this.listDropdownId = null
    },
  },
}
</script>

<style scoped>
.task-enter-active, .task-leave-active { transition: all 0.2s ease; }
.task-enter-from,  .task-leave-to      { opacity: 0; transform: translateY(-6px); }
</style>
