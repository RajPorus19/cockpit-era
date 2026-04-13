<script setup lang="ts">
import { computed } from 'vue'
import { useEraStore } from '../stores/era'
import EntityCard from '../components/EntityCard.vue'

const store = useEraStore()
const openRdv = computed(() => store.rdv.filter((r) => !r.done))
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">📅 RDV</div>
        <div class="ssub">Planning de la semaine</div>
      </div>
      <button class="btn btn-gold">+ Nouveau RDV</button>
    </div>
    <div class="grid">
      <EntityCard
        v-for="item in openRdv"
        :key="item.id"
        :title="item.client"
        :subtitle="item.date"
      >
        <template #badge><span class="badge b-blue">{{ item.objet || 'RDV' }}</span></template>
        <template #rows>
          <div class="card-row"><span class="card-lbl">Heure</span><span class="card-val">{{ item.heure }}</span></div>
          <div class="card-row"><span class="card-lbl">Tel</span><span class="card-val">{{ item.tel || '—' }}</span></div>
        </template>
      </EntityCard>
    </div>
  </section>
</template>
