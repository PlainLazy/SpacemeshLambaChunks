<template>
  <q-layout view="hhh lpR lFf" style="min-width: 870px">

    <q-header class="bg-white" style="color: black; box-shadow: 0 -8px 34px rgba(200, 200, 200, 0.8);">
      <q-toolbar style="height: 64px;" class="q-pa-none">
        <q-btn flat @click="drawer = true" icon="menu" size="25px" />
        <q-space/>
        SpaceMesh Lamba Chunks
        <q-space/>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="drawer"
      :width="810"
      behavior="mobile"
      bordered
      class="q-ma-sm text-center"
    >

      <div class="row items-center">
        <q-input
          v-model="coinbase"
          label="coinbase sm1qqqqqqq... or several with any separator"
          filled dense stack-label
          style="width: 795px"
        >
          <template v-slot:prepend>
            <q-icon name="account_balance_wallet" size="20px" :style="{color: coinbase ? '#217ad2' : '#636363'}" />
          </template>
          <template v-slot:append>
            <q-icon name="send" class="cursor-pointer" color="blue-8" style="height: 40px" @click="rewardsCheck"/>
          </template>
        </q-input>
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
            <q-icon name="palette" class="cursor-pointer material-icons-outlined" size="20px" :style="{'color': n.color, 'text-shadow': n.color ? '0 2px 10px rgba(0,0,0,0.6)' : ''}">
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
          style="width: 600px"
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

    </q-drawer>

    <q-page-container class="bg-blue-2">
      <q-page class="items-center justify-center column">

        <template v-if="loading">
          <q-spinner size="100px" color="blue-8" />
        </template>
        <template v-else>

          <div class="row q-pa-sm justify-center" >

            <template v-for="id in eligLayersIdList()" :key="id.layer">

              <div v-if="id.info" class="q-pa-sm q-ma-xs bg-deep-purple-2 flex column text-center justify-center" style="border-radius: 20px">
                <div><strong>{{ numberFormat(id.layer) }}</strong></div>
                <div v-for="(t, i) in id.info" :key="i">{{ t }}</div>
                <div v-if="currentLayer < id.layer">in {{ layersDiffInTime(currentLayer, id.layer) }}</div>
                <div v-else>{{ layersDiffInTime(currentLayer, id.layer) }} ago</div>
              </div>
              <div v-else-if="id.layer === -1" class="q-pa-sm q-ma-xs bg-blue-4 flex column text-center justify-center" style="border-radius: 20px">
                <div><strong>{{ numberFormat(currentLayer) }}</strong></div>
                <div>We are here</div>
                <div>{{ nowDate() }}</div>
                <div>{{ nowTime() }}</div>
              </div>
              <div v-else-if="id.layer === currentLayer" class="q-pa-sm q-ma-xs bg-blue-4 rounded-borders flex column text-center justify-center">
                <div><strong>{{ numberFormat(id.layer) }}</strong></div>
                <div v-for="(v, k) in id.nodes" :key="k" :style="{backgroundColor: nodeColor(v), borderRadius: '8px'}" class="q-px-sm">
                  {{ v }}
                </div>
                <div>JUST NOW</div>
                <div v-if="id.smh">+{{ id.smh }}</div>
              </div>
              <div v-else-if="id.layer < currentLayer" class="q-pa-sm q-ma-xs bg-green-3 rounded-borders flex column text-center justify-center">
                <div><strong>{{ numberFormat(id.layer) }}</strong></div>
                <div v-for="(v, k) in id.nodes" :key="k" :style="{backgroundColor: nodeColor(v), borderRadius: '8px'}" class="q-px-sm">
                  {{ v }}
                </div>
                <div>{{ layersDiffInTime(id.layer, currentLayer) }} ago</div>
                <div v-if="id.smh">+{{ id.smh }}</div>
                <div v-else-if="coinbase !== ''"><q-icon name="report" color="red-8" size="20px"/></div>
              </div>
              <div v-else class="q-pa-sm q-ma-xs bg-teal-2 rounded-borders flex column text-center justify-center">
                <div><strong>{{ numberFormat(id.layer) }}</strong></div>
                <div v-for="(v, k) in id.nodes" :key="k" :style="{backgroundColor: nodeColor(v), borderRadius: '8px'}" class="q-px-sm">
                  {{ v }}
                </div>
                <div>in {{ layersDiffInTime(currentLayer, id.layer) }}</div>
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
                PlainLazy
              </div>
            </q-btn>
          </div>

        </template>

      </q-page>
    </q-page-container>

  </q-layout>
</template>

<style lang="scss" scoped>
.q-btn {
  text-transform: none;
}
</style>

<script setup lang="ts">

import {computed, onBeforeMount, ref, watch} from 'vue'
import { copyToClipboard, Notify } from 'quasar'

const drawer = ref(false)
const beaconLayerId = 20500
const beaconLayerTime = '2023-09-23T15:20:00+0300'
const layerDurationMinutes = 5
let serial = ref(0)

