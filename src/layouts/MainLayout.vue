<template>
  <q-layout view="hHh lpR fFf" style="min-width: 870px">

    <q-header style="box-shadow: 0 -8px 34px rgba(200, 200, 200, 0.8);">
      <q-toolbar style="height: 64px;" class="q-pa-none">
        <q-btn flat @click="drawerLayers = true" icon="menu" size="25px" />
        <q-btn flat @click="drawerOptions = true" icon="settings" size="25px" />
        <q-space/>
        <span class="text-subtitle1">SpaceMesh Lamba Chunks</span>
        <q-space/>
        <q-btn flat @click="Dark.toggle" :icon="$q.dark.isActive ? 'light_mode' : 'dark_mode'" size="25px" />
        <q-btn v-if="table" flat @click="table = false" icon="format_list_numbered_rtl" size="25px" />
        <q-btn v-else flat @click="table = true" icon="snowing" size="25px" />
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="drawerOptions"
      :width="300"
      behavior="mobile"
      bordered
      class="q-ma-sm text-center"
    >
      <q-scroll-area class="full-width full-height">
        <div class="column">
          <q-checkbox v-model="optE24" label="Team24Early" />
          <q-checkbox v-model="opt12H" label="Default12H" />
          <q-checkbox v-model="optT24" label="Team24Late" />
        </div>
      </q-scroll-area>
    </q-drawer>

    <q-drawer
      v-model="drawerLayers"
      :width="710"
      behavior="mobile"
      bordered
      class="q-ma-sm text-center"
    >
    <q-scroll-area class="full-width full-height">

      <div class="row items-center no-wrap">
        <q-input
          v-model="coinbase"
          label="coinbase sm1qqqqqqq... or several with any separator"
          filled dense stack-label
          class="full-width"
        >
          <template v-slot:prepend>
            <q-icon name="account_balance_wallet" size="20px" :style="{color: coinbase ? '#217ad2' : '#636363'}" />
          </template>
          <template v-slot:append>
            <q-icon name="send" class="cursor-pointer" color="blue-8" style="height: 40px" @click="rewardsCheck"/>
          </template>
        </q-input>
        <q-btn flat dense rounded icon="more_vert" color="grey" class="q-ml-sm q-mr-md">
          <q-menu :offset="[0, 0]" anchor="bottom middle" self="top middle">
            <q-list>
              <q-item clickable @click="SMMUpload">
                <q-item-section>
                  <div>
                    <q-icon name="download" size="24px" color="blue" />
                    load SM-Monitor config
                  </div>
                </q-item-section>
                <q-uploader
                  v-show="false"
                  ref="uploaderRef"
                  accept=".json"
                  @added="SMMFileAdded"
                />
              </q-item>
              <q-item clickable @click="SMMDownload">
                <q-item-section class="row no-wrap">
                  <div>
                    <q-icon name="upload" size="24px" color="blue" />
                    save SM-Monitor config
                  </div>
                </q-item-section>
              </q-item>
              <q-item clickable @click="windowOpen('https://github.com/xeliuqa/SM-Monitor')">
                <q-item-section class="row no-wrap">
                  <div>
                    <q-icon name="info" size="24px" color="blue" />
                    SM-Monitor on github
                  </div>
                </q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </div>

      <div class="row items-center no-wrap" v-for="n in nodes" :key="n.serial">
        <q-input
          filled dense stack-label
          v-model="n.node"
          maxlength="10"
          label="Node"
          type="text"
          :style="{width: '190px'}"
        >
          <template v-slot:prepend>
            <q-icon
              :name="n.show ? 'visibility' : 'visibility_off'"
              :style="{color: n.show ? '#217ad2' : '#636363'}"
              size="20px" class="cursor-pointer" @click="n.show = !n.show"
            />
          </template>
          <template v-slot:append>
            <q-icon name="palette" class="cursor-pointer -material-icons-outlined" size="20px"
                    :style="{'color': n.color, 'text-shadow': n.color ? '0 2px 2px rgba(0,0,0,0.6)' : ''}">
              <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                <q-color v-model="n.color" />
              </q-popup-proxy>
            </q-icon>
            <q-icon name="close" v-if="n.color !== ''" flat class="cursor-pointer" @click="n.color = ''" />
          </template>
        </q-input>
        <q-input
          filled dense stack-label
          v-model="n.layers"
          label="Eligible for rewards in layers | proposal eligibility for an epoch"
          type="text"
          class="q-ma-xs"
          style="width: 500px"
        >
          <template v-slot:append>
            <q-icon name="adjust" class="cursor-pointer" v-if="!isLayersOkay(n.layers)">
              <q-menu self="top middle" anchor="bottom middle">
                <q-list>
                  <q-item clickable v-close-popup class="bg-white text-black">
                    <q-item-section @click="nodeAdjust(n.serial)">
                      Adjust
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-menu>
            </q-icon>
            <q-icon v-if="nodes.length > 1" name="delete" color="red-5" class="cursor-pointer">
              <q-menu self="top middle" anchor="bottom middle">
                <q-list>
                  <q-item clickable v-close-popup class="bg-red text-white">
                    <q-item-section @click="nodeDelete(n.serial)">
                      Delete
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-menu>
            </q-icon>
          </template>
        </q-input>

      </div>

      <q-btn label="ADD NODE" flat size="10px" style="height: 40px" @click="nodeCreate" />

    </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <q-page class="items-center justify-center column">

        <template v-if="loading">
          <q-spinner size="100px" color="blue-8" />
        </template>
        <template v-else>

          <q-markup-table flat dense v-if="table" class="q-mt-lg">
            <thead>
              <tr>
                <th>Layer</th>
                <th>Info</th>
                <th>SMH</th>
                <th>DateTime</th>
              </tr>
            </thead>
            <tbody>
            <template v-for="id in eligLayersIdList()" :key="id.layer">
              <template v-if="id.info">
                <tr class="my-info">
                  <td>{{ id.layer }}</td>
                  <td class="text-center">
                    {{ id.info.join(' - ') }}
                    <q-icon v-if="id.link" name="info" color="blue" size="18px" class="cursor-pointer" style="top: -1px" @click="windowOpen(id.link)" />
                  </td>
                  <td />
                  <td>{{ id.time }}</td>
                </tr>
              </template>
              <template v-else-if="id.layer === -1">
                <tr class="my-here">
                  <td>{{ currentLayer }}</td>
                  <td class="text-center">We are here</td>
                  <td />
                  <td>{{ nowDate() + ' ' + nowTime() }}</td>
                </tr>
              </template>
              <template v-else-if="id.layer === currentLayer">
                <tr class="my-here">
                  <td>{{ currentLayer }}</td>
                  <td class="text-center">
                    <span class="text-center q-mr-sm">Just now</span>
                    <span v-for="(n, i) in id.nodes" :key="i" :style="nodeNameStyle(n)">{{ n }}</span>
                  </td>
                  <td v-if="id.smh">{{ id.smh }}</td>
                  <td v-else-if="coinbase !== ''"></td>
                  <td v-else />
                  <td>{{ id.time }}</td>
                </tr>
              </template>
              <template v-else>
                <tr :class="[id.layer < currentLayer ? 'my-before' : 'my-after']">
                  <td>{{ id.layer }}</td>
                  <td class="text-center">
                    <span v-for="(n, i) in id.nodes" :key="i" :style="nodeNameStyle(n)">{{ n }}</span>
                  </td>
                  <td v-if="id.smh" class="text-center">
                    {{ id.smh }}
                  </td><td v-else-if="coinbase !== '' && id.layer < currentLayer" class="text-center">
                    <q-icon name="report" color="red-8" size="20px"/>
                  </td>
                  <td v-else />
                  <td>{{ id.time }}</td>
                </tr>
              </template>

            </template>
            </tbody>
          </q-markup-table>

          <div class="row q-pa-sm justify-center" v-else>

            <template v-for="id in eligLayersIdList()" :key="id.layer">

              <div v-if="id.info" class="my-info q-pa-sm q-ma-xs flex column text-center justify-center">
                <div><strong>{{ id.layer }}</strong></div>
                <div v-for="(t, i) in id.info" :key="i">
                  {{ t }}
                  <q-icon v-if="id.link && (i === id.info.length -1)" name="info" color="blue" size="18px" class="cursor-pointer" style="top: -1px" @click="windowOpen(id.link)" />
                </div>
                <div v-if="currentLayer < id.layer">+ {{ layersDiffInTime(currentLayer, id.layer) }}</div>
                <div v-else>- {{ layersDiffInTime(currentLayer, id.layer) }}</div>
              </div>
              <div v-else-if="id.layer === -1" class="my-here q-pa-sm q-ma-xs flex column text-center justify-center">
                <div><strong>{{ currentLayer }}</strong></div>
                <div>We are here</div>
                <div>{{ nowDate() }}</div>
                <div>{{ nowTime() }}</div>
              </div>
              <div v-else-if="id.layer === currentLayer" class="my-here q-pa-sm q-ma-xs flex column text-center justify-center">
                <div><strong>{{ id.layer }}</strong></div>
                <span v-for="(v, k) in id.nodes" :key="k" :style="nodeNameStyle(v)">{{ v }}</span>
                <div>Just now</div>
                <div v-if="id.smh">{{ id.smh }}</div>
              </div>
              <div v-else-if="id.layer < currentLayer" class="my-before q-pa-sm q-ma-xs flex column text-center justify-center">
                <div><strong>{{ id.layer }}</strong></div>
                <span v-for="(v, k) in id.nodes" :key="k" :style="nodeNameStyle(v)">{{ v }}</span>
                <div>- {{ layersDiffInTime(id.layer, currentLayer) }}</div>
                <div v-if="id.smh">{{ id.smh }}</div>
                <div v-else-if="coinbase !== ''"><q-icon name="report" color="red-8" size="20px"/></div>
              </div>
              <div v-else class="my-after q-pa-sm q-ma-xs flex column text-center justify-center">
                <div><strong>{{ id.layer }}</strong></div>
                <span v-for="(v, k) in id.nodes" :key="k" :style="nodeNameStyle(v)">{{ v }}</span>
                <div>+ {{ layersDiffInTime(currentLayer, id.layer) }}</div>
              </div>
            </template>

          </div>

          <div class="text-center q-ma-lg">
            <div class="text-grey content-center text-center text-caption">
              <span style="font-size: 16px" class="q-mr-sm">🥔</span>
              <span>{{ potato }}</span>
              <q-btn flat dense icon="content_copy" size="10px" class="-q-ml-sm" @click="()=>copyToClipboard(potato)" />
            </div>
            <q-btn flat href="https://github.com/PlainLazy/SpacemeshLambaChunks" class="text-grey">
              <div class="flex flex-center">
                <svg viewBox="0 0 16 16" width="38" height="24">
                  <path fill="#9e9e9e" d="M8 0c4.42 0 8 3.58 8 8a8.013 8.013 0 0 1-5.45 7.59c-.4.08-.55-.17-.55-.38 0-.27.01-1.13.01-2.2 0-.75-.25-1.23-.54-1.48 1.78-.2 3.65-.88 3.65-3.95 0-.88-.31-1.59-.82-2.15.08-.2.36-1.02-.08-2.12 0 0-.67-.22-2.2.82-.64-.18-1.32-.27-2-.27-.68 0-1.36.09-2 .27-1.53-1.03-2.2-.82-2.2-.82-.44 1.1-.16 1.92-.08 2.12-.51.56-.82 1.28-.82 2.15 0 3.06 1.86 3.75 3.64 3.95-.23.2-.44.55-.51 1.07-.46.21-1.61.55-2.33-.66-.15-.24-.6-.83-1.23-.82-.67.01-.27.38.01.53.34.19.73.9.82 1.13.16.45.68 1.31 2.69.94 0 .67.01 1.3.01 1.49 0 .21-.15.45-.55.38A7.995 7.995 0 0 1 0 8c0-4.42 3.58-8 8-8Z" />
                </svg>
                <span style="font-size: 12px">PlainLazy</span>
              </div>
            </q-btn>
          </div>

        </template>

      </q-page>
    </q-page-container>

    <q-footer style="height: 30px">
      <svg width="100%" height="30" viewBox="0 0 1000 30" preserveAspectRatio="none" style="width: 100%" class="my-footer">
        <rect x="0" y="0" :width="edges.pos * 1000" height="10" class="my-before" />
        <rect :x="edges.pos * 1000 - 1" y="0" :width="3" height="10" fill="black" />
        <rect :x="edges.pos * 1000" y="0" :width="(1-edges.pos) * 1000" height="10" class="my-after" />
        <g v-for="l in eligLayersIdList()" :key="l.layer">
          <line
            v-if="l.nodes && l.pos"
            :x1="l.pos * 1000" y1="12" :x2="l.pos * 1000" y2="30" stroke="grey"
          />
        </g>
      </svg>
    </q-footer>

  </q-layout>
