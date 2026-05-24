<template>
  <div class="flex h-screen overflow-hidden" style="background: #CACACA;">
    <LeftSidebar
      :lists="lists"
      :tasks="tasks"
      :today-tasks="todayTasks"
      :active-list="activeList"
      @select-list="activeList = $event"
      @add-list="addList"
      @toggle-task="toggleTask"
      @add-today-task="addTodayTask"
    />
    <MainContent
      :tasks="filteredTasks"
      :search-query="searchQuery"
      :active-filter="activeFilter"
      :lists="lists"
      :active-list="activeList"
      @update:search-query="searchQuery = $event"
      @update:active-filter="activeFilter = $event"
      @add-task="addTask"
      @toggle-task="toggleTask"
      @update-task="updateTask"
      @delete-task="deleteTask"
    />
    <RightSidebar
      :tasks="tasks"
      :completed-count="completedCount"
    />
  </div>
</template>

<script>
import LeftSidebar from './components/LeftSidebar.vue'
import MainContent from './components/MainContent.vue'
import RightSidebar from './components/RightSidebar.vue'

const TASKS_KEY = 'doer-tasks'
const LISTS_KEY = 'doer-lists'

const DEFAULT_LISTS = ['Inbox', 'Personal', 'Work', 'Health', 'Shopping', 'Projects']

const DEFAULT_TASKS = [
  { id: '1', title: 'Review wireframe designs', deadline: '2026-05-24', priority: 'P1', list: 'Work', done: false },
  { id: '2', title: 'Set up project repository', deadline: '2026-05-24', priority: 'P2', list: 'Work', done: true },
  { id: '3', title: 'Write component specifications', deadline: '2026-05-25', priority: 'P2', list: 'Work', done: false },
  { id: '4', title: 'Conduct user research session', deadline: '2026-05-25', priority: 'P1', list: 'Personal', done: false },
  { id: '5', title: 'Update project roadmap', deadline: '2026-05-26', priority: 'P3', list: 'Work', done: false },
  { id: '6', title: 'Fix navigation bug on mobile', deadline: '2026-05-26', priority: 'P1', list: 'Projects', done: false },
  { id: '7', title: 'Prepare weekly report', deadline: '2026-05-27', priority: 'P3', list: 'Work', done: false },
  { id: '8', title: 'Schedule stakeholder review', deadline: '2026-05-28', priority: 'P2', list: 'Work', done: false },
]

function migrateTasks(tasks) {
  return tasks.map(t => ({ list: 'Inbox', ...t }))
}

export default {
  name: 'App',
  components: { LeftSidebar, MainContent, RightSidebar },

  data() {
    const savedTasks = localStorage.getItem(TASKS_KEY)
    const savedLists = localStorage.getItem(LISTS_KEY)
    return {
      tasks: savedTasks ? migrateTasks(JSON.parse(savedTasks)) : DEFAULT_TASKS,
      lists: savedLists ? JSON.parse(savedLists) : DEFAULT_LISTS,
      searchQuery: '',
      activeFilter: 'ALL',
      activeList: 'Today',
    }
  },

  computed: {
    filteredTasks() {
      return this.tasks.filter(task => {
        const matchesList = this.activeList === 'Today' || task.list === this.activeList
        const matchesPriority = this.activeFilter === 'ALL' || task.priority === this.activeFilter
        const q = this.searchQuery.toLowerCase()
        const matchesSearch = !q || task.title.toLowerCase().includes(q) || (task.list || '').toLowerCase().includes(q)
        return matchesList && matchesPriority && matchesSearch
      })
    },
    completedCount() {
      return this.tasks.filter(t => t.done).length
    },
    todayTasks() {
      const today = new Date().toISOString().slice(0, 10)
      return this.tasks
        .filter(t => t.deadline === today || (t.deadline < today && !t.done))
        .sort((a, b) => {
          if (a.done !== b.done) return a.done ? 1 : -1
          const order = { P1: 0, P2: 1, P3: 2, P4: 3 }
          return (order[a.priority] ?? 4) - (order[b.priority] ?? 4)
        })
    },
  },

  watch: {
    tasks: {
      deep: true,
      handler(val) { localStorage.setItem(TASKS_KEY, JSON.stringify(val)) },
    },
    lists: {
      deep: true,
      handler(val) { localStorage.setItem(LISTS_KEY, JSON.stringify(val)) },
    },
  },

  methods: {
    addTask(title) {
      const today = new Date().toISOString().slice(0, 10)
      this.tasks.push({
        id: String(Date.now()),
        title,
        deadline: today,
        priority: 'P4',
        list: this.activeList === 'Today' ? 'Inbox' : this.activeList,
        done: false,
      })
    },
    toggleTask(id) {
      const task = this.tasks.find(t => t.id === id)
      if (task) task.done = !task.done
    },
    updateTask(updated) {
      const idx = this.tasks.findIndex(t => t.id === updated.id)
      if (idx !== -1) this.tasks[idx] = { ...this.tasks[idx], ...updated }
    },
    deleteTask(id) {
      this.tasks = this.tasks.filter(t => t.id !== id)
    },
    addTodayTask(title) {
      const today = new Date().toISOString().slice(0, 10)
      this.tasks.push({
        id: String(Date.now()),
        title,
        deadline: today,
        priority: 'P4',
        list: this.activeList === 'Today' ? 'Inbox' : this.activeList,
        done: false,
      })
    },
    addList(name) {
      const trimmed = name.trim()
      if (trimmed && !this.lists.includes(trimmed)) {
        this.lists.push(trimmed)
      }
    },
  },
}
</script>
