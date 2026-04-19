<script setup lang="ts">
import { computed, ref } from 'vue'
import { scoreMatch } from '../composables/useMatching'
import { useEraStore } from '../stores/era'
import type { Mandat } from '../types/domain'

const store = useEraStore()
const query = ref('')
const pipeFilter = ref('')
const steps = ['estimation', 'signature', 'actif', 'visite', 'offre', 'compromis', 'acte'] as const
const statutLabels: Record<string, string> = {
  estimation: 'Estimation',
  signature: 'En signature',
  actif: 'Actif',
  visite: 'Visites',
  offre: 'Offre',
  compromis: 'Compromis',
  acte: 'Acte signé',
  expire: 'Expiré',
}
const statutBadgeClass: Record<string, string> = {
  estimation: 'b-orange',
  signature: 'b-purple',
  actif: 'b-green',
  visite: 'b-green',
  offre: 'b-blue',
  compromis: 'b-blue',
  acte: 'b-green',
  expire: 'b-red',
  retire: 'b-gray',
}

function fmtP(v?: string | number) {
  const n = Number.parseInt(String(v || 0), 10)
  if (!n) return '—'
  return `${n.toLocaleString('fr-FR')}€`
}

function fmtD(v?: string) {
  if (!v) return '—'
  const dt = new Date(v)
  if (Number.isNaN(dt.getTime())) return v
  return dt.toLocaleDateString('fr-FR', { day: 'numeric', month: 'short', year: 'numeric' })
}

function calcNet(m: Mandat) {
  const prix = Number.parseInt(m.prix || '0', 10)
  if (!prix) return null
  const com = m.fai ? Number.parseInt(m.fai, 10) - prix : Math.round(prix * ((m.taux || 7) / 100))
  if (com <= 0) return null
  const apresMA = m.source === 'ma' ? com * 0.75 : com
  const brut = apresMA * 0.32
  return { com: Math.round(com), net: Math.round(brut * 0.78) }
}

function matchPipe(m: Mandat) {
  if (!pipeFilter.value) return true
  if (pipeFilter.value === 'compromis') return ['compromis', 'acte'].includes(m.statut)
  return m.statut === pipeFilter.value
}

function matchQuery(m: Mandat) {
  const q = query.value.trim().toLowerCase()
  if (!q) return true
  return m.adresse.toLowerCase().includes(q) || (m.vendeur || '').toLowerCase().includes(q)
}

const actifs = computed(() => {
  const activeStatuses = ['actif', 'visite', 'offre', 'compromis', 'acte', 'signature']
  return store.mandats.filter((m) => activeStatuses.includes(m.statut) && matchPipe(m) && matchQuery(m))
})

const estimations = computed(() => {
  if (pipeFilter.value && pipeFilter.value !== 'estimation') return []
  return store.mandats.filter((m) => m.statut === 'estimation' && matchQuery(m))
})

const allCount = computed(() => store.mandats.length)
const estimationCount = computed(() => store.mandats.filter((m) => m.statut === 'estimation').length)
const signatureCount = computed(() => store.mandats.filter((m) => m.statut === 'signature').length)
const actifCount = computed(() => store.mandats.filter((m) => ['actif', 'visite', 'offre'].includes(m.statut)).length)
const compromisCount = computed(() => store.mandats.filter((m) => ['compromis', 'acte'].includes(m.statut)).length)
const caActif = computed(() => actifs.value.reduce((sum, m) => sum + (calcNet(m)?.net || 0), 0))
const caEstim = computed(() => estimations.value.reduce((sum, m) => sum + (calcNet(m)?.net || 0), 0))

function daysToExpire(m: Mandat) {
  if (!m.dateExp || ['acte', 'expire'].includes(m.statut)) return null
  return Math.ceil((new Date(m.dateExp).getTime() - Date.now()) / 86400000)
}

function mandateType(m: Mandat) {
  const mTypes: Record<string, string> = { exclusif: '⭐ Exclusif', simple: '📄 Simple', 'co-exclusif': '👥 Co-exclusif' }
  return mTypes[m.mandat || ''] || m.mandat || '—'
}