</template>

<style lang="scss">
@use 'quasar/src/css/variables';
body {
  font-size: 14px;
}
.q-btn {
  text-transform: none;
}
.body--light {
  header {
    color: black;
    background-color: white;
  }
  .my-info {
    background-color: $deep-purple-2;
  }
  .my-here {
    background-color: $blue-4;
  }
  .my-before {
    background-color: $green-2;
    fill: $green-2;
  }
  .my-after {
    background-color: $teal-2;
    fill: $teal-2;
  }
  .my-footer {
    background-color: white;
  }
}
.body--dark {
  header {
    color: $grey-4;
    background-color: #252429;
  }
  main {
    color: $grey-4;
    background-color: #252429;
  }
  .my-info {
    background-color: rgba(170, 88, 177, 0.3);
  }
  .my-here {
    background-color: rgb(79, 71, 49);
  }
  .my-before {
    background-color: rgba(101, 176, 66, 0.2);
    fill: rgba(101, 176, 66, 0.2);
  }
  .my-after {
    background-color: #323031;
    fill: #323031;
  }
  .my-footer {
    background-color: #252429;
  }
  .q-drawer {
    background-color: #252429;
  }
}
</style>

<script setup lang="ts">

import {computed, onBeforeMount, ref, watch, nextTick} from 'vue'
import {copyToClipboard, Notify, date, QUploader, Dark} from 'quasar'

