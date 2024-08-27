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
  if (confirm('Are you sure to reset all state?'))
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

const debouncedRun = debounce(run, 250, { trailing: true })

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

const uploadQR = ref<string>()

const { isOverDropZone } = useDropZone(document.body, {
  onDrop(files) {
    if (view.value !== 'generator')
      return
    if (!files || !uploadTarget.value)
      return

    const file = files[0]
    if (file.type === 'image/png' || file.type === 'image/jpeg') {
      const reader = new FileReader()
      reader.onload = () => {
        const data = reader.result as string
        if (uploadTarget.value === 'qrcode')
          uploadQR.value = data
      }
      reader.readAsDataURL(file)
    }
  },
  onLeave() {
    uploadTarget.value = undefined
  },
  onOver(_, event) {
    if (uploadQR.value)
      uploadQR.value = undefined
    if (view.value !== 'generator')
      return
    if (!isOverDropZone.value)
      return

    const chain = Array.from(document.elementsFromPoint(event.clientX, event.clientY))
    if (chain.find(el => el.id === 'upload-zone-qrcode'))
      uploadTarget.value = 'qrcode'
    else
      uploadTarget.value = undefined
  },
})

watch(
  () => state.value,
  () => debouncedRun(),
  { deep: true, immediate: true },
)
</script>




