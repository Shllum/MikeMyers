<template>
  <widget :config="config" move-handles="none" readonly :removable="false">
    <div class="relative overflow-hidden"
      :style="{
        width: `${anchor.width}px`,
        height: `${anchor.height}px`,
        background: 'rgba(255,255,255,0.1)'
      }">
      <div
        class="absolute"
        :style="{
          top: `${(anchor.height - anchor.cell * 9) / 2}px`,
          left: `${(anchor.width - anchor.cell * 11) / 2}px`
        }">
          <div v-for="i in 9" :key="i" class="flex">
            <div v-for="j in 11" :key="j"
              class="border border-gray-600"
              :style="cellSize" />
          </div>
        </div>
    </div>
  </widget>
</template>

<script lang="ts">
import { computed, defineComponent, inject, PropType } from 'vue'
import Widget from './Widget.vue'
import { MainProcess } from '@/web/background/IPC'
import { Widget as IWidget, WidgetManager } from './interfaces'

export default defineComponent({
  components: { Widget },
  props: {
    config: {
      type: Object as PropType<IWidget>,
      required: true
    }
  },
  setup (props) {
    const wm = inject<WidgetManager>('wm')!

    MainProcess.onEvent('MAIN->OVERLAY::delve-grid', () => {
      if (props.config.wmWants === 'hide') {
        wm.show(props.config.wmId)
      } else {
        wm.hide(props.config.wmId)
      }
    })

    const anchor = computed(() => {
      const wmHeight = wm.size.value.height

      const height = Math.round(wmHeight * 808 / 1080)
      const width = Math.round(height * 1030 / 808)
      const top = Math.round(wmHeight * 67 / 1080)
      const cell = Math.round(wmHeight * 97 / 1080)

      return {
        pos: 'tc',
        y: (top / wmHeight) * 100,
        x: 50,
        height,
        width,
        cell
      }
    })

    const cellSize = computed(() => {
      return {
        width: `${anchor.value.cell}px`,
        height: `${anchor.value.cell}px`
      }
    })

    return {
      anchor,
      config: computed(() => ({ ...props.config, anchor: anchor.value })),
      cellSize
    }
  }
})
</script>
