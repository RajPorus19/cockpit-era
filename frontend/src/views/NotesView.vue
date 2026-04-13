<script setup lang="ts">
import { ref } from 'vue'
import { useEraStore } from '../stores/era'

const store = useEraStore()
const quickNote = ref('')

function submit() {
  if (!quickNote.value.trim()) return
  store.addNote(quickNote.value.trim())
  store.toast('Note enregistree')
  quickNote.value = ''
}
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">📝 Notes</div>
        <div class="ssub">Suivi terrain et rappels</div>
      </div>
    </div>
    <div style="display: flex; gap: 8px; margin-bottom: 12px">
      <input v-model="quickNote" class="fsel" placeholder="Ajouter une note..." />
      <button class="btn btn-gold" @click="submit">Ajouter</button>
    </div>
    <div class="notes-grid">
      <article v-for="item in store.notes" :key="item.id" class="note-card">
        <div class="note-ttl">{{ item.titre }}</div>
        <div class="note-body">{{ item.contenu }}</div>
        <div class="note-ft">
          <span class="badge b-blue">{{ item.type }}</span>
          <span class="note-dt">{{ new Date(item.date).toLocaleDateString('fr-FR') }}</span>
        </div>
      </article>
    </div>
  </section>
</template>