<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-lg-5">
        <div class="d-flex flex-column gap-2">
          <textarea v-model="state.text" placeholder="Text to encode" class="form-control"></textarea>
          <div class="card">
            <div class="card-body">
              <div class="mb-3">
                <label class="form-label">Error Correction</label>
                <div class="d-flex align-items-center">
                  <OptionSelectGroup v-model="state.ecc" :options="['L', 'M', 'Q', 'H']" class="me-2" />
                  <div class="form-check">
                    <input class="form-check-input" type="checkbox" v-model="state.boostECC" id="boostECC">
                    <label class="form-check-label" for="boostECC">Boost ECC</label>
                  </div>
                </div>
              </div>

              <div class="mb-3">
                <label class="form-label">Mask Pattern</label>
                <OptionSelectGroup v-model="state.maskPattern" :options="[-1, 0, 1, 2, 3, 4, 5, 6, 7]" :titles="['Auto']" />
              </div>

              <div class="mb-3">
                <label class="form-label">Rotate</label>
                <OptionSelectGroup v-model="state.rotate" :options="[0, 90, 180, 270]" :titles="['0°', '90°', '180°', '270°']" />
              </div>

              <hr>

              <div class="mb-3">
                <label class="form-label">Pixel Style</label>
                <OptionSelectGroup v-model="state.pixelStyle" :options="PixelStyles" :classes="PixelStyleIcons" />
              </div>

              <div class="mb-3">
                <label class="form-label">{{ state.markers.length ? 'Marker 1' : 'Markers' }}</label>
                <button class="btn btn-outline-secondary btn-sm float-end" @click="toggleMarkerStyleExpand">
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
                <hr>
              </template>

              <div v-if="qrcode?.version !== 1" class="mb-3">
                <label class="form-label">Sub Markers</label>
                <OptionSelectGroup v-model="state.markerSub" :options="MarkerSubShapes" :classes="MarkerSubShapeIcons" />
              </div>

              <hr>

              <SettingsMargin v-model="state.margin" :full-customizable="true" />

              <div class="mb-3 d-none">
                <label class="form-label">Margin Noise</label>
                <div class="form-check">
                  <input class="form-check-input" type="checkbox" v-model="state.marginNoise" id="marginNoise">
                  <label class="form-check-label" for="marginNoise">Add some random data points to the margin</label>
                </div>
              </div>

              <template v-if="state.marginNoise">
                <div class="mb-3 d-none">
                  <label class="form-label">Noise Rate</label>
                  <OptionSlider v-model="state.marginNoiseRate" :min="0" :max="1" :step="0.01" />
                </div>

                <SettingsRandomRange v-model="state.marginNoiseOpacity" title="Opacity" :min="0" :max="1" :step="0.01" />
              </template>

              <div class="mb-3 d-none">
                <label class="form-label">Safe Space</label>
                <OptionSelectGroup v-model="state.marginNoiseSpace" :options="['full', 'marker', 'minimal', 'extreme', 'none']" />
              </div>

              <div class="mb-3 d-none">
                <label class="form-label">Render Type</label>
                <OptionSelectGroup v-model="state.renderPointsType" :options="['all', 'function', 'data', 'guide', 'marker']" />
              </div>

              <div class="mb-3 d-none">
                <label class="form-label">Seed</label>
                <div class="input-group">
                  <input v-model.number="state.seed" type="number" class="form-control">
                  <button class="btn btn-outline-secondary" @click="state.seed = Math.round(Math.random() * 100000)">
                    <i class="bi-arrow-clockwise"></i>
                  </button>
                </div>
              </div>

              <div class="mb-3">
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
                  <button v-if="!state.backgroundImage" class="btn btn-outline-secondary ms-2" @click="state.backgroundImage = '#888888'">
                    <i class="bi-palette"></i>
                  </button>
                </div>
              </div>

              <div class="mb-3">
                <label class="form-label">Pixel Opacity</label>
                <OptionSlider v-model="state.pixelOpacity" :min="0" :max="1" :step="0.01" />
              </div>

              <div class="mb-3">
                <label class="form-label">Light Opacity</label>
                <OptionSlider v-model="state.pixelLightOpacity" :min="0" :max="1" :step="0.01" />
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

              <div class="mb-3">
                <label class="form-label">Logo</label>
                <div class="d-flex align-items-center">
                  <OptionColor v-if="state.logoImage?.startsWith('#')" v-model="state.logoImage" />
                  <button v-else class="btn btn-outline-secondary position-relative">
                    <img v-if="state.logoImage" :src="state.logoImage" class="top-0 start-0 w-100 h-100 rounded opacity-50">
                    <i class="bi-upload"></i> Upload
                    <ImageUpload v-model="state.logoImage" />
                  </button>
                  <button v-if="state.logoImage" class="btn btn-outline-secondary ms-2" @click="state.logoImage = undefined">
                    <i class="bi-x"></i>
                  </button>
                </div>
              </div>

              <div class="mb-3">
                <label class="form-label">Logo Scale</label>
                <OptionSlider v-model="state.logoScale" :min="0.2" :max="0.6" :step="0.01" unit="%" />
              </div>

              <hr>

              <div class="mb-3">
                <label class="form-label">Colors</label>
                <div class="d-flex align-items-center">
                  <OptionColor v-model="state.lightColor" class="me-2" />
                  <OptionColor v-model="state.darkColor" class="me-2" />
                  <div class="form-check">
                    <input class="form-check-input" type="checkbox" v-model="state.invert" id="invertColors">
                    <label class="form-check-label" for="invertColors">Invert</label>
                  </div>
                </div>
              </div>

              <hr>

              <div class="mb-3 d-none">
                <label class="form-label">Min Version</label>
                <OptionSlider v-model="state.minVersion" :min="1" :max="state.maxVersion" :step="1" />
              </div>

              <div class="mb-3  d-none">
                <label class="form-label">Max Version</label>
                <OptionSlider v-model="state.maxVersion" :min="state.minVersion" :max="40" :step="1" />
              </div>

              <div class="mb-3">
                <label class="form-label">Pixel Size</label>
                <OptionSlider v-model="state.scale" :min="1" :max="50" :step="1" unit="px" />
              </div>

              <div class="mb-3">
                <label class="form-label">Pixel Scale (smaller pixel fill)</label>
                <OptionSlider v-model="state.dotScale" :min="0.5" :max="1.1" :step="0.01" unit="%" />
              </div>

              <hr>

              <div class="mb-3  d-none">
                <label class="form-label">Effect</label>
                <OptionSelectGroup v-model="state.effect" :options="['none', 'crystalize', 'liquidify']" />
              </div>

              <template v-if="state.effect === 'crystalize'">
                <div class="mb-3  d-none">
                  <label class="form-label">Radius</label>
                  <OptionSlider v-model="state.effectCrystalizeRadius" :min="1" :max="20" :step="0.5" />
                </div>
              </template>

              <template v-if="state.effect === 'liquidify'">
                <div class="mb-3  d-none">
                  <label class="form-label">Distort Radius</label>
                  <OptionSlider v-model="state.effectLiquidifyDistortRadius" :min="1" :max="40" :step="1" />
                </div>
                <div class="mb-3  d-none">
                  <label class="form-label">Blur Radius</label>
                  <OptionSlider v-model="state.effectLiquidifyRadius" :min="1" :max="40" :step="1" />
                </div>
                <div class="mb-3  d-none">
                  <label class="form-label">Threshold</label>
                  <OptionSlider v-model="state.effectLiquidifyThreshold" :min="1" :max="254" :step="1" unit="/256" />
                </div>
              </template>

              <template v-if="state.effect !== 'none'">
                <div class="mb-3  d-none">
                  <label class="form-label">Effect Timing</label>
                  <OptionSelectGroup v-model="state.effectTiming" :options="['before', 'after']" />
                </div>
              </template>

              <hr>

              <div class="mb-3">
                <label class="form-label">Transform</label>
              </div>
              <div class="mb-3">
                <label class="form-label">Perspective X</label>
                <OptionSlider v-model="state.transformPerspectiveX" :min="-0.5" :max="0.5" :step="0.01" :default="0" />
              </div>
              <div class="mb-3">
                <label class="form-label">Perspective Y</label>
                <OptionSlider v-model="state.transformPerspectiveY" :min="-0.5" :max="0.5" :step="0.01" :default="0" />
              </div>
              <div class="mb-3">
                <label class="form-label">Scale</label>
                <OptionSlider v-model="state.transformScale" :min="0.5" :max="2" :step="0.01" :default="1" />
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
      <div class="col-lg-7" ref="rightPanelEl">
        <div class="d-flex flex-column gap-2" :class="{ 'position-fixed': floating }" :style="floating ? { top: '10px', left: `${rightPanelRect.left}px`, width: '20vw' } : {}">
          <canvas ref="canvas" class="w-100" width="1000" height="1000"></canvas>

          <div v-if="qrcode" class="card d-none">
            <div class="card-body">
              <div class="row g-2">
                <div class="col-4">