const drawerLayers = ref(false)
const drawerOptions = ref(false)
const table = ref(false)
const markLayerId = 20500
const markLayerTime = new Date('2023-09-23T15:20:00+0300')
const markEpochNumber = 6
const markEpochBegin = new Date('2023-10-06 11:00:00+0300')
const layerDurationMinutes = 5
let serial = ref(0)
const options = ref(['12H'])

type eventT = {time:Date, info:string[], link?:string, type?:string}
const events:eventT[] = []

const eDurationMs = 14*24*60*60*1000  // 2 weeks
const T24EarlyOffsetMs = 96*60*60*1000
const T24EarlyDurationMs = (96+24)*60*60*1000  // +24h
const default12hOffsetMs = 228*60*60*1000  // -228h
const default12hDurationMs2 = (228+12)*60*60*1000  // +12h
const T24LateOffsetMs = 264*60*60*1000  // -264h
const T24LateDurationMs = (264+24)*60*60*1000  // +24h
const ePassedNum = Math.floor((Date.now() - markEpochBegin.getTime()) / eDurationMs)
const eCurrentNum = markEpochNumber + ePassedNum
const eCurrentBegin = markEpochBegin.getTime() + eDurationMs * (eCurrentNum - markEpochNumber)
events.push({time: new Date(eCurrentBegin), info: [`Epoch ${eCurrentNum}`]})
events.push({time: new Date(eCurrentBegin + T24EarlyOffsetMs), info: ['T24E begin'], type: 'E24', link: 'https://discord.gg/HzyA2Z7EKW'})
events.push({time: new Date(eCurrentBegin + T24EarlyDurationMs), info: ['T24E end'], type: 'E24'})
events.push({time: new Date(eCurrentBegin + default12hOffsetMs), info: ['12h begin'], type: '12H'})
events.push({time: new Date(eCurrentBegin + default12hDurationMs2), info: ['12h end'], type: '12H'})
events.push({time: new Date(eCurrentBegin + T24LateOffsetMs), info: ['T24L begin'], type: 'T24', link: 'https://discord.gg/HzyA2Z7EKW'})
events.push({time: new Date(eCurrentBegin + T24LateDurationMs), info: ['T24L end'], type: 'T24'})
events.push({time: new Date(eCurrentBegin + eDurationMs), info: ['108h end'], type: '12H'})

