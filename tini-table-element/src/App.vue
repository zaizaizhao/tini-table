<script setup lang="ts">
import { ref } from 'vue';
import TiniTableElement from './components/TiniTableElement.vue';

const str = ref("hello")
const rows = ref<number>(5);
const cols = ref<number>(4);
</script>

<template>
  <TiniTableElement :title="str" :rows="rows" :cols="cols" />
</template>

<style scoped>

</style>
