<template>
  <home/>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import home from './components/Home.vue'

export default defineComponent({
  name: 'App',
  components: {
    home
  }
})
</script>

<style>
</style>