const optE24 = ref(false)
const opt12H = ref(false)
const optT24 = ref(false)

type nodeT = {
  serial: number
  show: boolean
  node: string
  layers: string
  color: string
}

const coinbase = ref('')
const nodes = ref<nodeT[]>([
  {
    serial: serial.value++,
    show: true,
    node: 'ABC123',
    layers: '24100, 24190, 24210, 24250, 24299, 25302, 25400, 26500, 26636, 27720, 27009',
    color: '#b9ebbb'
  },
  {
    serial: serial.value++,
    show: true,
    node: 'XYZ321',
    layers: '[{"layer": 25189, "slots": 1}, {"layer": 25250, "slots": 1}, {"layer": 28001, "slots": 1}]',
    color: '#c5a5e6'
  }
])

const loading = ref(true)
let rewards = [{layer: 0, total: 0}]

const currentLayer = computed(() => {
  const layersGapMinutes = (Date.now() - markLayerTime.getTime()) / 1000 / 60
  const layersDelta = Math.floor(layersGapMinutes / layerDurationMinutes)
  return markLayerId + layersDelta
})

type edgesT = {
  firstLayer:number
  lastLayer:number
  pos:number
}

const edges = computed(() => {
  const e:edgesT = {
    firstLayer: Number.MAX_VALUE,
    lastLayer: Number.MIN_VALUE,
    pos: 0,
  };
  [...initLayers.value].forEach(m => {
    if (e.firstLayer > m.layer) e.firstLayer = m.layer
    if (e.lastLayer < m.layer) e.lastLayer = m.layer
  })
  e.pos = (currentLayer.value - e.firstLayer) / (e.lastLayer - e.firstLayer)
  return e
})

