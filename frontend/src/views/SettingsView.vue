<script setup lang="ts">
import { ref } from 'vue'
import { useEraStore } from '../stores/era'

const store = useEraStore()
const theme = ref(localStorage.getItem('theme') || 'dark')

function applyTheme() {
  document.documentElement.dataset.theme = theme.value
  localStorage.setItem('theme', theme.value)
  store.toast('Theme mis a jour')
}

function resetData() {
  ;['mandats', 'acq', 'rdv_clients', 'notes', 't_secteurs', 't_copros'].forEach((k) => localStorage.removeItem(k))
  location.reload()
}
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">⚙️ Réglages</div>
        <div class="ssub">Personnalisation et sécurité</div>
      </div>
    </div>
    <div class="card">
      <div class="card-top">
        <div>
          <div class="card-ttl">Préférences</div>
          <div class="card-sub">Paramètres visuels et données locales</div>
        </div>
      </div>
      <div class="card-row">
        <span class="card-lbl">Thème</span>
        <select v-model="theme" class="fsel" @change="applyTheme">
          <option value="dark">Sombre</option>
          <option value="light">Clair</option>
        </select>
      </div>
      <div class="card-ft">
        <span class="muted">Les données sont stockées localement</span>
        <div class="card-actions">
          <button class="btn btn-ghost" @click="resetData">Réinitialiser</button>
        </div>
      </div>
    </div>
  </section>
</template>
