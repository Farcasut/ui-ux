<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-6"
    style="background: rgba(0,0,0,0.45); backdrop-filter: blur(2px);"
    @click.self="$emit('close')"
  >
    <div
      class="w-full max-w-2xl rounded-2xl shadow-2xl border border-black/10 overflow-hidden"
      style="background: var(--color-surface);"
    >
      <!-- Header -->
      <div class="px-6 pt-6 pb-4 flex items-start justify-between border-b border-black/10">
        <div>
          <h2 class="text-xl font-black">Doer / Partajare lista</h2>
          <p class="text-xs text-gray-500 mt-1">Control al accesului pentru owner, colaboratori read-only si colaboratori write.</p>
        </div>
        <button
          class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-black/10 cursor-pointer border-0 bg-transparent text-gray-500 hover:text-black transition-colors text-xl leading-none shrink-0 ml-4"
          @click="$emit('close')"
        >×</button>
      </div>

      <div class="px-6 py-5 flex flex-col gap-4 overflow-y-auto" style="max-height: calc(100vh - 200px);">

        <!-- Invita colaborator -->
        <div class="rounded-xl border border-black/15 p-4" style="background: var(--color-app-bg);">
          <p class="text-sm font-bold mb-3">Invita colaborator</p>
          <div class="flex gap-2 items-center">
            <input
              v-model="inviteEmail"
              type="email"
              placeholder="email@exemplu.ro"
              class="modal-input flex-1 min-w-0"
              @keydown.enter.prevent="sendInvite"
            />
            <select v-model="invitePermission" class="modal-input shrink-0 cursor-pointer" style="width: 148px;">
              <option value="read-only">Permisiune: read-only</option>
              <option value="write">Permisiune: write</option>
              <option value="write-and-share">Permisiune: write+share</option>
            </select>
            <button
              :class="['shrink-0 px-4 py-2.5 rounded-xl text-sm font-semibold border-0 transition-colors', canInvite ? 'bg-black text-white cursor-pointer hover:bg-gray-800' : 'bg-gray-300 text-gray-500 cursor-not-allowed']"
              :disabled="!canInvite"
              @click="sendInvite"
            >Trimite invitatie</button>
          </div>
          <p v-if="inviteError" class="text-xs text-red-500 mt-2">{{ inviteError }}</p>
          <p v-if="inviteSuccess" class="text-xs text-green-600 mt-2">{{ inviteSuccess }}</p>
        </div>

        <!-- Colaboratori existenti -->
        <div class="rounded-xl border border-black/15 overflow-hidden" style="background: var(--color-app-bg);">
          <p class="text-sm font-bold px-4 pt-4 pb-3">Colaboratori existenti</p>

          <table class="w-full text-sm" style="border-collapse: collapse;">
            <thead>
              <tr style="border-top: 1px solid rgba(0,0,0,0.08); border-bottom: 1px solid rgba(0,0,0,0.08);">
                <th class="text-left text-xs font-bold text-gray-500 px-4 py-2.5 uppercase tracking-wide">Utilizator</th>
                <th class="text-left text-xs font-bold text-gray-500 px-3 py-2.5 uppercase tracking-wide">Rol</th>
                <th class="text-left text-xs font-bold text-gray-500 px-3 py-2.5 uppercase tracking-wide">Permisiune</th>
                <th class="text-left text-xs font-bold text-gray-500 px-4 py-2.5 uppercase tracking-wide">Actiuni</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(c, i) in localCollaborators"
                :key="c.email"
                :style="i < localCollaborators.length - 1 ? 'border-bottom: 1px solid rgba(0,0,0,0.06)' : ''"
              >
                <td class="px-4 py-3 text-sm font-medium">{{ c.email }}</td>
                <td class="px-3 py-3">
                  <span
                    class="text-xs px-2 py-1 rounded-lg font-semibold"
                    :class="c.role === 'Owner' ? 'bg-black text-white' : 'bg-gray-200 text-gray-700'"
                  >{{ c.role }}</span>
                </td>
                <td class="px-3 py-3">
                  <select
                    v-model="c.permission"
                    :disabled="c.role === 'Owner'"
                    class="text-xs px-2 py-1.5 rounded-lg border border-black/20 outline-none cursor-pointer"
                    :class="c.role === 'Owner' ? 'opacity-50 cursor-not-allowed' : ''"
                    style="background: var(--color-surface);"
                    @change="emitUpdate"
                  >
                    <option value="read-only">read-only</option>
                    <option value="write">write</option>
                    <option value="write-and-share">write-and-share</option>
                  </select>
                </td>
                <td class="px-4 py-3">
                  <span v-if="c.role === 'Owner'" class="text-xs text-gray-400">—</span>
                  <div v-else class="flex items-center gap-2 text-xs">
                    <button
                      class="text-gray-500 hover:text-black underline cursor-pointer border-0 bg-transparent p-0 transition-colors"
                      @click="savePermission(c)"
                    >schimba</button>
                    <span class="text-gray-300">/</span>
                    <button
                      class="text-red-500 hover:text-red-700 underline cursor-pointer border-0 bg-transparent p-0 transition-colors"
                      @click="removeCollaborator(c.email)"
                    >elimina</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>

          <p v-if="localCollaborators.length === 0" class="text-sm text-gray-400 px-4 py-4 text-center">Niciun colaborator inca.</p>
        </div>

        <!-- Info note -->
        <div class="flex items-start gap-3 px-4 py-3 rounded-xl border border-black/10" style="background: var(--color-app-bg);">
          <svg class="w-4 h-4 text-gray-500 shrink-0 mt-0.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <circle cx="12" cy="12" r="10"/><path d="M12 16v-4M12 8h.01"/>
          </svg>
          <p class="text-xs text-gray-600 leading-relaxed">
            <strong>Regula:</strong> userii <em>read-only</em> pot vedea lista, dar nu pot modifica task-uri sau permisiuni.
            Userii <em>write</em> pot adauga si edita task-uri. Userii <em>write-and-share</em> pot si invita alti colaboratori.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ShareModal',
  emits: ['update-collaborators', 'close'],
  props: {
    listName:      { type: String, default: '' },
    collaborators: { type: Array,  default: () => [] },
  },
  data() {
    return {
      inviteEmail:      '',
      invitePermission: 'read-only',
      inviteError:      '',
      inviteSuccess:    '',
      localCollaborators: this.collaborators.map(c => ({ ...c })),
    }
  },
  computed: {
    canInvite() {
      return this.inviteEmail.trim().includes('@')
    },
  },
  methods: {
    sendInvite() {
      this.inviteError = ''
      this.inviteSuccess = ''
      const email = this.inviteEmail.trim().toLowerCase()
      if (!email.includes('@')) {
        this.inviteError = 'Introdu o adresa de email valida.'
        return
      }
      if (this.localCollaborators.some(c => c.email.toLowerCase() === email)) {
        this.inviteError = 'Acest utilizator este deja colaborator.'
        return
      }
      this.localCollaborators.push({ email, role: 'Colaborator', permission: this.invitePermission })
      this.inviteSuccess = `Invitatie trimisa catre ${email}.`
      this.inviteEmail = ''
      this.invitePermission = 'read-only'
      this.emitUpdate()
      setTimeout(() => { this.inviteSuccess = '' }, 3000)
    },
    removeCollaborator(email) {
      this.localCollaborators = this.localCollaborators.filter(c => c.email !== email)
      this.emitUpdate()
    },
    savePermission() {
      this.emitUpdate()
    },
    emitUpdate() {
      this.$emit('update-collaborators', this.localCollaborators.map(c => ({ ...c })))
    },
  },
  watch: {
    collaborators(val) {
      this.localCollaborators = val.map(c => ({ ...c }))
    },
  },
}
</script>
