<script setup lang="ts">
import type { ComponentInternalInstance } from 'vue'
import { computed, defineComponent, getCurrentInstance, onMounted, reactive, ref, useSlots, watch } from 'vue'
import { PREFIX } from '../_utils'
import { barrageEmits, barrageProps } from './barrage'

const props = defineProps(barrageProps)

const emit = defineEmits(barrageEmits)
const instance = getCurrentInstance() as ComponentInternalInstance

defineExpose({ add })
const classTime = new Date().getTime()

const slotDefault = !!useSlots().default

const timeId = ref(new Date().getTime())
const danmuList = ref<any>(props.danmu)
const danmuListSlots = ref<any>([])
const rows = ref<number>(props.rows)
const top = ref<number>(props.top)
const speeds = props.speeds

const classes = computed(() => {
  const prefixCls = componentName
  return {
    [prefixCls]: true,
    [`nut-barrage--dmBody${timeId.value}`]: true,
  }
})

onMounted(() => {
  if (slotDefault) {
    const list = document
      .getElementsByClassName(`nut-barrage__slotBody${classTime}`)[0]
      .getElementsByClassName('nut-barrage__item')
    const childrens = list?.[0]?.children || []
    danmuList.value = childrens
  }
  runStep()
})

watch(
  () => props.danmu,
  (newValue) => {
    danmuList.value = [...newValue]
  },
)

function add(word: string) {
  danmuList.value = [...danmuList.value, word]
  runStep()
}

function getNode(index: number) {
  const query = uni.createSelectorQuery().in(instance)
  setTimeout(() => {
    let width = 100
    query.select(`.nut-barrage--dmBody${timeId.value}`).boundingClientRect((rec: any) => {
      width = rec?.width || 300
    })
    query
      .select(`.nut-barrage__item${index}`)
      .boundingClientRect((recs: any) => {
        const height = recs?.height
        const nodeTop = `${(index % rows.value) * (height + top.value) + 20}px`
        styleInfo(index, nodeTop, width)
      })
      .exec()
  }, 500)
}

function runStep() {
  danmuList.value.forEach((item: any, index: number) => {
    getNode(index)
  })
}
// const distance = ref('0');
const styleList: any[] = reactive([])
function styleInfo(index: number, nodeTop: string, width: number) {
  const timeIndex = index - rows.value > 0 ? index - rows.value : 0
  const list = styleList
  const time = list[timeIndex] ? Number(list[timeIndex]['--time']) : 0
  // distance.value = '-' + (speeds / 1000) * 200 + '%';

  const obj = {
    'top': nodeTop,
    '--time': `${props.frequency * index + time}`,
    'animationDuration': `${speeds}ms`,
    'animationIterationCount': `${props.loop ? 'infinite' : 1}`,
    'animationDelay': `${props.frequency * index + time}ms`,
    '--move-distance': `-${width}px`,
  }
  if (slotDefault && danmuList.value[index]?.el) {
    const orginalSty = danmuList.value[index].el.style
    danmuList.value[index].el.style = Object.assign(orginalSty, obj)
  }
  else {
    styleList.push(obj)
  }
}
</script>

<script lang="ts">
const componentName = `${PREFIX}-barrage`

export default defineComponent({
  name: componentName,
  options: {
    virtualHost: true,
    addGlobalClass: true,
    styleIsolation: 'shared',
  },
})
</script>

<template>
  <view :class="classes">
    <div>
      <div :class="[`nut-barrage__slotBody${classTime}`]">
        <view
          v-for="(item, index) of danmuList"
          :key="`danmu${index}`"
          class="nut-barrage__item move" :class="[`nut-barrage__item${index}`]"
          :style="styleList[index]"
        >
          {{ item.length > 8 ? `${item.substr(0, 8)}...` : item }}
        </view>
      </div>
    </div>
  </view>
</template>

<style lang="scss">
@import './index';
</style>
