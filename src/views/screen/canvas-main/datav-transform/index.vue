<template>
  <div
    class="datav-transform"
    :class="transformClass"
    :style="transformStyle"
  >
    <refer-line
      v-if="referLine.enable && com.selected"
      :attr="com.attr"
      :scale="scale"
    />
    <div
      :class="['datav-scale', { hovered: com.hovered }]"
      :style="hideStyle"
      @mouseenter="onEnter"
      @mouseleave="onLeave"
      @mousedown.prevent.stop="onMove"
    >
      <div
        class="transform-handler"
        :class="handlerClass"
        :style="handlerStyle"
      >
        <div class="datav-com" :style="hideStyle">
          <slot></slot>
          <div
            class="datav-wrapper-event-disable"
            :style="wrapperStyle"
            @contextmenu="showMenu"
          ></div>
        </div>
        <template v-for="(v, k) in points" :key="k">
          <i v-if="v.rotateStyle" :class="`${v.name}-handler`">
            <span
              class="rotate-handler"
              :style="v.rotateStyle"
              @mousedown.prevent.stop="onRotate"
            >
              <span
                class="control-point"
                :style="v.style"
                @mousedown.prevent.stop="onZoom($event, k)"
              ></span>
            </span>
          </i>
          <i v-else :class="`${v.name}-handler`">
            <span
              class="control-point"
              :style="v.style"
              @mousedown.prevent.stop="onZoom($event, k)"
            ></span>
          </i>
        </template>
        <div class="transform-bg"></div>
      </div>
    </div>
  </div>
</template>

<script lang='ts'>
import { defineComponent, PropType, computed, getCurrentInstance } from 'vue'
import { DatavComponent } from '@/components/datav-component'
import { EditorModule } from '@/store/modules/editor'
import {
  Direction, getCursors, setHover,
  handleMove, handleZoom, handleRotate,
} from './index'
import { useContextMenu } from '../../editor-context-menu/index'
import ReferLine from './refer-line.vue'

export default defineComponent({
  name: 'DatavTransform',
  components: {
    ReferLine,
  },
  props: {
    com: {
      type: Object as PropType<DatavComponent>,
      required: true,
    },
  },
  setup(props) {
    const instance = getCurrentInstance()
    const referLine = computed(() => EditorModule.referLine)
    const scale = computed(() => EditorModule.canvas.scale)

    const transformClass = computed(() => ({
      selected: props.com.selected,
      hided: props.com.hided,
      locked: props.com.locked,
    }))

    const transformStyle = computed(() => ({
      top: 0,
      left: 0,
      width: `${props.com.attr.w}px`,
      height: `${props.com.attr.h}px`,
      transform: `translate(${props.com.attr.x}px, ${props.com.attr.y}px)`,
    }))

    const hideStyle = computed(() => ({
      display: props.com.hided ? 'none' : 'block',
    }))

    const handlerClass = computed(() => ({
      hided: !props.com.selected || props.com.locked,
    }))

    const handlerStyle = computed(() => ({
      cursor: 'move',
      transform: `rotate(${props.com.attr.deg}deg)`,
    }))

    const wrapperStyle = computed(() => ({
      width: `${props.com.attr.w}px`,
      height: `${props.com.attr.h}px`,
    }))

    const cursor = computed(() => getCursors(props.com.attr.deg))

    const points = computed<{
      [k in Direction]: {
        name: string
        style: Partial<CSSStyleDeclaration>
        rotateStyle?: Partial<CSSStyleDeclaration>
      }
    }>(() => {
      const transform = `scale(${1 / scale.value}, ${1 / scale.value})`
      return {
        't': {
          name: 'top',
          style: { cursor: cursor.value.t, transform },
        },
        'rt': {
          name: 'top-right',
          style: { cursor: cursor.value.rt },
          rotateStyle: { 'transform-origin': '25% 75%',  transform },
        },
        'r': {
          name: 'right',
          style: { cursor: cursor.value.r, transform },
        },
        'rb': {
          name: 'bottom-right',
          style: { cursor: cursor.value.rb },
          rotateStyle: { 'transform-origin': '25% 25%',  transform },
        },
        'b': {
          name: 'bottom',
          style: { cursor: cursor.value.b, transform },
        },
        'lb': {
          name: 'bottom-left',
          style: { cursor: cursor.value.lb },
          rotateStyle: { 'transform-origin': '75% 25%',  transform },
        },
        'l': {
          name: 'left',
          style: { cursor: cursor.value.l, transform },
        },
        'lt': {
          name: 'top-left',
          style: { cursor: cursor.value.lt },
          rotateStyle: { 'transform-origin': '75% 75%',  transform },
        },
      }
    })

    const selectCom = () => {
      if (props.com.selected) {
        return
      }

      EditorModule.selectCom(props.com.id)
    }

    const onEnter = () => {
      setHover(props.com, true)
    }

    const onLeave = () => {
      setHover(props.com, false)
    }

    const onMove = (ev: MouseEvent) => {
      selectCom()
      handleMove(ev, props.com, scale.value, EditorModule.pageConfig.grid)

    }

    const onZoom = (ev: MouseEvent, dir: Direction) => {
      selectCom()
      handleZoom(ev, dir, props.com, scale.value)
    }

    const onRotate = (ev: MouseEvent) => {
      handleRotate(ev, instance!.vnode.el as HTMLElement, props.com)
    }

    const { showMenu } = useContextMenu()

    return {
      referLine,
      scale,
      transformClass,
      transformStyle,
      hideStyle,
      handlerClass,
      handlerStyle,
      wrapperStyle,
      points,
      onEnter,
      onLeave,
      onMove,
      onZoom,
      onRotate,
      showMenu,
    }
  },
})
</script>

<style lang="scss" scoped>
@import './style';

</style>
