<script setup lang="ts">
import { randomStr } from '@antfu/utils'
import { generateQRCode } from '../logic/generate'
import { defaultGeneratorState } from '../logic/state'
import type { MarginObject, QRCodeGeneratorState } from '../logic/types'
import { MarkerInnerShapes, MarkerShapes, PixelStyles } from '../logic/types'
import {ref} from 'vue';
import OptionItem from 'Robin-Swailes-Design-and-Development-QR/components/OptionItem.vue' //VTooltip
import OptionSlider from 'Robin-Swailes-Design-and-Development-QR/components/OptionSlider.vue'
import OptionCheckbox from 'Robin-Swailes-Design-and-Development-QR/components/OptionCheckbox.vue'
import OptionColor from 'Robin-Swailes-Design-and-Development-QR/components/OptionColor.vue'
import OptionSelectGroup from 'Robin-Swailes-Design-and-Development-QR/components/OptionSelectGroup.vue'
import SettingsMarkerStyle from 'Robin-Swailes-Design-and-Development-QR/components/SettingsMarkerStyle.vue'
import SettingsMargin from 'Robin-Swailes-Design-and-Development-QR/components/SettingsMargin.vue'
import SettingsRandomRange from 'Robin-Swailes-Design-and-Development-QR/components/SettingsRandomRange.vue'
import ImageUpload from 'Robin-Swailes-Design-and-Development-QR/components/ImageUpload.vue'

import { useLocalStorage, useEventListener} from '@vueuse/core';

interface BatchState {
  filenamePrefix: string
  amount: number
  stringLength: number
  margin: number | MarginObject
}

const canvas = ref<HTMLCanvasElement>()

const options = useLocalStorage<BatchState>('qrt-batch', {
  filenamePrefix: 'batch-',
  amount: 10,
  stringLength: 10,
  margin: 2,
}, {
  mergeDefaults: true,
})

function randomRange(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1) + min)
}

function randomSelect<T>(arr: readonly T[]) {
  return arr[randomRange(0, arr.length - 1)]
}

async function download() {
  const c = canvas.value!
  const a = document.createElement('a')

  for (let i = 0; i < options.value.amount; i++) {
    const text = randomStr(options.value.stringLength)
    const state: QRCodeGeneratorState = {
      ...defaultGeneratorState(),
      text,
      margin: options.value.margin,
      pixelStyle: randomSelect(PixelStyles),
      markerShape: randomSelect(MarkerShapes),
      markerInnerShape: randomSelect(MarkerInnerShapes),
      maskPattern: randomRange(-1, 7),
      effect: Math.random() > 0.5 ? 'none' : 'crystalize',
      marginNoise: Math.random() > 0.5,
      marginNoiseRate: Math.random() * 0.5 + 0.1,
    }

    await generateQRCode(c, state)
    a.href = c.toDataURL('image/png')
    a.download = `${options.value.filenamePrefix}${i}-${c.width}x${c.height}-${text}.distorted.png`
    a.click()

    await new Promise(resolve => setTimeout(resolve, 50))

    state.effect = 'none'
    state.pixelStyle = 'square'
    state.markerShape = 'square'
    state.markerInnerShape = 'square'
    await generateQRCode(c, state)
    a.href = c.toDataURL('image/png')
    a.download = `${options.value.filenamePrefix}${i}-${c.width}x${c.height}-${text}.normal.png`
    a.click()

    await new Promise(resolve => setTimeout(resolve, 50))
  }
}
</script>
<template>
  <div class="justify-center  p-10 pb-15 md:flex">
    <div class="flex flex-col gap-4 min-h-[calc(100vh-100px)] w-250">
      <p>
        This page is to generate a batch of QR codes with random settings, for training purposes.
      </p>
      <div class="border border-base rounded p-4">
        <canvas ref="canvas" />
      </div>
      <div class="flex flex-col gap-2 border border-base rounded p-4">
        <OptionItem title="Filename Prefix">
          <input
            v-model="options.filenamePrefix"
            type="text"
            class="border border-base rounded w-auto bg-secondary pl-2"
          >
        </OptionItem>
        <OptionItem title="Amount">
          <OptionSlider
            v-model="options.amount"
            :min="1"
            :max="1000"
            :step="1"
          />
        </OptionItem>
        <OptionItem title="Content Length">
          <OptionSlider
            v-model="options.stringLength"
            :min="1"
            :max="100"
            :step="1"
          />
        </OptionItem>
        <SettingsMargin v-model="options.margin" />
      </div>
      <div class="text-button" @click="download">
        Generate
      </div>
    </div>
  </div>
</template>
