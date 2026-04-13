<script setup lang="ts">
import { computed, ref } from 'vue'
import { useEraStore } from '../stores/era'
import { useVoice } from '../composables/useVoice'

const store = useEraStore()
const { listening, transcript, start, stop, askAssistant } = useVoice()
const prompt = ref('')
const answer = ref('')
const openRdv = computed(() => store.rdv.filter((r) => !r.done).length)
const actifs = computed(() => store.mandats.filter((m) => m.statut !== 'estimation').length)
const caNet = computed(() =>
  store.mandats
    .filter((m) => m.statut !== 'estimation')
    .reduce((sum, m) => sum + Number.parseInt(m.prix || '0', 10), 0),
)
const chauds = computed(() => store.acquereurs.filter((a) => ['qualifie', 'offre'].includes(a.statut)).length)

async function sendPrompt() {
  if (!prompt.value.trim()) return
  answer.value = await askAssistant(prompt.value.trim())
}
</script>

<template>
  <section class="panel on">
    <div style="padding: 4px 0 10px">
      <div class="ssub">{{ new Date().toLocaleDateString('fr-FR', { weekday: 'long', day: 'numeric', month: 'long' }) }}</div>
      <div class="stitle">Aujourd'hui</div>
    </div>
    <div class="kpi-row">
      <div class="kpi-cell">
        <div class="kpi-val" style="color: #d4a017">{{ actifs }}</div>
        <div class="kpi-lbl">Mandats</div>
      </div>
      <div class="kpi-cell">
        <div class="kpi-val" style="font-size: 12px; color: #2ecc8a">{{ caNet.toLocaleString('fr-FR') }} €</div>
        <div class="kpi-lbl">CA net</div>
      </div>
      <div class="kpi-cell">
        <div class="kpi-val" style="color: #ff453a">{{ chauds }}</div>
        <div class="kpi-lbl">Chauds</div>
      </div>
      <div class="kpi-cell">
        <div class="kpi-val" style="font-size: 12px; color: #0a84ff">{{ openRdv }}</div>
        <div class="kpi-lbl">Prochain RDV</div>
      </div>
    </div>
    <div class="card">
      <div class="card-top">
        <div>
          <div class="card-ttl">Assistant vocal / IA</div>
          <div class="card-sub">{{ listening ? transcript : 'Inactif' }}</div>
        </div>
        <span class="badge b-blue">Assistant</span>
      </div>
      <div class="card-row" v-for="item in store.todayTasks" :key="item">
        <span class="card-lbl">A faire</span>
        <span class="card-val">{{ item }}</span>
      </div>
      <div style="display: flex; gap: 8px; margin-top: 10px">
        <button class="btn btn-ghost" @click="start">Demarrer</button>
        <button class="btn btn-ghost" @click="stop">Stop</button>
      </div>
      <div style="display: flex; gap: 8px; margin-top: 8px">
        <input v-model="prompt" class="fsel" placeholder="Question assistant..." />
        <button class="btn btn-gold" @click="sendPrompt">Envoyer</button>
      </div>
      <p class="muted" style="margin-top: 8px">{{ answer }}</p>
    </div>
  </section>
</template>
