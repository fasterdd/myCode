<template>
  <TestForm></TestForm>
  <select-test />
</template>
<script lang="ts">
import { defineComponent } from 'vue'
import TestForm from './components/TestForm.vue'
import SelectTest from './components/SelectTest.vue'
export default defineComponent({
  name: 'App',
  components: {
    TestForm,
    SelectTest,
  },
})
</script>

<style></style>
