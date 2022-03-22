<template>
  <div class="col-9 col-md-6 col-lg-4">
    <div class="form">
      <input type="text" v-model="playerName">
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
    }
  },
  props: ['playerName'],
  created() {
  },
  watch: {
  }
}
</script>