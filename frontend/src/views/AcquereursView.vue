<script setup lang="ts">
import { computed } from 'vue'
import { useEraStore } from '../stores/era'
import EntityCard from '../components/EntityCard.vue'
import { scoreMatch } from '../composables/useMatching'

const store = useEraStore()
const chauds = computed(() => store.acquereurs.filter((a) => ['qualifie', 'offre'].includes(a.statut)))
const actifs = computed(() => store.acquereurs.filter((a) => !['qualifie', 'offre'].includes(a.statut)))
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">👥 Acquéreurs</div>
        <div class="ssub">Qualification et matching avec vos mandats</div>
      </div>
      <button class="btn btn-gold">+ Ajouter</button>
    </div>
    <div class="stats">
      <div class="sc"><div class="sv">{{ store.acquereurs.length }}</div><div class="sl">Total</div></div>
      <div class="sc"><div class="sv">{{ chauds.length }}</div><div class="sl">Chauds</div></div>
      <div class="sc"><div class="sv">{{ store.mandats.length }}</div><div class="sl">Mandats</div></div>
      <div class="sc"><div class="sv">{{ store.rdv.length }}</div><div class="sl">RDV</div></div>
    </div>
    <div class="srch"><input placeholder="Rechercher un acquéreur..." /></div>
    <div class="sh"><div class="stitle" style="font-size: 16px">🔥 Chauds</div></div>
    <div class="grid">
      <EntityCard
        v-for="item in chauds"
        :key="item.id"
        :title="item.nom"
        :subtitle="item.secteur"
      >
        <template #badge><span class="badge b-red">{{ item.statut }}</span></template>
        <template #rows>
          <div class="card-row"><span class="card-lbl">Budget</span><span class="card-val">{{ item.budget || '—' }}</span></div>
          <div class="card-row"><span class="card-lbl">Type</span><span class="card-val">{{ item.type || '—' }}</span></div>
          <div class="card-row"><span class="card-lbl">Matching</span><span class="card-val">{{ store.mandats.filter((m) => scoreMatch(item, m) > 35).length }}</span></div>
        </template>
      </EntityCard>
    </div>
    <div class="sh" style="margin-top: 16px"><div class="stitle" style="font-size: 16px">🟡 Actifs</div></div>
    <div class="grid">
      <EntityCard
        v-for="item in actifs"
        :key="item.id"
        :title="item.nom"
        :subtitle="item.secteur"
      >
        <template #badge><span class="badge b-orange">{{ item.statut }}</span></template>
      </EntityCard>
    </div>
  </section>
</template>
