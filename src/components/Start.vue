<template>
  <div class="col-9 col-md-6 col-lg-4">
    <div class="form">
      <input type="text" :value="modelValue.name"
      @input="$emit('update:modelValue', objectReturn($event.target.value, 'name'))">
      
      <div>
        <input type="radio" :value="modelValue.number"
        @input="$emit('update:modelValue', objectReturn(parseInt($event.target.value), 'number'))">
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
    }
  },
  props: ['modelValue'],
  emits: ['update:modelValue'],
  created() {
  },
  methods: {
    objectReturn(val, index) {
      let model = this.modelValue
      model[index] = val
      return model
    }
  }
}
</script>