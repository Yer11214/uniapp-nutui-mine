<script setup lang="ts">
import { type ComponentInternalInstance, computed, defineComponent, getCurrentInstance, inject, onMounted, reactive, ref, watch } from 'vue'
import { PREFIX } from '../_utils'
import { useRect, useTouch } from '../_hooks'
import { type SwipePosition, swipeEmits, swipeProps } from './swipe'

const props = defineProps(swipeProps)
const emit = defineEmits(swipeEmits)

const instance = getCurrentInstance() as ComponentInternalInstance
const refRandomId = Math.random().toString(36).slice(-8)
const lockClick = ref(false)

async function getRefWidth(elementId: string) {
  const rect = await useRect(elementId, instance)
  return rect.width || 0
}

const leftRefId = `leftRef-${refRandomId}`
const leftRefWidth = ref(0)
const rightRefId = `rightRef-${refRandomId}`
const rightRefWidth = ref(0)

async function initWidth() {
  leftRefWidth.value = await getRefWidth(leftRefId)
  rightRefWidth.value = await getRefWidth(rightRefId)
}

const parent = inject('swipeGroup', null) as any

watch(
  () => parent?.name?.value,
  (name) => {
    if (props.name !== name && parent && parent.lock)
      close()
  },
)

onMounted(() => {
  setTimeout(() => {
    initWidth()
  }, 100)
})

const opened = ref(false)

let position: SwipePosition = ''
let oldPosition: SwipePosition = ''

const state = reactive({
  offset: 0,
  moving: false,
})

function open(p: SwipePosition = '') {
  parent && parent.update(props.name)
  opened.value = true
  if (p)
    state.offset = p === 'left' ? -rightRefWidth.value : leftRefWidth.value

  emit('open', {
    name: props.name,
    position: position || p,
  })
}

function close() {
  state.offset = 0
  if (opened.value) {
    opened.value = false
    emit('close', {
      name: props.name,
      position,
    })
  }
}

function onClick(e: Event, position: string, lock: boolean) {
  if (lock)
    e.stopPropagation()
  else
    close()

  emit('click', position)
}

const touchStyle = computed(() => {
  return {
    transform: `translate3d(${state.offset}px, 0, 0)`,
  }
})

function setoffset(deltaX: number) {
  position = deltaX > 0 ? 'right' : 'left'
  let offset = deltaX
  switch (position) {
    case 'left':
      if (opened.value && oldPosition === position)
        offset = -rightRefWidth.value

      else
        offset = Math.abs(deltaX) > rightRefWidth.value ? -rightRefWidth.value : deltaX

      break
    case 'right':
      if (opened.value && oldPosition === position)
        offset = leftRefWidth.value

      else
        offset = Math.abs(deltaX) > leftRefWidth.value ? leftRefWidth.value : deltaX

      break
  }
  state.offset = offset
}

const touch = useTouch()
const touchMethods = {
  onTouchStart(event: Event) {
    if (props.disabled)
      return
    touch.start(event)
  },
  async onTouchMove(event: Event) {
    if (props.disabled)
      return
    touch.move(event)
    if (touch.isHorizontal()) {
      lockClick.value = true
      state.moving = true
      setoffset(touch.deltaX.value)
      if (props.touchMovePreventDefault)
        event.preventDefault()

      if (props.touchMoveStopPropagation)
        event.stopPropagation()
    }
  },
  onTouchEnd() {
    if (state.moving) {
      state.moving = false
      oldPosition = position
      switch (position) {
        case 'left':
          if (Math.abs(state.offset) <= rightRefWidth.value / 2) {
            close()
          }
          else {
            state.offset = -rightRefWidth.value
            open()
          }
          break
        case 'right':
          if (Math.abs(state.offset) <= leftRefWidth.value / 2) {
            close()
          }
          else {
            state.offset = leftRefWidth.value
            open()
          }
          break
      }
      setTimeout(() => {
        lockClick.value = false
      }, 0)
    }
  },
}
defineExpose({
  open, close,
})
</script>

<script lang="ts">
const componentName = `${PREFIX}-swipe`
export default defineComponent({
  name: componentName,
  options: {
    virtualHost: true,
    addGlobalClass: true,
    // #ifndef H5
    styleIsolation: 'shared',
    // #endif
  },
})
</script>

<template>
  <view
    class="nut-swipe" :style="touchStyle" @touchstart="touchMethods.onTouchStart" @touchmove="touchMethods.onTouchMove" @touchend="touchMethods.onTouchEnd"
    @touchcancel="touchMethods.onTouchEnd"
  >
    <view :id="leftRefId" class="nut-swipe__left" @click="onClick($event, 'left', true)">
      <slot name="left" />
    </view>

    <view class="nut-swipe__content" @click="onClick($event, 'content', lockClick)">
      <slot name="default" />
    </view>

    <view :id="rightRefId" class="nut-swipe__right" @click="onClick($event, 'right', true)">
      <slot name="right" />
    </view>
  </view>
</template>

<style lang="scss">
@import './index';
</style>