const getLayerByTime = (time:Date):number => {
  const layersGapMinutes = (time.getTime() - markLayerTime.getTime()) / 1000 / 60
  const layersDelta = Math.floor(layersGapMinutes / layerDurationMinutes)
  return markLayerId + layersDelta
}

const getTimeByLayer = (targetLayerId:number):Date => {
  return new Date(markLayerTime.getTime() + (targetLayerId - markLayerId) * layerDurationMinutes * 60 * 1000)
}

const nodeColor = (_node:string) => {
  const n = nodes.value.find(n => n.node === _node)
  return n ? n.color : null
}

type layerT = {
  nodes?:string[]
  layer:number
  smh?:number
  info?:string[]
  link?:string
  pos?:number
  time?:string
}

const initLayers = computed(():layerT[] => events
  .filter(e => e.type == null || options.value.indexOf(e.type) !== -1)
  .map(e => ({layer: getLayerByTime(e.time), info: e.info, link: e.link}))
)

const eligLayersIdList = ():layerT[] => {

  const merged:layerT[] = [...initLayers.value]

  nodes.value.forEach(n => {
    if (n.show) {
      const layers = n.layers || ''
      const jsonNamedNumbers = [...layers.matchAll(/"layer":[\s|\t]*(\d+)/g)].map(m => Number(m[1]))
      if (jsonNamedNumbers.length > 0) {
        jsonNamedNumbers.forEach(l => {
          merged.push({nodes: [n.node], layer: l})
        })
      } else {
        const bunchOfNumbers = (layers.match(/\d+/g) || []).map(a=>Number(a))
        bunchOfNumbers.forEach(l => {
          merged.push({nodes: [n.node], layer: l})
        })
      }
    }
  })
  merged.sort((a,b)=>a.layer-b.layer)

  // group by layer
  const grouped:layerT[] = []
  let last:layerT
  merged.forEach(m => {
    if (last && last.layer === m.layer && m.nodes && last.nodes) {
      last.nodes.push(m.nodes[0])
    } else {
      grouped.push(m)
      last = m
    }
  })

  grouped.forEach(m => {
    m.pos = (m.layer - edges.value.firstLayer) / (edges.value.lastLayer - edges.value.firstLayer)
  })

  if (Array.isArray(rewards)) {
    grouped.forEach(m => {
      const rw = rewards.find(r => r.layer === m.layer)
      if (rw) {
        m.smh = Math.round(rw.total / 1000000) / 1000
      }
    })
  }

  grouped.forEach(l => {
    l.time = date.formatDate(getTimeByLayer(l.layer), 'YYYY-MM-DD HH:mm')
  })

  const i = grouped.findIndex((e) => e.layer > currentLayer.value)
  return i !== -1 ? [...grouped.slice(0, i), {layer: -1, nodes: []}, ...grouped.slice(i)] : [...grouped, {layer: -1, nodes: []}]
}

const nodeCreate = () => {
  nodes.value.push({
    serial: serial.value++,
    show: true,
    node: serial.value.toString(16).padStart(4, '0'),
    layers: '',
    color: '',
  })
}

const nodeDelete = (_serial:number) => {
  nodes.value = [ ... nodes.value.filter(n => n.serial != _serial) ]
}

