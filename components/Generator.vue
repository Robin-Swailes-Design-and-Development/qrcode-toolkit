<script setup lang="ts">
import { debounce } from 'perfect-debounce'
import { sendParentEvent } from '../logic/messaging'
import { generateQRCode } from '../logic/generate'
import { dataUrlGeneratedQRCode, defaultGeneratorState, generateQRCodeInfo, hasParentWindow, isLargeScreen, qrcode } from '../logic/state'
import { view } from '../logic/view'
import type { State } from '../logic/types'
import { MarkerSubShapeIcons, MarkerSubShapes, PixelStyleIcons, PixelStyles } from '../logic/types'
import { getAspectRatio, sendQRCodeToCompare } from '../logic/utils'
import { ref, computed, reactive, watch } from 'vue';
import { useElementBounding, useDropZone } from '@vueuse/core';
import OptionItem from 'Robin-Swailes-Design-and-Development-QR/components/OptionItem.vue' //VTooltip
import OptionSlider from 'Robin-Swailes-Design-and-Development-QR/components/OptionSlider.vue'
import OptionCheckbox from 'Robin-Swailes-Design-and-Development-QR/components/OptionCheckbox.vue'
import OptionColor from 'Robin-Swailes-Design-and-Development-QR/components/OptionColor.vue'
import OptionSelectGroup from 'Robin-Swailes-Design-and-Development-QR/components/OptionSelectGroup.vue'
import SettingsMarkerStyle from 'Robin-Swailes-Design-and-Development-QR/components/SettingsMarkerStyle.vue'
import SettingsMargin from 'Robin-Swailes-Design-and-Development-QR/components/SettingsMargin.vue'
import SettingsRandomRange from 'Robin-Swailes-Design-and-Development-QR/components/SettingsRandomRange.vue'
import ImageUpload from 'Robin-Swailes-Design-and-Development-QR/components/ImageUpload.vue'
// import DialogScan from 'Robin-Swailes-Design-and-Development-QR/components/DialogScan.vue'


const props = defineProps<{
  state: State
}>()

const rightPanelEl = ref<HTMLElement>()
const uploadTarget = ref<'image' | 'qrcode'>()
const state = computed(() => props.state.qrcode)
const rightPanelRect = reactive(useElementBounding(rightPanelEl))
const floating = computed(() => rightPanelRect.top < 10 && isLargeScreen.value)


const canvas = ref<HTMLCanvasElement>()

async function run() {
  if (!canvas.value)
    return
  await generateQRCode(canvas.value, state.value)
  dataUrlGeneratedQRCode.value = canvas.value.toDataURL()
}

function download() {
  if (!canvas.value)
    return
  const a = document.createElement('a')
  a.href = dataUrlGeneratedQRCode.value!
  a.download = `${state.value.text.replace(/\W/g, '_')}[${state.value.ecc}_x${state.value.scale}].png`
  a.click()
}

function reset() {
  // eslint-disable-next-line no-alert
  if (confirm('Are you sure to reset state?'))
    Object.assign(state.value, defaultGeneratorState())
}

function downloadState() {
  const data = {
    '//': 'Generator state of RS\'s QR Toolkit',
    ...state.value,
  }
  

  const text = JSON.stringify(data, null, 2)
  const a = document.createElement('a')
  a.href = URL.createObjectURL(new Blob([text], { type: 'application/json' }))
  a.download = `qr-options-${state.value.text.replace(/\W/g, '_')}.json`
  a.click()
}

// save state to db 
function saveState() {
  
}

// load state from db 
function loadState() {

}

async function readState(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file)
    return

  const reader = new FileReader()
  const promise = new Promise<string>((resolve, reject) => {
    reader.onload = () => {
      resolve(reader.result as any)
    }
    reader.onerror = reject
  })
  reader.readAsText(file)
  const text = await promise
  try {
    const data = JSON.parse(text)
    // eslint-disable-next-line no-alert
    if (confirm('Are you sure to override the state with file uploaded?')) {
      const keys = Object.keys(state.value)
      for (const key of keys) {
        if (key in data)
          // @ts-expect-error anyway
          state.value[key] = data[key]
      }
    }
  }
  catch (e) {
    // eslint-disable-next-line no-alert
    alert('Invalid JSON file')
  }
}

