<template>
  <div class="flex h-screen overflow-hidden relative" style="background: #CACACA;">
    <Transition name="sidebar">
      <LeftSidebar
        v-if="sidebarOpen"
        :lists="personalLists"
        :shared-lists="allSharedLists"
        :tasks="tasks"
        :active-list="activeList"
        @select-list="selectList"
        @add-list="addList"
        @delete-list="deleteList"
        @toggle="sidebarOpen = false"
      />
    </Transition>

    <MainContent
      :tasks="filteredTasks"
      :all-tasks="tasks"
      :lists="allListNames"
      :active-list="activeList"
      :selected-task-id="selectedTaskId"
      :search-query="searchQuery"
      :active-filter="activeFilter"
      :is-shared="activeListIsShared"
      :shared-list-names="allSharedListNames"
      :sidebar-open="sidebarOpen"
      @update:search-query="searchQuery = $event"
      @update:active-filter="activeFilter = $event"
      @select-task="selectedTaskId = $event"
      @open-add-modal="openAddModal"
      @open-edit-modal="openEditModal"
      @open-share-modal="openShareModal"
      @toggle-task="toggleTask"
      @update-task="updateTask"
      @delete-task="deleteTask"
      @toggle-sidebar="sidebarOpen = true"
    />
    <RightSidebar
      :tasks="tasks"
      :view-tasks="filteredTasks"
      :completed-count="completedCount"
      :active-list="activeList"
      :selected-task="selectedTask"
      :is-shared="activeListIsShared"
      @update-task="updateTask"
      @open-edit-modal="openEditModal"
      @clear-selection="selectedTaskId = null"
      @delete-task="deleteTask"
    />
    <TaskModal
      v-if="showModal"
      :task="modalTask"
      :lists="allListNames"
      :shared-list-names="allSharedListNames"
      :share-data="shareData"
      @save="saveModalTask"
      @close="showModal = false"
    />
    <ShareModal
      v-if="showShareModal"
      :list-name="shareModalList"
      :collaborators="shareData[shareModalList] || []"
      @update-collaborators="updateShareCollaborators"
      @close="showShareModal = false"
    />
  </div>
</template>

<script>
import LeftSidebar from './components/LeftSidebar.vue'
import MainContent from './components/MainContent.vue'
import RightSidebar from './components/RightSidebar.vue'
import TaskModal from './components/TaskModal.vue'
import ShareModal from './components/ShareModal.vue'

const TASKS_KEY = 'doer-tasks-v2'
const LISTS_KEY = 'doer-lists-v2'

const DEFAULT_LISTS = ['Munca', 'Facultate', 'Personal']
const BUILTIN_SHARED_LISTS = ['Cumparaturi', 'UI/UX Project', 'Treburi']

const CURRENT_USER = 'Mihai'
const today = new Date().toISOString().slice(0, 10)
const inDays = n => { const dt = new Date(); dt.setDate(dt.getDate() + n); return dt.toISOString().slice(0, 10) }

const DEFAULT_TASKS = [
  { id: '1', title: 'Finalizare wireframes',    deadline: today,       time: '18:00', priority: 'P1', list: 'Facultate',   done: false, assignee: 'Mihai',  reminder: 'Cu 2 ore inainte', notes: '', tags: [] },
  { id: '2', title: 'Trimite documentatia',     deadline: today,       time: '22:00', priority: 'P1', list: 'Facultate',   done: false, assignee: 'Andrei', reminder: '',                 notes: '', tags: [] },
  { id: '3', title: 'Corecteaza user stories',  deadline: today,       time: '',      priority: 'P2', list: 'Facultate',   done: true,  assignee: 'Alex',   reminder: '',                 notes: '', tags: [] },
  { id: '4', title: 'Pregateste prezentarea',   deadline: inDays(5),   time: '',      priority: 'P2', list: 'Facultate',   done: false, assignee: 'Fabian', reminder: '',                 notes: '', tags: [] },
  { id: '5', title: 'Revizuire finala',         deadline: inDays(6),   time: '',      priority: 'P3', list: 'Facultate',   done: false, assignee: 'echipa', reminder: '',                 notes: '', tags: [] },
  { id: '6', title: 'Review PR',                deadline: today,       time: '19:30', priority: 'P1', list: 'Munca',       done: false, assignee: 'Mihai',  reminder: '',                 notes: '', tags: [] },
  { id: '7', title: 'Send email',               deadline: today,       time: '20:00', priority: 'P2', list: 'Munca',       done: false, assignee: 'Mihai',  reminder: '',                 notes: '', tags: [] },
  { id: '8', title: 'Doctor Appointment',       deadline: inDays(1),   time: '07:25', priority: 'P2', list: 'Personal',    done: false, assignee: 'Mihai',  reminder: 'Cu 1 ora inainte', notes: '', tags: [] },
  { id: '9', title: 'Cumparaturi saptamanale',  deadline: inDays(2),   time: '',      priority: 'P3', list: 'Cumparaturi', done: false, assignee: 'Mihai',  reminder: '',                 notes: '', tags: [] },
]