const nodeAdjust = (_serial:number) => {
  const n = nodes.value.find(e => e.serial === _serial)
  if (n) {
    let layerNumbers:number[] = []
    const layers = n.layers || ''
    const jsonNamedNumbers = [...layers.matchAll(/"layer":[\s|\t]*(\d+)/g)].map(m => Number(m[1]))
    if (jsonNamedNumbers.length > 0) {
      jsonNamedNumbers.forEach(l => {
        layerNumbers.push(l)
      })
    } else {
      const bunchOfNumbers = (layers.match(/\d+/g) || []).map(a=>Number(a))
      bunchOfNumbers.forEach(l => {
        layerNumbers.push(l)
      })
    }
    layerNumbers.sort((a,b)=>a-b)
    n.layers = layerNumbers.join(' ')
  }
}

const isLayersOkay = (layers:string):boolean => (/^[\d\s]*$/.test(layers))

const layersDiffInTime = (l0:number, l1:number) => {
  let sec = Math.abs(l1 - l0) * layerDurationMinutes * 60
  let a = []
  if (sec > 60*60*24) {
    const d = Math.floor(sec / 60 / 60 / 24)
    a.push(d.toString() + 'd')
    sec -= d * 60 * 60 * 24
  }
  if (sec > 60*60 || a.length > 0) {
    const h = Math.floor(sec / 60 / 60)
    a.push(h.toString().padStart(2, '0') + 'h')
    sec -= h * 60 * 60
  }
  if (a.length < 2 && (sec > 60 || a.length > 0)) {
    const m = Math.floor(sec / 60)
    a.push(m.toString().padStart(2, '0') + 'm')
  }
  return a.join(' ')
}

const nowDate = () => date.formatDate(Date.now(), 'YYYY-MM-DD')
const nowTime = () => date.formatDate(Date.now(), 'HH:mm')

const rewardsPerPage  = 2000
const rewardsLimit    = 20000  // there can be more than one reward per layer, so more than 4032 per epoch, lets limit 20k

const rewardsCheck = async () => {

  drawerLayers.value = false

  rewards = []
  loading.value = true

  if ((coinbase.value || '').length > 0) {
    const wallets = [ ... coinbase.value.matchAll(/sm1qqqqqq[0-9a-z]*/g)]
    for (const w of wallets) {

      // before 72576
      /*
      const uri = `https://mainnet-explorer-3-api.spacemesh.network/accounts/${w}/rewards`
      let pageCount = 0
      await fetch(uri)
        .then(async index => {

          const j0 = await index.json()
          pageCount = Math.ceil(Number(j0['pagination']['totalCount']) / rewardsPerPage)

          let p = pageCount
          while (rewards.length <= rewardsLimit && p > 0) {
            await fetch(`${uri}?page=${p}&pagesize=${rewardsPerPage}`)
              .then(async chunk => {
                const j1 = await chunk.json()
                rewards = [ ... rewards, ... j1['data'] ]
              })
            p--
          }

        })
        .catch(() => {
          Notify.create({message: `failed to fetch wallet ${w} rewards from explorer`, color: 'red', position: 'top'})
        })
      */

      // after 72576
      // pagination reversed
      const uri = `https://mainnet-explorer-api.spacemesh.network/accounts/${w}/rewards`
      let pageCount = 0
      await fetch(uri)
        .then(async index => {
          const j0 = await index.json()
          pageCount = Math.ceil(Number(j0['pagination']['totalCount']) / rewardsPerPage)
          let p = 1
          while (rewards.length <= rewardsLimit && p <= pageCount) {
            await fetch(`${uri}?page=${p}&pagesize=${rewardsPerPage}`)
              .then(async chunk => {
                const j1 = await chunk.json()
                rewards = [ ... rewards, ... j1['data'] ]
              })
            p++
          }
        })
        .catch(() => {
          Notify.create({message: `failed to fetch wallet ${w} rewards from explorer`, color: 'red', position: 'top'})
        })

    }
  }

  const grouped:{layer:number, total:number}[] = []
  let last
  rewards.forEach(r => {
    if (last && last?.layer === r?.layer) {
      last.total += r?.total
    } else {
      grouped.push(r)
      last = r
    }
  })
  rewards = [... grouped]

  loading.value = false

}

const uploaderRef = ref<QUploader>()
const SMMUpload = (e:Event) => {
  uploaderRef.value?.reset()
  nextTick(() =>
    uploaderRef.value?.pickFiles(e)
  )
}

