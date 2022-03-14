<script>
export default {
  props: ['screen'],
  data() {
    return {
      darkTheme: true
    }
  },
  methods: {
    changeTheme(switchTheme = true) {
      if (switchTheme) 
        this.darkTheme = !this.darkTheme

      document.querySelector('html').setAttribute('data-theme', this.darkTheme ? 'dark' : 'light')
      localStorage.setItem('theme', (this.darkTheme ? 'dark' : 'light'))
    }
  },
  mounted() {
    if (!this.darkTheme && localStorage.getItem('theme') && localStorage.getItem('theme') === 'dark') {
      this.darkTheme = true
    }
    this.changeTheme(false)
  }
}
</script>
<template>
  <nav class="navbar navbar-expand-sm fixed-top" :class="screen != 'start' ? 'focus-game' : ''">
    <div class="container-fluid">
      <a :class="screen == 'start' && 'hidden-logo'" class="navbar-brand" href="#">
        <img src="@/assets/logo.svg" alt="Vueno">
      </a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <svg style="width:24px;height:24px" viewBox="0 0 24 24">
          <path fill="currentColor" d="M12,16A2,2 0 0,1 14,18A2,2 0 0,1 12,20A2,2 0 0,1 10,18A2,2 0 0,1 12,16M12,10A2,2 0 0,1 14,12A2,2 0 0,1 12,14A2,2 0 0,1 10,12A2,2 0 0,1 12,10M12,4A2,2 0 0,1 14,6A2,2 0 0,1 12,8A2,2 0 0,1 10,6A2,2 0 0,1 12,4Z" />
        </svg>
      </button>
      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav mx-auto">
          <li class="nav-item">
            <a class="nav-link active" aria-current="page" href="#">Play</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#" data-bs-toggle="modal" data-bs-target="#historyModal">History</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#" type="button" data-bs-toggle="offcanvas" data-bs-target="#about" aria-controls="about">About</a>
          </li>
          <li class="nav-item">
            <!-- <a class="nav-link" @click="changeTheme()" href="#">Theme: {{ darkTheme ? 'Dark' : 'Light' }}</a> -->
          </li>
        </ul>
      </div>
    </div>
  </nav>
</template>