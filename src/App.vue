<template>
  <div class="container">
    <Calendar />
  </div>
</template>

<script lang="ts">
  import { defineComponent } from 'vue'
  import Calendar from './components/Calendar/Calendar.vue'

  export default defineComponent({
    name: 'TestComponent',
    components: {
      Calendar,
    },
  })
</script>

<style scoped lang="scss">
  .container {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