const SMMFileAdded = async (files:readonly unknown[]) => {
  if (files.length < 1) return
  const content = await (files[0] as Blob).text()
  console.log(content)

  let j:any[] = []
  try {
    j = JSON.parse(content)
    if (!Array.isArray(j)) {
      throw Error('array expected')
    }
    // todo: check more
  } catch (e) {
    Notify.create({message: 'bad json', color: 'red', position: 'top'})
    return
  }

  const nodesImported:nodeT[] = j.map(smm => ({
    serial: serial.value++,
    show: true,
    node: smm.nodeName,
    layers: smm.eligibilities,
    color: '',
  } as nodeT))

  nodes.value = [...nodesImported]

}

const SMMDownload = () => {
  const eligibilities = (n:nodeT):string => {
    let list:number[] = []
    const layers = n.layers || ''
    const jsonNamedNumbers = [...layers?.matchAll(/"layer":[\s|\t]*(\d+)/g)].map(m => Number(m[1]))
    if (jsonNamedNumbers.length > 0) {
      jsonNamedNumbers.forEach(l => {
        list.push(l)
      })
    } else {
      const bunchOfNumbers = (layers?.match(/\d+/g) || []).map(a=>Number(a))
      bunchOfNumbers.forEach(l => {
        list.push(l)
      })
    }
    return list.join(',')
  }
  const url = window.URL.createObjectURL(
    new Blob([
      new Uint8Array([0xEF, 0xBB, 0xBF]),
      JSON.stringify(
        nodes.value.map(n => ({
          nodeName: n.node,
          eligibilities: eligibilities(n),
        })),
        null,
        '  '
      )
    ],
    {type: 'text/json;charset=utf-8;'}
  ))
  const a = document.createElement('a')
  a.style.display = 'none'
  a.href = url
  a.download = 'SM-Monitor ' + date.formatDate(Date.now(), 'YYYY-MM-DD HH:mm') + '.json'
  document.body.appendChild(a)
  a.click()
  window.URL.revokeObjectURL(url)
}

const windowOpen = (uri:string) => {
  window.open(uri, '_blank')
}

watch(nodes, () => localStorage.setItem('nodes', JSON.stringify(nodes.value)), {deep: true})
watch(coinbase, () => localStorage.setItem('coinbase', coinbase.value))
watch(table, () => localStorage.setItem('table', String(table.value)))
watch([optE24, opt12H, optT24], () => {
  options.value = [optE24.value ? 'E24' : '', opt12H.value ? '12H' : '', optT24.value ? 'T24' : ''].filter(v => v)
  localStorage.setItem('options', String(options.value))
})
watch(() => Dark.isActive, isDark => { localStorage.setItem('dark', String(isDark)) })

onBeforeMount(async () => {

  const raw = localStorage.getItem('nodes')
  if (raw) {
    const l = JSON.parse(raw)
    nodes.value = [... l]
    nodes.value.forEach(n => {
      if (serial.value < n.serial + 1) {
        serial.value = n.serial + 1
      }
      if (n.show == null) {
        n.show = true
      }
    })
  }

  coinbase.value = localStorage.getItem('coinbase') || ''

  table.value = Boolean(localStorage.getItem('table') === 'true')

  const isDark:string|null = localStorage.getItem('dark')
  Dark.set(isDark != null ? Boolean(isDark === 'true') : 'auto')

  let opts:string|null = localStorage.getItem('options')
  if (opts == null) opts = '12H'
  options.value = opts.split(',')
  options.value.forEach((v:string) => {
    if (v === 'E24') optE24.value = true
    if (v === '12H') opt12H.value = true
    if (v === 'T24') optT24.value = true
  })

  await rewardsCheck()

})

const potato = 'sm1qqqqqqq04h9wlpqgfj878kfumrfsk0h92u0sxjcmjjpux'

const nodeNameStyle = (n:string) => {
  const c = nodeColor(n)
  return Dark.isActive ?
    (c ?
      {borderColor: c, borderWidth: '2px', borderRadius: '12px', borderStyle: 'solid', padding: '1px 7px 1px 7px', lineHeight: '15px'} :
      {borderRadius: '12px', padding: '4px 10px 4px 10px'}
    ) :
    {backgroundColor: c, borderRadius: '12px', padding: '4px 10px 4px 10px', lineHeight: '15px'}
}

</script>
