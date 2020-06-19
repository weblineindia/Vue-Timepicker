<!-- *************************************************************************

This File will be used to input text

************************************************************************* -->

<template>
  <input
    :class="['p-inputtext p-component', { 'p-filled': filled }]"
    :value="value"
    v-on="listeners"
  />
</template>

<!-- *************************************************************************

SCRIPT

************************************************************************* -->

<script>
export default {
  props: {
    value: null,
  },
  computed: {
    listeners() {
      return {
        ...this.$listeners,
        input: (event) => this.$emit('input', event.target.value),
      }
    },
    filled() {
      return this.value != null && this.value.toString().length > 0
    },
  },
}
</script>

<!-- *************************************************************************
     STYLE
	************************************************************************* -->
  
<style scoped>
@import "./css/inputtext.css";
</style>