const colorPalette = [
  '#000000', // Black
  '#FFFFFF', // White
  '#808080', // Gray
  '#C0C0C0', // Silver
  '#1E90FF', // DodgerBlue
  '#4169E1', // RoyalBlue
  '#32CD32', // LimeGreen
  '#228B22', // ForestGreen
  '#FF0000', // Red
  '#DC143C', // Crimson
  '#FFA500', // Orange
  '#FFD700', // Gold
  '#FFFF00', // Yellow
  '#8B4513', // SaddleBrown
  '#800080', // Purple
  '#FF1493', // DeepPink
  '#00CED1', // DarkTurquoise
];

const patternColors = [...colorPalette];
const backgroundColors = [...colorPalette];

const debouncedRun = debounce(run, 250, { trailing: true });

const showPixelSize = computed(() => {
  const styles = ['diamond', 'dot', 'square'];
  return styles.includes(state.value.pixelStyle) ||
         styles.includes(state.value.markerStyle);
});

const mayNotScannable = computed(() => {
  if ((state.value.marginNoise || state.value.backgroundImage) && state.value.marginNoiseSpace === 'none')
    return true
  if (state.value.effect === 'crystalize' && state.value.effectCrystalizeRadius / state.value.scale > 0.4)
    return true
  if (
    state.value.effect === 'liquidify'
    && (
      (state.value.effectLiquidifyRadius / state.value.scale > 0.4)
      || (Math.abs(state.value.effectLiquidifyThreshold - 128) > 32)
    )
  )
    return true
  if (state.value.markerShape === 'tiny-plus')
    return true
  if (!['square', 'circle', 'box'].includes(state.value.markerSub))
    return true
  if (Math.abs(state.value.transformPerspectiveX) > 0.1 || Math.abs(state.value.transformPerspectiveY) > 0.1)
    return true
  if (state.value.transformScale > 1.05)
    return true
})

const hasNonCenteredMargin = computed(() => {
  if (typeof state.value.margin === 'number')
    return false
  return state.value.margin.top !== state.value.margin.bottom
    || state.value.margin.left !== state.value.margin.right
})

function sendCompare() {
  sendQRCodeToCompare(props.state)
  view.value = 'compare'
}

function sendToWebUI() {
  sendParentEvent('setControlNet', dataUrlGeneratedQRCode.value!)
}

function toggleMarkerStyleExpand() {
  if (!props.state.qrcode.markers?.length) {
    props.state.qrcode.markers = [
      {
        markerShape: props.state.qrcode.markerShape,
        markerStyle: props.state.qrcode.markerStyle,
        markerInnerShape: props.state.qrcode.markerInnerShape,
      },
      {
        markerShape: props.state.qrcode.markerShape,
        markerStyle: props.state.qrcode.markerStyle,
        markerInnerShape: props.state.qrcode.markerInnerShape,
      },
    ]
  }
  else {
    props.state.qrcode.markers = []
  }
}

const socialLogos = [
  { name: 'Instagram', file: '/img/logos/instagram.png' },
  { name: 'Facebook', file: '/img/logos/facebook.png' },
  { name: 'Twitter', file: '/img/logos/twitter.png' },
  { name: 'LinkedIn', file: '/img/logos/linkedin.png' },
  { name: 'YouTube', file: '/img/logos/youtube.png' },
  { name: 'TikTok', file: '/img/logos/tik-tok.png' },
  { name: 'WhatsApp', file: '/img/logos/whatsapp.png' },
];

function selectLogo(logoFile: string) {
  state.value.logoImage = logoFile;
}
const uploadQR = ref<string>()

