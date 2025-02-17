<template>
  <div class="canvas-container" :style="CanvasContainerStyle" :class="canvasContainerClass">
    <refer-line v-if="component.selected" :attr="component.attr" :scale="scale" />
    <div
      :class="['datav-scale', { hovered: isHover }]"
      :style="hideStyle"
      ref="containScaleRef"
      @mousedown.prevent.stop="onMove"
    >
      <div class="transform-handler" :class="handlerClass" :style="handlerStyle">
        <div class="datav-com">
          <slot></slot>
          <div
            class="datav-wrapper-event-disable"
            :style="wrapperStyle"
            @contextmenu="showMenu"
          ></div>
        </div>
        <ControlPoint
          :component="component"
          :scale="scale"
          :selectCom="selectCom"
          :instance="instance"
        />
        <div class="transform-bg"></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { useEditorComStore } from '@/store/modules/editorCom'
  import { useHover } from 'vue3-hooks-plus'
  import { handleMove } from './utils'
  import ControlPoint from './ControlPoint.vue'
  import ReferLine from './refer-line.vue'
  import { useContextMenu } from '@/hooks/useContextMenu'
  import { AbstractvComponent } from '@/components/componentFactory'

  const editorComStore = useEditorComStore()
  const { showMenu } = useContextMenu()
  const instance = getCurrentInstance()
  const props = defineProps<{ component: AbstractvComponent }>()

  const containScaleRef = ref()
  const mouseIsHover = useHover(containScaleRef)

  watch(mouseIsHover, (b) => {
    editorComStore.setComponentHover(b, props.component.id)
  })

  const isHover = computed(() => props.component.hovered)

  const scale = computed(() => editorComStore.getCanvasScale)

  const CanvasContainerStyle = computed(() => ({
    top: `${props.component.attr.y}px`,
    left: `${props.component.attr.x}px`,
    width: `${props.component.attr.w}px`,
    height: `${props.component.attr.h}px`,
    opacity: props.component.attr.opacity,
    // transform: `matrix(${scale.value * Math.cos(props.component.attr.deg)},${
    //   scale.value * Math.sin(props.component.attr.deg)
    // },${-1 * scale.value * Math.sin(props.component.attr.deg)},${
    //   scale.value * Math.cos(props.component.attr.deg)
    // },${props.component.attr.x},${props.component.attr.y})`,
  }))

  console.log(props.component)

  const canvasContainerClass = computed(() => ({
    selected: props.component.selected,
    hided: props.component.hided,
    locked: props.component.locked,
  }))
  const hideStyle = computed(() => ({
    display: props.component.hided ? 'none' : 'block',
  }))
  const handlerClass = computed(() => ({
    hided: !props.component.selected || props.component.locked,
  }))
  const handlerStyle = computed(() => ({
    // cursor: 'move',
    // transform: `rotate(${props.component.attr.deg}deg)`,
  }))
  const wrapperStyle = computed(() => ({
    width: `${props.component.attr.w}px`,
    height: `${props.component.attr.h}px`,
  }))

  const selectCom = () => {
    if (props.component.selected) {
      return
    }
    editorComStore.selectComponentActive(props.component.id)
  }

  const onMove = (e: MouseEvent) => {
    handleMove({
      component: props.component,
      mouseStartEvent: e,
      grid: 8,
      scale: scale.value,
      calcAlignLine: editorComStore.calcAlignLine,
      hideAlignLine: editorComStore.hideAlignLine,
    })
    selectCom()
  }
</script>
<style lang="scss" scoped>
  @import './style.scss';
</style>
