<script setup lang="ts">
import { onMounted } from 'vue';
import PieChart from './PieChart.vue';
import { nest } from 'd3-collection';

const props = defineProps<{ sliceData: Array<any> }>();

// Count instances of each character
function countUnique(data: Array<object>, property:string) {
  // let flatten = data.map(d => d[property]);
}
</script>

<template>
  <div>
    <!-- <PieChart /> -->
  </div>
</template>

<style>

</style>