watch(
  () => state.value,
  () => debouncedRun(),
  { deep: true, immediate: true },
)
</script>
<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-lg-7">
        <div class="d-flex flex-column gap-3">
          <textarea v-model="state.text" placeholder="Target text or URL" class="form-control"></textarea>
          
          <ul class="nav nav-tabs" id="optionTabs" role="tablist">
            <li class="nav-item" role="presentation">
              <button class="nav-link active" id="style-tab" data-bs-toggle="tab" data-bs-target="#style" type="button" role="tab" aria-controls="style" aria-selected="true">Style & Markers</button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link" id="colors-tab" data-bs-toggle="tab" data-bs-target="#colors" type="button" role="tab" aria-controls="colors" aria-selected="false">Colours</button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link" id="logo-bg-tab" data-bs-toggle="tab" data-bs-target="#logo-bg" type="button" role="tab" aria-controls="logo-bg" aria-selected="false">Logo & Background</button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link" id="other-tab" data-bs-toggle="tab" data-bs-target="#other" type="button" role="tab" aria-controls="other" aria-selected="false">Other Options</button>
            </li>
          </ul>
          
          <div class="tab-content" id="optionTabsContent">
            <div class="tab-pane fade show active" id="style" role="tabpanel" aria-labelledby="style-tab">
              <div class="card">
                <div class="card-body">
                  <div class="mb-3">
                    <label class="form-label d-block">Pixel Style</label>
                    <OptionSelectGroup v-model="state.pixelStyle" :options="PixelStyles" :classes="PixelStyleIcons" />
                  </div>
                  
                  <div class="mb-3 border-top pt-3">
                    <label class="form-label">{{ state.markers.length ? 'Marker 1' : 'Markers' }}</label>
                    <button class="d-none btn btn-outline-secondary btn-sm float-end" @click="toggleMarkerStyleExpand">
                      <i :class="state.markers.length ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
                    </button>
                  </div>
                  
                  <template v-if="!state.markers.length">
                    <SettingsMarkerStyle :state="state" nested number="Marker" />
                  </template>
                  <template v-else>
                    <SettingsMarkerStyle :state="state" nested />
                    <div class="mb-3">
                      <label class="form-label">Marker 2</label>
                    </div>
                    <SettingsMarkerStyle :state="state.markers[0]" nested />
                    <div class="mb-3">
                      <label class="form-label">Marker 3</label>
                    </div>
                    <SettingsMarkerStyle :state="state.markers[1]" nested />
                  </template>
                  
                  <div v-if="qrcode?.version !== 1" class="mb-3 d-none">
                    <label class="form-label">Sub Markers</label>
                    <OptionSelectGroup v-model="state.markerSub" :options="MarkerSubShapes" :classes="MarkerSubShapeIcons" />
                  </div>
                  
                  <div class="mb-3 d-none">
                    <label class="form-label d-block">Rotate</label>
                    <OptionSelectGroup v-model="state.rotate" :options="[0, 90, 180, 270]" :titles="['0°', '90°', '180°', '270°']" />
                  </div>
                    <div v-if="showPixelSize" class="mb-3 border-top pt-3">
                        <label class="form-label">Pixel Size</label>
                        <OptionSlider v-model="state.dotScale" :min="0.7" :max="1.1" :step="0.01" unit="%" />
                    </div>
                </div>
              </div>
            </div>
            
            <div class="tab-pane fade" id="colors" role="tabpanel" aria-labelledby="colors-tab">
              <div class="card">
                <div class="card-body">
                  <div class="mb-4">
                    <label class="form-label">Pattern Color</label>
                    <div class="d-flex align-items-center mb-2">
                      <OptionColor v-model="state.darkColor" class="me-2" />
                    </div>
                    <div class="d-flex flex-wrap gap-2">
                      <button v-for="color in patternColors" 
                              :key="color" 
                              class="btn btn-sm" 
                              :style="{ backgroundColor: color, width: '30px', height: '30px', border: color === state.darkColor ? '2px solid #007bff' : '1px solid #ced4da' }"
                              @click="state.darkColor = color"
                              :title="color"></button>
                    </div>
                  </div>

                  <div class="mb-3 border-top pt-3">
                    <label class="form-label">Background Color</label>
                    <div class="d-flex align-items-center mb-2">
                      <OptionColor v-model="state.lightColor" class="me-2" />
                    </div>
                    <div class="d-flex flex-wrap gap-2">
                      <button v-for="color in backgroundColors" 
                              :key="color" 
                              class="btn btn-sm" 
                              :style="{ backgroundColor: color, width: '30px', height: '30px', border: color === state.lightColor ? '2px solid #007bff' : '1px solid #ced4da' }"
                              @click="state.lightColor = color"
                              :title="color"></button>
                    </div>
                  </div>

                  <div class="form-check">
                    <input class="form-check-input" type="checkbox" v-model="state.invert" id="invertColors">
                    <label class="form-check-label" for="invertColors">Invert Colours</label>
                  </div>
                </div>
              </div>
            </div>
            
            





            <div class="tab-pane fade" id="logo-bg" role="tabpanel" aria-labelledby="logo-bg-tab">
    <div class="card">
      <div class="card-body">                  
        <div class="mb-3">
          <label class="form-label">Logo</label>
          <div class="d-flex align-items-center mb-2">
            <div class="position-relative" style="width: 100px; height: 100px;">
              <ImageUpload v-model="state.logoImage" />
              <img
                   :src="state.logoImage" 
                   alt="Logo" 
                   class="position-absolute top-0 start-0 w-100 h-100 p-2 object-fit-contain"
              >
            </div>

            <button v-if="state.logoImage" class="btn btn-outline-secondary ms-2" @click="state.logoImage = undefined">
              <i class="bi-x"></i>
            </button>
          </div>
          <div class="d-flex flex-wrap gap-2 mt-2">
            <button v-for="logo in socialLogos" 
                    :key="logo.name" 
                    class="btn p-1"
                    @click="selectLogo(logo.file)"
                    :title="logo.name">
              <img :src="logo.file" :alt="logo.name" style="width: 32px; height: 32px;">
            </button>
          </div>
        </div>
        
        <div class="mb-3">
          <label class="form-label">Logo Scale</label>
          <OptionSlider v-model="state.logoScale" :min="0.2" :max="0.3" :step="0.01" unit="%" />
        </div>


                  <div class="mb-3 border-top pt-3">
                    <label class="form-label">Background</label>
                    <div class="d-flex align-items-center">
                      <OptionColor v-if="state.backgroundImage?.startsWith('#')" v-model="state.backgroundImage" />
                      <button v-else class="btn btn-outline-secondary position-relative">
                        <img v-if="state.backgroundImage" :src="state.backgroundImage" class="top-0 start-0 w-100 h-100 rounded opacity-50">
                        <i class="bi-upload"></i> Upload
                        <ImageUpload v-model="state.backgroundImage" />
                      </button>
                      <button v-if="state.backgroundImage" class="btn btn-outline-secondary ms-2" @click="state.backgroundImage = undefined">
                        <i class="bi-x"></i>
                      </button>
                      <button v-if="!state.backgroundImage" class="btn btn-outline-secondary ms-2 d-none" @click="state.backgroundImage = '#888888'">
                        <i class="bi-palette"></i>
                      </button>

                    </div>
                  </div>
                </div>

              </div>
            </div>
            
            <div class="tab-pane fade" id="other" role="tabpanel" aria-labelledby="other-tab">
              <div class="card">
                <div class="card-body">
                  <div class="mb-3">
                    <label class="form-label">Pixel Opacity</label>
                    <OptionSlider v-model="state.pixelOpacity" :min="0" :max="1" :step="0.01" />
                  </div>
                  
                  <div class="mb-3">
                    <label class="form-label">Light Opacity</label>
                    <OptionSlider v-model="state.pixelLightOpacity" :min="0" :max="1" :step="0.1" />
                  </div>
                  
                  <div class="mb-3">
                    <label class="form-label">Dark Opacity</label>
                    <OptionSlider v-model="state.pixelDarkOpacity" :min="0" :max="1" :step="0.01" />
                  </div>
                  
                  <div class="mb-3">
                    <label class="form-label">Promo Text</label>
                    <input v-model="state.promoText" type="text" class="form-control">
                  </div>
                  
                  <div class="mb-3">
                    <label class="form-label">Promo Font Size</label>
                    <OptionSlider v-model="state.promoTextSize" :min="10" :max="60" :step="1" unit="px" />
                  </div>
                  
                </div>
              </div>
            </div>
          </div>
          
          <div class="d-flex gap-2">
            <button class="btn btn-outline-secondary btn-sm" @click="downloadState()">
              <i class="bi-download"></i> Save state
            </button>
            <div class="position-relative">
              <button class="btn btn-outline-secondary btn-sm">
                <i class="bi-upload"></i> Load state
              </button>
              <input type="file" accept="application/json" class="position-absolute top-0 start-0 opacity-0 w-100 h-100" @input="readState">
            </div>
            <button class="btn btn-outline-danger btn-sm ms-auto" @click="reset()">
              <i class="bi-trash"></i> Reset State
            </button>
          </div>
        </div>
      </div>
      <div class="col-lg-5" ref="rightPanelEl">
        <div class="d-flex flex-column gap-2" :class="'position-sticky'" style="top: '10px';">
          <canvas ref="canvas" class="w-100" width="1000" height="1000"></canvas>

          <button class="btn btn-primary" @click="download()">
            <i class="bi-download"></i> Download
          </button>
          <div v-if="mayNotScannable" class="alert alert-warning" role="alert">
            This QR Code may or may not be scannable. Please verify before using.
          </div>
        </div>
      </div>
    </div>
  </div>

  <DialogScan v-if="uploadQR" :model-value="true" :qrcode="uploadQR" :state="props.state" @update:model-value="uploadQR = undefined" />
</template>