function timelineDotClass(m: Mandat, index: number) {
  const cur = steps.indexOf((m.statut as (typeof steps)[number]) || 'estimation')
  if (m.statut === 'expire' && index === 0) return 'var(--red)'
  if (index === cur) return 'var(--gold)'
  if (index < cur) return 'rgba(52,199,89,.5)'
  return 'var(--surface2)'
}

function passerEnActif(id: string) {
  store.setMandatStatut(id, 'actif')
  store.toast("Mandat passe en actif, pensez a renseigner l'expiration")
}

function waLink(tel?: string) {
  if (!tel) return '#'
  return `https://wa.me/${tel.replace(/\s/g, '').replace(/^0/, '+33')}`
}

function acqMatches(m: Mandat) {
  return store.acquereurs
    .filter((a) => !['compromis', 'inactif'].includes(a.statut))
    .map((a) => ({ a, score: scoreMatch(a, m) }))
    .filter((x) => x.score >= 30)
    .sort((a, b) => b.score - a.score)
}
</script>

<template>
  <section class="panel on">
    <div class="sh">
      <div>
        <div class="stitle">📋 Mandats</div>
        <div class="ssub">Pipeline et calcul de commission automatique</div>
      </div>
      <button class="btn btn-gold">+ Nouveau mandat</button>
    </div>
    <div class="pipeline">
      <button class="pipe-step" :class="{ 'active-filter': pipeFilter === '' }" @click="pipeFilter = ''"><div class="pipe-num">{{ allCount }}</div><div class="pipe-dot" style="background: var(--text3)" /><div class="pipe-lbl">Tous</div></button>
      <button class="pipe-step" :class="{ 'active-filter': pipeFilter === 'estimation' }" @click="pipeFilter = 'estimation'"><div class="pipe-num">{{ estimationCount }}</div><div class="pipe-dot" style="background: var(--orange)" /><div class="pipe-lbl">Estimation</div></button>
      <button class="pipe-step" :class="{ 'active-filter': pipeFilter === 'signature' }" @click="pipeFilter = 'signature'"><div class="pipe-num">{{ signatureCount }}</div><div class="pipe-dot" style="background: var(--purple)" /><div class="pipe-lbl">Signature</div></button>
      <button class="pipe-step" :class="{ 'active-filter': pipeFilter === 'actif' }" @click="pipeFilter = 'actif'"><div class="pipe-num">{{ actifCount }}</div><div class="pipe-dot" style="background: var(--green)" /><div class="pipe-lbl">Actif</div></button>
      <button class="pipe-step" :class="{ 'active-filter': pipeFilter === 'compromis' }" @click="pipeFilter = 'compromis'"><div class="pipe-num">{{ compromisCount }}</div><div class="pipe-dot" style="background: var(--blue)" /><div class="pipe-lbl">Compromis</div></button>
    </div>
    <div class="srch"><input v-model="query" placeholder="Rechercher un mandat, un vendeur..." /></div>

    <div class="m-section-header">
      <div class="m-section-dot" />
      <span class="m-section-title">Mandats actifs</span>
      <span class="m-section-count">{{ actifs.length }}</span>
      <span class="ssub" style="margin-left:8px">{{ caActif ? `CA net ≈ ${caActif.toLocaleString('fr-FR')} €` : '' }}</span>
    </div>
    <div class="grid">
      <article v-for="item in actifs" :key="item.id" class="card">
        <div class="card-top">
          <div style="flex:1">
            <div class="card-ttl">{{ item.adresse }}</div>
            <div v-if="item.vendeur" class="card-sub">👤 {{ item.vendeur }}</div>
          </div>
          <span class="tag">{{ mandateType(item) }}</span>
        </div>
        <div v-if="item.ref" class="card-row"><span class="card-lbl">Réf</span><span class="card-val" style="font-size:11px;color:var(--text3)">{{ item.ref }}</span></div>
        <template v-if="item.prix">
          <div class="card-row"><span class="card-lbl">🏷️ Prix FAI</span><span class="card-val" style="color:var(--text);font-weight:700">{{ fmtP(item.fai || Math.round(Number(item.prix) * (1 + (item.taux || 7) / 100))) }}</span></div>
          <div class="card-row"><span class="card-lbl">🤝 Net vendeur</span><span class="card-val" style="color:var(--green);font-weight:700">{{ fmtP(item.prix) }}</span></div>
        </template>
        <div v-if="calcNet(item)" class="net-row">
          <div>
            <div class="net-lbl">💵 Net Samir</div>
            <div class="net-detail">COM {{ fmtP(calcNet(item)?.com) }} · taux {{ item.taux || 7 }}%{{ item.source === 'ma' ? ' · via MA' : '' }}</div>
          </div>
          <div class="net-val">{{ fmtP(calcNet(item)?.net) }}</div>
        </div>
        <div v-if="item.surface" class="card-row"><span class="card-lbl">📐 Surface</span><span class="card-val">{{ item.surface }} m²</span></div>
        <div v-if="item.tel" class="card-row"><span class="card-lbl">📞</span><span class="card-val"><a :href="`tel:${item.tel}`" style="color:var(--text)">{{ item.tel }}</a></span></div>
        <div v-if="item.dateExp" class="card-row"><span class="card-lbl">⏳ Expire</span><span class="card-val">{{ fmtD(item.dateExp) }}</span></div>
        <div v-if="item.notes" style="font-size:11px;color:var(--text3);margin-top:8px;line-height:1.5">{{ item.notes.slice(0, 100) }}{{ item.notes.length > 100 ? '…' : '' }}</div>
        <div style="display:flex;align-items:center;gap:5px;margin-top:10px;padding-top:8px;border-top:0.5px solid var(--border)">
          <div v-for="(step, idx) in steps" :key="step" :style="{ width: idx === steps.indexOf(item.statut as never) ? '8px' : '6px', height: idx === steps.indexOf(item.statut as never) ? '8px' : '6px', borderRadius: '50%', background: timelineDotClass(item, idx), flexShrink: '0' }" />
          <span style="font-size:9px;color:var(--text3);margin-left:auto">{{ statutLabels[item.statut] || item.statut }}</span>
        </div>
        <div class="card-ft">
          <div style="display:flex;gap:5px;flex-wrap:wrap">
            <span class="badge" :class="statutBadgeClass[item.statut] || 'b-gray'">{{ statutLabels[item.statut] || item.statut }}</span>
            <span v-if="daysToExpire(item) !== null && (daysToExpire(item) || 0) < 0" class="badge b-red">Expiré</span>
            <span v-else-if="daysToExpire(item) !== null && (daysToExpire(item) || 0) <= 30" class="badge b-orange">⏰ {{ daysToExpire(item) }}j</span>
          </div>
          <div class="acq-act-row">
            <a v-if="item.tel" :href="`tel:${item.tel.replace(/\s/g, '')}`" class="acq-ico ai-call">📞</a>
            <button v-else class="acq-ico ai-call" style="opacity:.3;cursor:default">📞</button>
            <a v-if="item.tel" :href="waLink(item.tel)" target="_blank" class="acq-ico ai-wa">💬</a>
            <button v-else class="acq-ico ai-wa" style="opacity:.3;cursor:default">💬</button>
            <button class="acq-ico ai-note" @click="store.toast('Feuille note a brancher')">📝</button>
            <button class="acq-ico ai-match" @click="store.toast('Matching a brancher')">👥</button>
            <button class="ai-more" @click="store.toast('Menu mandat a brancher')">⋯</button>
          </div>
        </div>
      </article>
    </div>

    <div class="m-section-header" style="margin-top:16px">
      <div class="m-section-dot m-section-dot-estim" />
      <span class="m-section-title">Estimations en cours</span>
      <span class="m-section-count m-section-count-estim">{{ estimations.length }}</span>
      <span class="ssub" style="margin-left:8px">{{ caEstim ? `Potentiel ≈ ${caEstim.toLocaleString('fr-FR')} €` : '' }}</span>
    </div>
    <div class="grid">
      <article v-for="item in estimations" :key="item.id" class="card card-estim">
        <div class="card-top">
          <div style="flex:1">
            <div class="card-ttl">{{ item.adresse }}</div>
            <div v-if="item.vendeur" class="card-sub">👤 {{ item.vendeur }}</div>
          </div>
          <span class="tag" style="background:var(--orange-bg);color:var(--orange)">🟡 Estimation</span>
        </div>
        <div v-if="item.ref" class="card-row"><span class="card-lbl">Réf</span><span class="card-val" style="font-size:11px;color:var(--text3)">{{ item.ref }}</span></div>
        <template v-if="item.prix">
          <div class="card-row"><span class="card-lbl">🏷️ Prix estim. FAI</span><span class="card-val" style="color:var(--text);font-weight:700">{{ fmtP(item.fai || Math.round(Number(item.prix) * (1 + (item.taux || 7) / 100))) }}</span></div>
          <div class="card-row"><span class="card-lbl">🤝 Net vendeur</span><span class="card-val">{{ fmtP(item.prix) }}</span></div>
        </template>
        <div v-if="calcNet(item)" class="net-row" style="opacity:.8">
          <div>
            <div class="net-lbl">💵 Net Samir estimé</div>
            <div class="net-detail">si mandat signé · taux {{ item.taux || 7 }}%{{ item.source === 'ma' ? ' · via MA' : '' }}</div>
          </div>
          <div class="net-val">{{ fmtP(calcNet(item)?.net) }}</div>
        </div>
        <div v-if="item.surface" class="card-row"><span class="card-lbl">📐 Surface</span><span class="card-val">{{ item.surface }} m²</span></div>
        <div v-if="item.tel" class="card-row"><span class="card-lbl">📞</span><span class="card-val"><a :href="`tel:${item.tel}`" style="color:var(--text)">{{ item.tel }}</a></span></div>
        <div v-if="acqMatches(item).length" style="display:flex;gap:5px;flex-wrap:wrap;align-items:center;margin-top:8px">
          <span style="font-size:9px;color:var(--text3);font-weight:600">Acq. potentiels :</span>
          <span v-for="entry in acqMatches(item).slice(0, 3)" :key="entry.a.id" class="badge b-blue">👤 {{ entry.a.nom.split(' ')[0] }}</span>
          <span v-if="acqMatches(item).length > 3" style="font-size:9px;color:var(--text3);padding:2px 6px">+{{ acqMatches(item).length - 3 }}</span>
        </div>
        <div v-if="item.notes" style="font-size:11px;color:var(--text3);margin-top:8px;line-height:1.5">{{ item.notes.slice(0, 100) }}{{ item.notes.length > 100 ? '…' : '' }}</div>
        <div style="display:flex;align-items:center;gap:5px;margin-top:10px;padding-top:8px;border-top:0.5px solid var(--border)">
          <div v-for="(step, idx) in steps" :key="step" :style="{ width: idx === steps.indexOf(item.statut as never) ? '8px' : '6px', height: idx === steps.indexOf(item.statut as never) ? '8px' : '6px', borderRadius: '50%', background: timelineDotClass(item, idx), flexShrink: '0' }" />
          <span style="font-size:9px;color:var(--text3);margin-left:auto">{{ statutLabels[item.statut] || item.statut }}</span>
        </div>
        <div class="card-ft">
          <button class="estim-btn-actif" @click="passerEnActif(item.id)">▶ Passer en mandat actif</button>
          <div class="acq-act-row">
            <a v-if="item.tel" :href="`tel:${item.tel.replace(/\s/g, '')}`" class="acq-ico ai-call">📞</a>
            <button v-else class="acq-ico ai-call" style="opacity:.3;cursor:default">📞</button>
            <a v-if="item.tel" :href="waLink(item.tel)" target="_blank" class="acq-ico ai-wa">💬</a>
            <button v-else class="acq-ico ai-wa" style="opacity:.3;cursor:default">💬</button>
            <button class="acq-ico ai-note" @click="store.toast('Feuille note a brancher')">📝</button>
            <button class="acq-ico" style="background:var(--green-bg);color:var(--green)" @click="passerEnActif(item.id)">▶</button>
            <button class="ai-more" @click="store.toast('Menu mandat a brancher')">⋯</button>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>