export default {
  name: 'App',
  components: { LeftSidebar, MainContent, RightSidebar, TaskModal, ShareModal },

  data() {
    const savedTasks = localStorage.getItem(TASKS_KEY)
    const savedLists = localStorage.getItem(LISTS_KEY)
    return {
      tasks:          savedTasks ? JSON.parse(savedTasks) : DEFAULT_TASKS,
      lists:          savedLists ? JSON.parse(savedLists) : DEFAULT_LISTS,
      searchQuery:    '',
      activeFilter:   'ALL',
      activeList:     'Azi',
      selectedTaskId: null,
      showModal:      false,
      modalTask:      null,
      sidebarOpen:    true,
      showShareModal: false,
      shareModalList: null,
      shareData: {
        Cumparaturi: [
          { email: 'mihai@email.com',  role: 'Owner',       permission: 'write-and-share' },
          { email: 'alex@email.com',   role: 'Colaborator', permission: 'write' },
          { email: 'andrei@email.com', role: 'Colaborator', permission: 'write' },
        ],
        'UI/UX Project': [
          { email: 'mihai@email.com',  role: 'Owner',       permission: 'write-and-share' },
          { email: 'alex@email.com',   role: 'Colaborator', permission: 'write' },
          { email: 'andrei@email.com', role: 'Colaborator', permission: 'write' },
        ],
        Treburi: [
          { email: 'mihai@email.com',  role: 'Owner',       permission: 'write-and-share' },
          { email: 'alex@email.com',   role: 'Colaborator', permission: 'write' },
          { email: 'andrei@email.com', role: 'Colaborator', permission: 'write' },
        ],
      },
    }
  },

  computed: {
    todayStr() {
      return new Date().toISOString().slice(0, 10)
    },
    // Lists that have been shared (have at least 1 collaborator beyond the owner)
    sharedUserLists() {
      return this.lists.filter(l => (this.shareData[l] || []).length > 1)
    },
    // User-created lists that are still personal (no collaborators yet)
    personalLists() {
      return this.lists.filter(l => !this.sharedUserLists.includes(l))
    },
    // All lists shown in "Liste partajate": built-ins + any user list that got shared
    allSharedLists() {
      return [...BUILTIN_SHARED_LISTS, ...this.sharedUserLists]
    },
    // Flat array of shared list names — passed to TaskModal
    allSharedListNames() {
      return this.allSharedLists
    },
    // All list names for dropdowns
    allListNames() {
      return [...this.lists, ...BUILTIN_SHARED_LISTS]
    },
    activeListIsShared() {
      return this.allSharedLists.includes(this.activeList)
    },
    filteredTasks() {
      return this.tasks.filter(task => {
        if (this.activeList === 'Azi') {
          const isToday = task.deadline === this.todayStr || (task.deadline < this.todayStr && !task.done)
          if (!isToday) return false
          // On shared lists, only show tasks assigned to me or unassigned
          const isSharedList = this.allSharedLists.includes(task.list)
          if (isSharedList && task.assignee && task.assignee !== CURRENT_USER) return false
        } else {
          if (task.list !== this.activeList) return false
        }
        const matchesPriority = this.activeFilter === 'ALL' || task.priority === this.activeFilter
        const q = this.searchQuery.toLowerCase()
        const matchesSearch = !q || task.title.toLowerCase().includes(q) || (task.list || '').toLowerCase().includes(q)
        return matchesPriority && matchesSearch
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
    deleteList(name) {
      this.lists = this.lists.filter(l => l !== name)
      this.tasks = this.tasks.filter(t => t.list !== name)
      delete this.shareData[name]
      if (this.activeList === name) this.activeList = 'Azi'
    },
    openShareModal(listName) {
      if (!this.shareData[listName]) {
        this.shareData[listName] = [{ email: 'mihai@email.com', role: 'Owner', permission: 'write-and-share' }]
      }
      this.shareModalList = listName
      this.showShareModal = true
    },
    updateShareCollaborators(collaborators) {
      this.shareData[this.shareModalList] = collaborators
    },
  },
}
</script>
