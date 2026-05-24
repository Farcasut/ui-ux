<template>
  <aside class="sidebar">
    <!-- Logo -->
    <div class="px-5 pt-6 pb-5 shrink-0">
      <span class="text-3xl font-black tracking-tight">Doer</span>
    </div>

    <div class="sidebar-divider"></div>

    <!-- Primary nav -->
    <nav class="px-3 pt-3 pb-2 flex flex-col gap-0.5 shrink-0">
      <div
        :class="['nav-item', { 'is-active': activeList === 'Azi' }]"
        @click="$emit('select-list', 'Azi')"
      >
        <svg class="w-4 h-4 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/>
        </svg>
        <span class="flex-1">Azi</span>
        <span v-if="todayCount > 0" class="text-[10px]" :class="activeList === 'Azi' ? 'text-white/70' : 'text-gray-400'">{{ todayCount }}</span>
      </div>
      <div class="nav-item opacity-40 cursor-default select-none">
        <svg class="w-4 h-4 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <path d="M4 4h16v16H4z"/><path d="M4 9h16"/>
        </svg>
        <span>Inbox</span>
      </div>
      <div class="nav-item opacity-40 cursor-default select-none">
        <svg class="w-4 h-4 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 3"/>
        </svg>
        <span>Calendar</span>
      </div>
      <div class="nav-item opacity-40 cursor-default select-none">
        <svg class="w-4 h-4 shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <path d="M18 20V10M12 20V4M6 20v-6"/>
        </svg>
        <span>Progress</span>
      </div>
    </nav>

    <div class="sidebar-divider"></div>

    <!-- My Lists -->
    <div class="px-3 pt-3 pb-2 flex flex-col gap-0.5 shrink-0">
      <div class="flex items-center justify-between px-3 mb-1">
        <span class="section-label">Liste</span>
        <button
          class="w-5 h-5 flex items-center justify-center rounded-full bg-black text-white text-sm leading-none cursor-pointer border-0 hover:bg-gray-700"
          title="Lista noua"
          @click="showNewList = true"
        >+</button>
      </div>

      <input
        v-if="showNewList"
        ref="newListInput"
        v-model="newListName"
        type="text"
        placeholder="Nume lista..."
        class="mx-1 text-sm border border-black rounded-lg px-3 py-1.5 outline-none mb-1"
        style="background: var(--color-app-bg);"
        @keydown.enter.prevent="submitNewList"
        @keydown.escape.prevent="cancelNewList"
        @blur="cancelNewList"
      />

      <div
        v-for="list in lists"
        :key="list"
        :class="['nav-item', { 'is-active': activeList === list }]"
        @click="$emit('select-list', list)"
      >
        <span class="w-2 h-2 rounded-full shrink-0" :class="activeList === list ? 'bg-white' : 'bg-gray-400'"></span>
        <span class="flex-1 truncate">{{ list }}</span>
        <span v-if="listCount(list)" class="text-[10px]" :class="activeList === list ? 'text-white/70' : 'text-gray-400'">{{ listCount(list) }}</span>
      </div>
    </div>

    <div class="sidebar-divider"></div>

    <!-- Shared Lists -->
    <div class="px-3 pt-3 pb-3 flex-1 overflow-y-auto">
      <div class="px-3 mb-1">
        <span class="section-label">Liste partajate</span>
      </div>
      <div
        v-for="list in sharedLists"
        :key="list"
        :class="['nav-item', { 'is-active': activeList === list }]"
        @click="$emit('select-list', list)"
      >
        <svg class="w-3.5 h-3.5 shrink-0" :class="activeList === list ? 'text-white' : 'text-gray-400'" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/>
        </svg>
        <span class="flex-1 truncate">{{ list }}</span>
      </div>
    </div>
  </aside>
</template>

<script>
export default {
  name: 'LeftSidebar',
  emits: ['select-list', 'add-list'],
  props: {
    activeList:  { type: String, default: 'Azi' },
    lists:       { type: Array,  default: () => [] },
    sharedLists: { type: Array,  default: () => [] },
    tasks:       { type: Array,  default: () => [] },
  },
  data() {
    return { showNewList: false, newListName: '' }
  },
  computed: {
    todayCount() {
      const today = new Date().toISOString().slice(0, 10)
      return this.tasks.filter(t => !t.done && (t.deadline === today || t.deadline < today)).length
    },
  },
  methods: {
    listCount(name) {
      return this.tasks.filter(t => t.list === name && !t.done).length || ''
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
