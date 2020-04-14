<template>
  <button class="button" @click="sendNumber">
    {{ operation }}
  </button>
</template>

<script>
export default {
  name: "OperationInput",
  props: {
    operation: {
      type: String,
      required: true,
    }
  },
  methods: {
    sendNumber() {
      this.$emit('clicked', this.operation);
    }
  }
}
</script>

<style scoped>
.button {
  border: 1px solid green;
}
</style>