type eventT = {time:Date, info:string[]}
const events:eventT[] = [
  //{time: new Date('2023-10-06 11:00:00+0300'), info: ['Epoch 5 End', 'Epoch 6 Begin']},
  //{time: new Date('2023-10-15 23:00:00+0300'), info: ['PoST 5', 'Begin']},
  //{time: new Date('2023-10-16 11:00:00+0300'), info: ['PoST 5', '12h End']},
  //{time: new Date('2023-10-20 11:00:00+0300'), info: ['PoST 5', '108h End']},
  {time: new Date('2023-10-20 11:00:00+0300'), info: ['Epoch 6 End', 'Epoch 7 Begin']},
  {time: new Date('2023-10-29 23:00:00+0300'), info: ['PoST 6', 'Begin']},
  {time: new Date('2023-10-30 10:00:00+0300'), info: ['PoST 6', '12h End']},
  {time: new Date('2023-11-03 11:00:00+0300'), info: ['PoST 6', '108h End']},
  {time: new Date('2023-11-03 11:00:00+0300'), info: ['Epoch 7 End', 'Epoch 8 Begin']},
]

const coinbase = ref('')
const nodes = ref([
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
  const layersGapMinutes = (Date.now() - new Date(beaconLayerTime).getTime()) / 1000 / 60
  const layersDelta = Math.floor(layersGapMinutes / layerDurationMinutes)
  return beaconLayerId + layersDelta
})

const getLayerByTime = (time:Date):number => {
  const layersGapMinutes = (time.getTime() - new Date(beaconLayerTime).getTime()) / 1000 / 60
  const layersDelta = Math.floor(layersGapMinutes / layerDurationMinutes)
  return beaconLayerId + layersDelta
}

const nodeColor = (_node:string) => {
  const n = nodes.value.find(n => n.node === _node)
  return n ? n.color : null;
}

type layerT = {nodes?:string[], layer:number, smh?:number, info?:string[]}

const eligLayersIdList = ():layerT[] => {
  const merged:layerT[] = []

  events.forEach(e => {
    merged.push({layer: getLayerByTime(e.time), info: e.info})
  })

  nodes.value.forEach(n => {
    if (n.show) {
      const logStyleLayers = [...n.layers.matchAll(/"layer": (\d+)/g)].map(e => Number(e[1]))
      if (logStyleLayers.length > 0) {
        logStyleLayers.forEach(l => {
          merged.push({nodes: [n.node], layer: l})
        })
      } else {
        const uiStyleLayers = (n.layers.match(/\d+/g) || []).map(a=>Number(a))
        uiStyleLayers.forEach(l => {
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

  if (Array.isArray(rewards)) {
    grouped.forEach(m => {
      const rw = rewards.find(r => r.layer === m.layer)
      if (rw) {
        m.smh = Math.round(rw.total / 1000000) / 1000
      }
    })
  }

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
    const logStyleLayers = [...n.layers.matchAll(/"layer": (\d+)/g)].map(e => Number(e[1]))
    if (logStyleLayers.length > 0) {
      logStyleLayers.forEach(l => {
        layerNumbers.push(l)
      })
    } else {
      const uiStyleLayers = (n.layers.match(/\d+/g) || []).map(a=>Number(a))
      uiStyleLayers.forEach(l => {
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

const nowDate = () => {
  const d = new Date(Date.now())
  return [d.getDate(), d.getMonth() + 1, d.getFullYear()]
    .map(e => e.toString().padStart(2, '0'))
    .join('-')
}

const nowTime = () => {
  const d = new Date(Date.now())
  return [d.getHours(), d.getMinutes(), d.getSeconds()]
    .map(e => e.toString().padStart(2, '0'))
    .join(':')
}

const numberFormat = (n:number):string => {
  let l0 = String(n).split('')
  let s1 = ''
  l0.forEach((v, i) => {
    s1 += v
    if ((l0.length - i) % 3 === 1 && i < l0.length - 1) {
      s1 += ' '
    }
  })
  return s1
}

const pageSize = 500

const rewardsCheck = async () => {

  drawer.value = false

  rewards = []
  loading.value = true

  if ((coinbase.value || '').length > 0) {
    //const wallets = coinbase.value.split(' ')
    const wallets = [ ... coinbase.value.matchAll(/sm1qqqqqq[0-9a-z]*/g)]
    for (const w of wallets) {
      const uri = `https://mainnet-explorer-api.spacemesh.network/accounts/${w}/rewards`
      let pageCount = 0
      await fetch(uri)
        .then(async index => {

          const j0 = await index.json()
          pageCount = Math.ceil(Number(j0['pagination']['totalCount']) / pageSize)
          //console.log('pageCount', pageCount)

          let p = pageCount
          while (rewards.length <= 4032 && p > 0) {
            await fetch(`${uri}?page=${p}&pagesize=${pageSize}`)
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
    }
    //console.log('//', rewards)
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

watch(nodes, () => localStorage.setItem('nodes', JSON.stringify(nodes.value)), {deep: true})
watch(coinbase, () => localStorage.setItem('coinbase', coinbase.value))

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

  await rewardsCheck()

})

const potato = 'sm1qqqqqqq04h9wlpqgfj878kfumrfsk0h92u0sxjcmjjpux'

</script>
