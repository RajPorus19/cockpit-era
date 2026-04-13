<script setup lang="ts">
import { computed } from 'vue'
import { useEraStore } from '../stores/era'
import EntityCard from '../components/EntityCard.vue'

const store = useEraStore()
const grouped = computed(() =>
  store.secteurs.map((secteur) => ({
    secteur,
    copros: store.copros.filter((c) => c.secteurId === secteur.id),
  })),
)
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">🗺️ Terrain</div>
        <div class="ssub">Secteurs et copropriétés</div>
      </div>
    </div>
    <div class="grid">
      <EntityCard
        v-for="item in grouped"
        :key="item.secteur.id"
        :title="item.secteur.nom"
        :subtitle="item.secteur.ville"
      >
        <template #badge><span class="badge b-purple">{{ item.copros.length }} copros</span></template>
        <template #rows>
          <div class="card-row" v-for="copro in item.copros.slice(0, 3)" :key="copro.id">
            <span class="card-lbl">📍</span><span class="card-val">{{ copro.adresse }}</span>
          </div>
        </template>
      </EntityCard>
    </div>
  </section>
</template>