<small class="text-muted">Size</small>
                  <div>{{ qrcode.size }}</div>
                </div>
                <div class="col-4">
                  <small class="text-muted">Mask</small>
                  <div>{{ qrcode.maskPattern }}</div>
                </div>
                <div class="col-4">
                  <small class="text-muted">Version</small>
                  <div>{{ qrcode.version }}</div>
                </div>
              </div>
              <div v-if="generateQRCodeInfo" class="row g-2 mt-2">
                <div class="col-6">
                  <small class="text-muted">Dimension</small>
                  <div>{{ generateQRCodeInfo.width }} x {{ generateQRCodeInfo.height }}</div>
                </div>
                <div class="col-6">
                  <small class="text-muted">Aspect</small>
                  <div>{{ getAspectRatio(generateQRCodeInfo.width, generateQRCodeInfo.height) }}</div>
                </div>
              </div>
            </div>
          </div>
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

  <div v-if="isOverDropZone" class="position-fixed top-0 start-0 end-0 bottom-0 d-flex align-items-center justify-content-center bg-black bg-opacity-25">
    <div id="upload-zone-qrcode" class="d-flex flex-column align-items-center justify-content-center p-5 bg-light bg-opacity-75 border border-3 border-dashed rounded-3" :class="{ 'border-primary': uploadTarget === 'qrcode' }">
      <i class="bi-qr-code display-1"></i>
      <div class="fs-4">Scan QR Code</div>
    </div>
  </div>

  <DialogScan v-if="uploadQR" :model-value="true" :qrcode="uploadQR" :state="props.state" @update:model-value="uploadQR = undefined" />
</template>