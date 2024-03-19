<template>
  <template v-for="content in option.contents" :key="content.key">
    <span>{{ content.prefix }}</span>
    <input v-if="content.type === 'text'" class="blank" type="text" v-model="content.value" />
    <span>{{ content.suffix }}</span>
  </template>
</template>

<script setup>
import { defineProps, ref,toRefs } from 'vue'
const props = defineProps({
  option: {
    type: Object,
    required: true
  }
})
const { option } = toRefs(props);

</script>

<style lang="scss" scoped>
.blank {
  display: inline;
  width: 100px;
  outline: none;
  border: unset;
  border-bottom: 1px solid #ccc;
  margin: 0 5px
}
</style>