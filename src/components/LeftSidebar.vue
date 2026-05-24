<template>
  <aside class="sidebar">
    <!-- DOER Logo -->
    <div class="px-5 pt-5 pb-4 shrink-0">
      <span class="text-5xl font-black tracking-tight leading-none">DOER</span>
    </div>

    <div class="sidebar-divider"></div>

    <!-- TODAY section -->
    <div class="px-4 pt-4 pb-3 shrink-0">
      <div class="flex items-center justify-between mb-2">
        <p class="section-label">Today</p>
        <span class="text-[10px] text-gray-400">{{ todayTasks.filter(t => !t.done).length }} left</span>
      </div>

      <div class="overflow-y-auto flex flex-col gap-0.5" style="max-height: 160px;">
        <div
          v-for="(task, i) in todayTasks"
          :key="task.id"
          :class="['nav-item', { 'opacity-50': task.done }]"
          @click="$emit('toggle-task', task.id)"
        >
          <span class="text-[10px] text-gray-400 w-3 text-right shrink-0">{{ i + 1 }}</span>
          <div :class="['task-checkbox shrink-0', { 'is-done': task.done }]" style="width:16px; height:16px; border-radius:3px; min-width:16px;">
            <svg v-if="task.done" viewBox="0 0 10 8" class="w-2.5 h-2.5" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M1 4l3 3 5-6"/>
            </svg>
          </div>
          <span :class="['text-sm truncate flex-1', task.done ? 'line-through text-gray-400' : '']">{{ task.title }}</span>
          <span :class="['w-1.5 h-1.5 rounded-full shrink-0', priorityDot(task.priority)]"></span>
        </div>

        <p v-if="todayTasks.length === 0" class="text-xs text-gray-400 px-2 py-1">No tasks for today</p>
      </div>

      <!-- Quick add for today -->
      <div class="mt-2">
        <div v-if="!showTodayAdd" class="nav-item" @click="openTodayAdd">
          <span class="text-gray-400 text-sm leading-none">+</span>
          <span class="text-xs text-gray-400">Add for today</span>
        </div>
        <input
          v-else
          ref="todayAddInput"
          v-model="todayNewTitle"
          type="text"
          placeholder="Task title..."
          class="w-full text-xs border border-black rounded px-2 py-1 outline-none"
          style="background: var(--color-surface);"
          @keydown.enter.prevent="submitTodayTask"
          @keydown.escape.prevent="cancelTodayAdd"
          @blur="cancelTodayAdd"
        />
      </div>
    </div>

    <div class="sidebar-divider"></div>

    <!-- LIST section -->
    <div class="flex-1 overflow-y-auto px-4 pt-4 pb-2">
      <div class="flex items-center justify-between mb-2">
        <p class="section-label">List</p>
        <button
          class="w-5 h-5 flex items-center justify-center rounded-full bg-black text-white text-xs leading-none cursor-pointer border-0 hover:bg-gray-700"
          title="New list"
          @click="showNewList = true"
        >+</button>
      </div>

      <input
        v-if="showNewList"
        ref="newListInput"
        v-model="newListName"
        type="text"
        placeholder="List name..."
        class="w-full text-sm border border-black rounded px-2 py-1 outline-none mb-1"
        style="background: var(--color-surface);"
        @keydown.enter.prevent="submitNewList"
        @keydown.escape.prevent="cancelNewList"
        @blur="cancelNewList"
      />

      <div :class="['nav-item mb-0.5', { 'is-active': activeList === 'Today' }]" @click="$emit('select-list', 'Today')">
        <span class="truncate flex-1">Today</span>
        <span class="text-[10px] text-gray-400">{{ tasks.length }}</span>
      </div>

      <nav class="flex flex-col gap-0.5">
        <div
          v-for="list in lists"
          :key="list"
          :class="['nav-item', { 'is-active': activeList === list }]"
          @click="$emit('select-list', list)"
        >
          <span class="truncate flex-1">{{ list }}</span>
          <span class="text-[10px] text-gray-400 shrink-0">{{ listCount(list) }}</span>
        </div>
      </nav>
    </div>

    <div class="sidebar-divider"></div>

    <div class="px-4 py-3 shrink-0">
      <nav class="flex flex-col gap-0.5">
        <div v-for="item in bottomItems" :key="item" class="nav-item">
          <span>{{ item }}</span>
        </div>
      </nav>
    </div>
  </aside>
</template>

<script>
export default {
  name: 'LeftSidebar',
  emits: ['select-list', 'add-list', 'toggle-task', 'add-today-task'],
  props: {
    activeList:  { type: String, default: 'Today' },
    lists:       { type: Array,  default: () => [] },
    tasks:       { type: Array,  default: () => [] },
    todayTasks:  { type: Array,  default: () => [] },
  },
  data() {
    return {
      showNewList:    false,
      newListName:    '',
      showTodayAdd:   false,
      todayNewTitle:  '',
      bottomItems:    ['Settings', 'Archive'],
    }
  },
  methods: {
    listCount(listName) {
      const n = this.tasks.filter(t => t.list === listName && !t.done).length
      return n || ''
    },
    priorityDot(p) {
      return { P1: 'bg-black', P2: 'bg-gray-600', P3: 'bg-gray-400', P4: 'bg-gray-300' }[p] || 'bg-gray-300'
    },
    openTodayAdd() {
      this.showTodayAdd = true
      this.$nextTick(() => this.$refs.todayAddInput?.focus())
    },
    submitTodayTask() {
      const title = this.todayNewTitle.trim()
      if (title) this.$emit('add-today-task', title)
      this.todayNewTitle = ''
      this.showTodayAdd = false
    },
    cancelTodayAdd() {
      this.todayNewTitle = ''
      this.showTodayAdd = false
    },
    submitNewList() {
      if (this.newListName.trim()) {
        this.$emit('add-list', this.newListName)
        this.newListName = ''
        this.showNewList = false
      }
    },
    cancelNewList() {
      this.newListName = ''
      this.showNewList = false
    },
  },
  watch: {
    showNewList(val) {
      if (val) this.$nextTick(() => this.$refs.newListInput?.focus())
    },
  },
}
</script>
