<template>
  <section
    :style="{
      gridArea,
      background: grayed ? '#e8e8e8' : '#ffffff',
    }"
    :class="{
      lastcol: section.col.start === colsNumber && section.row.start === 1,
      lastrow: section.row.start === rowsNumber && section.col.start === 1,
      implicit:
        section.row.start < 1 ||
        section.row.end > rowsNumber + 1 ||
        section.col.start < 1 ||
        section.col.end > colsNumber + 1,
      dragging,
      grayed,
      focused,
    }"
    class="grid-section"
  >
    <div
      v-if="!(selection && selection.parent !== area && selection.fresh)"
      :class="['grid-cell', { 'selection-dragging': selection && selection.fresh }]"
      :data-col-start="section.col.start"
      :data-row-start="section.row.start"
      :data-col-end="section.col.end"
      :data-row-end="section.row.end"
      @pointerdown="$emit('pointerdown', $event)"
      @mouseover="$emit('overcell', { col: section.col.start, row: section.row.start })"
    ></div>
  </section>
</template>

<script setup>
import { useAppState, parseValue, pause, resume } from '../../store.js'
import { useGridDimensions } from '../../composables/area.js'
import { asValidGridArea } from '../../utils/grid.js'

let { dragging, selection } = $(useAppState())

const props = defineProps({
  section: { type: Object, required: true },
  area: { type: Object, required: true },
  grayed: { type: Boolean, default: false },
  focused: { type: Boolean, default: false },
  implicitGrid: { type: Object, required: true },
})
defineEmits(['pointerdown', 'overcell'])

let grid = $computed(() => props.area.grid)

let gridArea = $computed(() => {
  const s = props.section
  return asValidGridArea(s.row.start, s.col.start, s.row.end, s.col.end, props.implicitGrid)
})

let { colsNumber, rowsNumber } = $(useGridDimensions($$(grid)))

let isDraggingGrid = $computed(() => dragging && dragging.grid === grid)

let isDraggingSection = $computed(
  () => isDraggingGrid && (dragging.colLine === props.section.col.start || dragging.rowLine === props.section.row.start)
)

let isDraggingCol = $computed(() => isDraggingGrid && dragging.colLine === props.section.col.start)

let isDraggingRow = $computed(() => isDraggingGrid && dragging.rowLine === props.section.row.start)

let computePadding = (condition, gap) => (condition ? '7px' : parseValue(gap) !== 0 ? `calc(-0.5 * ${gap})` : '0')

let paddingTop = $computed(() =>
  computePadding(props.section.row.start === props.implicitGrid.ri, props.area.grid.row.gap)
)
let paddingLeft = $computed(() =>
  computePadding(props.section.col.start === props.implicitGrid.ci, props.area.grid.col.gap)
)
let paddingBottom = $computed(() =>
  computePadding(props.section.row.end === props.implicitGrid.rows + props.implicitGrid.ri, props.area.grid.row.gap)
)
let paddingRight = $computed(() =>
  computePadding(props.section.col.end === props.implicitGrid.cols + props.implicitGrid.ci, props.area.grid.col.gap)
)
</script>

<style scoped lang="postcss">
section {
  touch-action: none;
  pointer-events: none;
  height: 100%;
  position: relative;
  &:not(.dragging) {
    cursor: pointer;
  }
  /*&.grayed {
    background: #ddddddcc;
  }*/
  &:not(.dragging):hover {
    background: var(--color-green-hover) !important;
  }
  &.focused {
    background: var(--color-green-hover) !important;
  }
  &.implicit {
    background: transparent !important;
    &:hover {
      background: rgba(var(--color-white-rgb), 0.5) !important;
    }
  }
}

section div {
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  &:not(.selection-dragging) {
    left: v-bind(paddingLeft);
    right: v-bind(paddingRight);
    top: v-bind(paddingTop);
    bottom: v-bind(paddingBottom);
  }
  position: absolute;
  pointer-events: initial;
}
</style>
