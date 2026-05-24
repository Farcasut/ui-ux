<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-6"
    style="background: rgba(0,0,0,0.45); backdrop-filter: blur(2px);"
    @click.self="$emit('close')"
  >
    <div
      class="w-full max-w-md rounded-2xl shadow-2xl border border-black/10 overflow-hidden"
      style="background: var(--color-surface);"
    >
      <!-- Header -->
      <div class="px-6 pt-6 pb-4 border-b border-black/10">
        <h2 class="text-xl font-black">Adauga / editeaza task</h2>
        <p class="text-xs text-gray-500 mt-1">Flux rapid, cu optiuni clare pentru prioritate si reminder.</p>
      </div>

      <!-- Fields -->
      <div class="px-6 py-5 flex flex-col gap-4">

        <!-- Titlu -->
        <div>
          <label class="modal-label">Titlu task</label>
          <input
            ref="titleInput"
            v-model="form.title"
            type="text"
            placeholder="Ex: Trimite raport UX"
            class="modal-input"
            @keydown.enter.prevent="save"
            @keydown.escape.prevent="$emit('close')"
          />
        </div>

        <!-- Lista + Prioritate -->
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="modal-label">Lista / notebook</label>
            <select v-model="form.list" class="modal-input cursor-pointer">
              <option v-for="l in lists" :key="l" :value="l">{{ l }}</option>
            </select>
          </div>
          <div>
            <label class="modal-label">Prioritate</label>
            <select v-model="form.priority" class="modal-input cursor-pointer">
              <option value="P1">P1 / Urgent</option>
              <option value="P2">P2 / Normal</option>
              <option value="P3">P3 / Low</option>
            </select>
          </div>
        </div>

        <!-- Deadline + Reminder -->
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="modal-label">Deadline</label>
            <div class="flex gap-1.5">
              <input v-model="form.deadline" type="date" class="modal-input flex-1 min-w-0" />
              <input v-model="form.time" type="time" class="modal-input w-24 shrink-0" placeholder="ora" />
            </div>
          </div>
          <div>
            <label class="modal-label">Reminder</label>
            <select v-model="form.reminder" class="modal-input cursor-pointer">
              <option value="">La deadline / custom</option>
              <option value="La deadline">La deadline</option>
              <option value="Cu 30 minute inainte">Cu 30 min inainte</option>
              <option value="Cu 1 ora inainte">Cu 1 ora inainte</option>
              <option value="Cu 2 ore inainte">Cu 2 ore inainte</option>
              <option value="Cu o zi inainte">Cu o zi inainte</option>
            </select>
          </div>
        </div>

        <!-- Responsabil — only for shared lists -->
        <div v-if="isShared">
          <label class="modal-label">Responsabil</label>
          <select v-model="form.assignee" class="modal-input cursor-pointer">
            <option value="">Neatribuit</option>
            <option v-for="member in listMembers" :key="member" :value="member">{{ member }}</option>
          </select>
        </div>

        <!-- Note -->
        <div>
          <label class="modal-label">Note optionale</label>
          <textarea
            v-model="form.notes"
            rows="2"
            placeholder="Detalii scurte, instructiuni sau context..."
            class="modal-input resize-none"
          ></textarea>
        </div>
      </div>

      <!-- Actions -->
      <div class="px-6 pb-6 flex items-center justify-between gap-3">
        <button
          class="px-6 py-2.5 rounded-xl border border-black text-sm font-semibold bg-transparent cursor-pointer hover:bg-black/5 transition-colors"
          @click="$emit('close')"
        >Cancel</button>
        <button
          :class="['px-8 py-2.5 rounded-xl text-sm font-semibold border-0 transition-colors', form.title.trim() ? 'bg-black text-white cursor-pointer hover:bg-gray-800' : 'bg-gray-300 text-gray-500 cursor-not-allowed']"
          :disabled="!form.title.trim()"
          @click="save"
        >Salveaza</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TaskModal',
  emits: ['save', 'close'],
  props: {
    task:            { type: Object, default: null },
    lists:           { type: Array,  default: () => [] },
    sharedListNames: { type: Array,  default: () => [] },
    shareData:       { type: Object, default: () => ({}) },
  },
  data() {
    const today = new Date().toISOString().slice(0, 10)
    return {
      form: this.task
        ? { title: '', time: '', reminder: '', assignee: '', notes: '', tags: [], ...this.task }
        : {
            title: '', list: this.lists[0] || '', priority: 'P2',
            deadline: today, time: '', reminder: '',
            assignee: '', notes: '', tags: [],
          },
    }
  },
  mounted() {
    this.$nextTick(() => this.$refs.titleInput?.focus())
  },
  computed: {
    isShared() {
      return this.sharedListNames.includes(this.form.list)
    },
    listMembers() {
      const collaborators = (this.shareData || {})[this.form.list] || []
      return collaborators.map(c => {
        const name = c.email.split('@')[0]
        return name.charAt(0).toUpperCase() + name.slice(1)
      })
    },
  },
  methods: {
    save() {
      if (!this.form.title.trim()) return
      this.$emit('save', { ...this.form })
    },
  },
}
</script>
