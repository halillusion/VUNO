<template>
  <div class="progress">
    <div class="bar" :style="'width: '+width+'%'"></div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      width: 0
    }
  },
  created() {

    const WIDTH = setInterval(() => {
      this.width+=5
      if (this.width>100) {
        this.width = 0
        // clearInterval(WIDTH)
      } else if (this.width==50 || this.width==80) {
        this.width+=20
      } else if (this.width==15) {
        this.width+=25
      }
    }, 1000)

  }
}
</script>