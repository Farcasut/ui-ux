<template>
  <div class="flex h-screen overflow-hidden" style="background: #CACACA;">
    <LeftSidebar
      :lists="lists"
      :shared-lists="sharedLists"
      :tasks="tasks"
      :active-list="activeList"
      @select-list="selectList"
      @add-list="addList"
    />
    <MainContent
      :tasks="filteredTasks"
      :all-tasks="tasks"
      :lists="allListNames"
      :active-list="activeList"
      :selected-task-id="selectedTaskId"
      :search-query="searchQuery"
      :active-filter="activeFilter"
      @update:search-query="searchQuery = $event"
      @update:active-filter="activeFilter = $event"
      @select-task="selectedTaskId = $event"
      @open-add-modal="openAddModal"
      @open-edit-modal="openEditModal"
      @toggle-task="toggleTask"
      @update-task="updateTask"
      @delete-task="deleteTask"
    />
    <RightSidebar
      :tasks="tasks"
      :completed-count="completedCount"
      :active-list="activeList"
      :selected-task="selectedTask"
      @update-task="updateTask"
      @open-edit-modal="openEditModal"
    />
    <TaskModal
      v-if="showModal"
      :task="modalTask"
      :lists="allListNames"
      @save="saveModalTask"
      @close="showModal = false"
    />
  </div>
</template>

<script>
import LeftSidebar from './components/LeftSidebar.vue'
import MainContent from './components/MainContent.vue'
import RightSidebar from './components/RightSidebar.vue'
import TaskModal from './components/TaskModal.vue'

const TASKS_KEY = 'doer-tasks-v2'
const LISTS_KEY = 'doer-lists-v2'

const DEFAULT_LISTS = ['Munca', 'Facultate', 'Personal']
const DEFAULT_SHARED_LISTS = ['Cumparaturi', 'UI/UX Project', 'Treburi']

const today = new Date().toISOString().slice(0, 10)
const nextDay = d => { const dt = new Date(d); dt.setDate(dt.getDate() + 1); return dt.toISOString().slice(0, 10) }
const inDays = n => { const dt = new Date(); dt.setDate(dt.getDate() + n); return dt.toISOString().slice(0, 10) }

const DEFAULT_TASKS = [
  { id: '1', title: 'Finalizare wireframes',    deadline: today,       time: '18:00', priority: 'P1', list: 'Facultate', done: false, assignee: 'Mihai',  reminder: 'Cu 2 ore inainte', notes: '', tags: [] },
  { id: '2', title: 'Trimite documentatia',     deadline: today,       time: '22:00', priority: 'P1', list: 'Facultate', done: false, assignee: 'Andrei', reminder: '',                 notes: '', tags: [] },
  { id: '3', title: 'Corecteaza user stories',  deadline: today,       time: '',      priority: 'P2', list: 'Facultate', done: true,  assignee: 'Alex',   reminder: '',                 notes: '', tags: [] },
  { id: '4', title: 'Pregateste prezentarea',   deadline: inDays(5),   time: '',      priority: 'P2', list: 'Facultate', done: false, assignee: 'Fabian', reminder: '',                 notes: '', tags: [] },
  { id: '5', title: 'Revizuire finala',         deadline: inDays(6),   time: '',      priority: 'P3', list: 'Facultate', done: false, assignee: 'echipa', reminder: '',                 notes: '', tags: [] },
  { id: '6', title: 'Review PR',                deadline: today,       time: '19:30', priority: 'P1', list: 'Munca',     done: false, assignee: 'Mihai',  reminder: '',                 notes: '', tags: [] },
  { id: '7', title: 'Send email',               deadline: today,       time: '20:00', priority: 'P2', list: 'Munca',     done: false, assignee: 'Mihai',  reminder: '',                 notes: '', tags: [] },
  { id: '8', title: 'Doctor Appointment',       deadline: inDays(1),   time: '07:25', priority: 'P2', list: 'Personal',  done: false, assignee: 'Mihai',  reminder: 'Cu 1 ora inainte', notes: '', tags: [] },
  { id: '9', title: 'Cumparaturi saptamanale',  deadline: inDays(2),   time: '',      priority: 'P3', list: 'Cumparaturi', done: false, assignee: 'Mihai', reminder: '',                notes: '', tags: [] },
]

export default {
  name: 'App',
  components: { LeftSidebar, MainContent, RightSidebar, TaskModal },

  data() {
    const savedTasks = localStorage.getItem(TASKS_KEY)
    const savedLists = localStorage.getItem(LISTS_KEY)
    return {
      tasks:         savedTasks ? JSON.parse(savedTasks) : DEFAULT_TASKS,
      lists:         savedLists ? JSON.parse(savedLists) : DEFAULT_LISTS,
      sharedLists:   DEFAULT_SHARED_LISTS,
      searchQuery:   '',
      activeFilter:  'ALL',
      activeList:    'Azi',
      selectedTaskId: null,
      showModal:     false,
      modalTask:     null,
    }
  },

  computed: {
    todayStr() {
      return new Date().toISOString().slice(0, 10)
    },
    allListNames() {
      return [...this.lists, ...this.sharedLists]
    },
    filteredTasks() {
      return this.tasks.filter(task => {
        const matchesList = this.activeList === 'Azi'
          ? (task.deadline === this.todayStr || (task.deadline < this.todayStr && !task.done))
          : task.list === this.activeList
        const matchesPriority = this.activeFilter === 'ALL' || task.priority === this.activeFilter
        const q = this.searchQuery.toLowerCase()
        const matchesSearch = !q || task.title.toLowerCase().includes(q) || (task.list || '').toLowerCase().includes(q)
        return matchesList && matchesPriority && matchesSearch
      })
    },
    completedCount() {
      return this.tasks.filter(t => t.done).length
    },
    selectedTask() {
      return this.tasks.find(t => t.id === this.selectedTaskId) || null
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
    activeList() {
      this.selectedTaskId = null
      this.activeFilter = 'ALL'
      this.searchQuery = ''
    },
  },

  methods: {
    selectList(list) {
      this.activeList = list
    },
    openAddModal(listName) {
      this.modalTask = {
        title: '', list: listName || this.lists[0] || '',
        priority: 'P2', deadline: this.todayStr, time: '',
        reminder: '', assignee: '', notes: '', tags: [],
      }
      this.showModal = true
    },
    openEditModal(task) {
      this.modalTask = { ...task }
      this.showModal = true
    },
    saveModalTask(taskData) {
      if (taskData.id) {
        const idx = this.tasks.findIndex(t => t.id === taskData.id)
        if (idx !== -1) this.tasks[idx] = { ...this.tasks[idx], ...taskData }
      } else {
        this.tasks.push({ id: String(Date.now()), done: false, tags: [], ...taskData })
      }
      this.showModal = false
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
      if (this.selectedTaskId === id) this.selectedTaskId = null
    },
    addList(name) {
      const trimmed = name.trim()
      if (trimmed && !this.lists.includes(trimmed)) this.lists.push(trimmed)
    },
  },
}
</script>
