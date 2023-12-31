<script setup lang="ts">
import { computed, defineComponent, ref, useSlots, watch } from 'vue'
import { PREFIX } from '../_utils'
import type { CalendarInst } from '../calendaritem/type'
import NutPoPUp from '../popup/popup.vue'
import NutCalendarItem from '../calendaritem/calendaritem.vue'
import { calendarEmits, calendarProps } from './calendar'

const props = defineProps(calendarProps)
const emit = defineEmits(calendarEmits)
const slots = useSlots()
defineExpose({
  scrollToDate,
  initPosition,
})
const showTopBtn = computed(() => {
  return slots.btn
})
const topInfo = computed(() => {
  return slots['top-info']
})
const dayInfo = computed(() => {
  return slots.day
})
const bottomInfo = computed(() => {
  return slots['bottom-info']
})
const show = ref(props.visible)
// element refs
const calendarRef = ref<null | CalendarInst>(null)
function scrollToDate(date: string) {
  calendarRef.value?.scrollToDate(date)
}
function initPosition() {
  calendarRef.value?.initPosition()
}

// methods
function update() {
  show.value = false
  emit('update:visible', false)
}

function close() {
  show.value = false
  emit('close')
  emit('update:visible', false)
}

function choose(param: string) {
  close()
  emit('choose', param)
}

function closePopup() {
  close()
}
function select(param: string) {
  // close();
  emit('select', param)
}
watch(
  () => props.visible,
  (value: boolean) => {
    show.value = value
  },
)
</script>

<script lang="ts">
const componentName = `${PREFIX}-calendar`
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
  <NutPoPUp
    v-if="poppable"
    v-bind="props"
    v-model:visible="show"
    position="bottom"
    round
    closeable
    :custom-style="{ height: '85vh' }"
    @click-overlay="closePopup"
    @click-close-icon="closePopup"
  >
    <NutCalendarItem
      v-if="show"
      ref="calendarRef"
      :type="type"
      :is-auto-back-fill="isAutoBackFill"
      :poppable="poppable"
      :title="title"
      :default-value="defaultValue"
      :start-date="startDate"
      :end-date="endDate"
      :confirm-text="confirmText"
      :start-text="startText"
      :end-text="endText"
      :show-today="showToday"
      :show-title="showTitle"
      :show-sub-title="showSubTitle"
      :to-date-animation="toDateAnimation"
      :first-day-of-week="firstDayOfWeek"
      @update="update"
      @close="close"
      @choose="choose"
      @select="select"
    >
      <template v-if="showTopBtn" #btn>
        <slot name="btn" />
      </template>
      <template v-if="dayInfo" #day="date">
        <slot name="day" :date="date.date" />
      </template>
      <template v-if="topInfo" #top-info="date">
        <slot name="top-info" :date="date.date" />
      </template>
      <template v-if="bottomInfo" #bottom-info="date">
        <slot name="bottom-info" :date="date.date" />
      </template>
    </NutCalendarItem>
  </NutPoPUp>
  <NutCalendarItem
    v-else
    :type="type"
    :is-auto-back-fill="isAutoBackFill"
    :poppable="poppable"
    :title="title"
    :confirm-text="confirmText"
    :start-text="startText"
    :end-text="endText"
    :default-value="defaultValue"
    :start-date="startDate"
    :end-date="endDate"
    :show-title="showTitle"
    :show-sub-title="showSubTitle"
    :to-date-animation="toDateAnimation"
    :show-today="showToday"
    :first-day-of-week="firstDayOfWeek"
    @close="close"
    @choose="choose"
    @select="select"
  >
    <template v-if="showTopBtn" #btn>
      <slot name="btn" />
    </template>
    <template v-if="dayInfo" #day="date">
      <slot name="day" :date="date.date" />
    </template>
    <template v-if="topInfo" #top-info="date">
      <slot name="top-info" :date="date.date" />
    </template>
    <template v-if="bottomInfo" #bottom-info="date">
      <slot name="bottom-info" :date="date.date" />
    </template>
  </NutCalendarItem>
</template>

<style lang="scss">
@import './index';
</style>
