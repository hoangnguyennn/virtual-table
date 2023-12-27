<template>
  <virtual-table
    :rows="state.rows"
    :columns="state.columns"
    :cell-width="state.cellWidth"
    :cell-height="state.cellHeight"
  />
</template>

<script setup lang="ts">
import { reactive } from 'vue';
import VirtualTable from './components/VirtualTable.vue';

// 150x90 = 13.500 ô
const state = reactive({
  rows: 150,
  columns: 90,
  cellWidth: 100,
  cellHeight: 80,
});
</script>
