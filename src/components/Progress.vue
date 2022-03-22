<template>
  <div class="col-9 col-md-6 col-lg-4">
    <div class="loader" :class="loaderClass">
      <img src="../assets/logo.png" alt="Logo">
      <div class="row justify-content-center">
        <div class="col-8">
          <div class="progress">
            <div class="bar" :style="'width: '+progress+'%'">Loading</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      loaderClass: ''
    }
  },
  props: ['progress'],
  created() {
  },
  watch: {
    progress(value) {

      if (value > 95) {
        this.loaderClass = 'opacity-down'
      } else {
        this.loaderClass = ''
      }
    }
  }
}
</script>